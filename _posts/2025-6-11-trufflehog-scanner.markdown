---
layout: post
title: "Trufflehog - Scanner"
date: 2025-6-11
tags: ["devsecops"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's build an Trufflehog scanner that scans a GitHub organization and pipes the data to Grafana.

# Building Blocks
The fundamental building blocks will be:

* `org-admin` IAM user with YubiKey MFA
* `terraform` IAM user + API key
* Local Terraform
* S3 Bucket
* Trufflehog
    * GitHub Action
    * Fargate 
* Athena/Glue
* Grafana Cloud

The reason I'm not using AWS Lambda is that Lambdas have a maximum runtime of 15 minutes per execution[^1]. GitHub Actions have a longer runtime, i.e. 35 days[^2] and can natively pull code. It could be argued that Actions are more catered to CI/CD workflows such as PRs and are not intended to be used for regular security scans[^3]. Additionally, Actions have a rich history of outages. But Actions are much easier to setup, so I'll be testing first using GH Actions. 

# AWS S3 Bucket
S3 Bucket created: `jkuo-prod-us-east-1`

#### Pricing
Based on the expected usage, I should only pay 2.3 cents per month. Requests are negligible and data transfer shouldn't surpass 100GB. Storage will never get close to 50 TB and all objects are auto-expired after 90 days. 

* Storage:
    * S3 Standard, General Frequent Access, First 50 TB / Month, $0.023 per GB[^4]
* Requests
    * PUT, COPY, POST, LIST requests (per 1,000 requests)	
        * $0.005
    * GET, SELECT, and all other requests (per 1,000 requests)
        * $0.0004
* Data transfer:
    * Data Transfer IN To Amazon S3 From Internet
        * All data transfer in - $0.00 per GB
    * Data Transfer OUT From Amazon S3 To Internet
        * First 10 TB / Month - $0.09 per GB
        * AWS customers receive 100GB of data transfer out to the internet free each month
* Security
    * Bucket owner
        * Objects - List, Write
        * Bucket ACL - Read, Write

#### Terraform
```yml
resource "aws_s3_bucket" "main_bucket" {
   bucket = "jkuo-prod-us-east-1"
}

resource "aws_s3_bucket_public_access_block" "main_block" {
  bucket                  = aws_s3_bucket.main_bucket.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_lifecycle_configuration" "main_lifecycle" {
  bucket = aws_s3_bucket.main_bucket.id
  rule {
    status = "Enabled"
    id     = "expire_all_files"
    
    filter {}
    
    expiration {
        days = 90
    }
  }
}
```

# Trufflehog - GitHub Actions
My scanner will live in [https://github.com/JacksonKuo/scanner-trufflehog](https://github.com/JacksonKuo/scanner-trufflehog) and will scan the org: [https://github.com/jkuo-org](https://github.com/jkuo-org). I'll be using a fine-grained PAT token with Contents `readonly` access: `scanner-trufflehog-gh-pat` to scan the following repos:
* [https://github.com/jkuo-org/repo-private](https://github.com/jkuo-org/repo-private)
* [https://github.com/jkuo-org/repo-public](https://github.com/jkuo-org/repo-public)

#### Pricing
My Actions linux usage for the month of May was 27 min with a $0.008 price per minute totaling $0.22[^5]. The GitHub Free tier provides 2000 free minutes per month[^6], so I'm not close to surpassing the limits.

#### Download Trufflehog
There's a lot of different ways to install Trufflehog using GitHub Actions
* Trufflehog docker container
* Trufflehog GH Action
* Shell scripting

The Trufflehog GH Action is intended to be used for scanning PRs using checkout. Shell scripting is fine, but the easier method is to use the packaged container made by Trufflehog and available on GH container registry: [https://github.com/trufflesecurity/trufflehog/pkgs/container/trufflehog](https://github.com/trufflesecurity/trufflehog/pkgs/container/trufflehog). The Trufflehog step requires `continue-on-error: true` to be set because in some settings when a secret is found Trufflehog may return a non-zero exit code, which will be interpreted as a failed step and stop the rest of the action. 

#### Json Reduction
I'm using [Canary Tokens](https://canarytokens.org) to proc Trufflehog and then using `jq` to reduce the Trufflehog output to a minimal JSON file. The Trufflehog container `alpine:3.21`[^7] doesn't have `jq`, `app-get`, `aws`, but it does allow `apk add jq aws-cli`[^8]. 

```json
{"repo":"https://github.com/jkuo-org/repo-private.git","detector":"AWS","link":"https://github.com/jkuo-org/repo-private/blob/9faaa8f4fb5e8e61d8cfd87ffb890b851257c583/canary.properties#L8"}
```
Initial results are saved in artifacts. Artifacts in public repos are public, while private repo artifacts are scoped to those who have permissions to the repo. Note I'm just using artifacts to see the output for testing, eventually the file will be filtered and written to S3. 

#### S3 OIDC
A couple of notes. The `thumbprint_list` is now Optional and for AWS is no longer required: *AWS relies on its own library of trusted root certificate authorities (CAs) for validation instead of using any configured thumbprints*.[^9]

Some good guides for OIDC setup:
* [https://medium.com/@thiagosalvatore/using-terraform-to-connect-github-actions-and-aws-with-oidc-0e3d27f00123](https://medium.com/@thiagosalvatore/using-terraform-to-connect-github-actions-and-aws-with-oidc-0e3d27f00123)
* [https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)

In public repos by default `id-token` is set to  `write` but for private repos by default `id-token` is set to `none`. Don't forget to set `id-token: write`.[^10] Also when `id-token` is explicitly set, all permissions are overwritten to `none`, so `content: read` needs to be reset. 

| Repo Type | id-token | contents |
|------------|----------|----------|
| Public | write | read |
| Private | none | read |

My terraform does the following:
* Add GitHub as a Identity Provider `aws_iam_openid_connect_provider`
* Create a IAM role `github_oidc_role`
* Attaches a inline `assume_role_policy` that creates a trust policy with the GitHub repo: `JacksonKuo/scanner-trufflehog`
* Attaches a inline `aws_iam_policy` that creates a permission policy for S3 `PutObject`


And `aws-actions/configure-aws-credentials@v4` will fail if no role can be assumed. The S3 file write follows this format `trufflehog/date=${DATE}/scan.json"`, which allows one file for each date. Any new files within a day will overwrite the last file.  

# Glue
Glue is a metadata repository that holds partition data. There are two methods to do Glue partitioning. Both methods parse the same folder naming structures: `…/partitionKey=value/` which is called Apache Hive style partitions, where *data paths contain key value pairs connected by equal signs (year=2021/month=01/day=26/)*[^11]. However there are important differences:
* Hive-style partitioning
    * must run `MSCK REPAIR TABLE` by Athena or Glue crawler
    * declared using `PARTITIONED BY (dt string)`
    * AWS Glue crawler auto runs `MSCK`
    * AWS Glue crawler stores partition info in the Glue metadata catalog
    * Adds partition column at runtime
* Partition projection
    * Athena only
    * Skips Glue catalog lookup
    * Skips the Glue crawler
    * declared using `TBLPROPERTIES (`
    * Adds partition column at runtime

#### Pricing[^12]
* Data Catalog
    * Metadata storage:
        * Free for the first million objects stored
        * $1.00 per 100,000 objects stored above 1 million per month
    * Metadata requests:
        * Free for the first million requests per month
        * $1.00 per million requests above 1 million per month
* Glue Crawler
    * $0.44 per DPU-Hour, billed per second, with a 10-minute minimum per crawler run
    * 1 DPU × 0.1667 hours × $0.44/DPU-hour ≈ $0.07 per crawl

Importantly using partition projection skips having to use the Glue crawler. 

# Athena
Athena requires setting a workgroup, which is the S3 output location of the Athena results[^13]. Set `enforce_workgroup_configuration = true` to prevent the location from being changed. 

#### Pricing
* SQL queries
    * $5.00 per TB of data scanned.
    * You are charged for the number of bytes scanned per query, rounded up to the nearest megabyte, with a 10 MB minimum per query
    * Minimum query cost: 10MB ÷ 1,048,576MB/​TB × $5/TB ≈ $0.00005

# Grafana Cloud
* Create a new subdomain: `jkuo.grafana.net`
* Install Athena Plugin: 
* Connections > Data sources: [https://jkuo.grafana.net/plugins/grafana-athena-datasource](https://jkuo.grafana.net/plugins/grafana-athena-datasource)

Couple of things to note. Currently OIDC support does not exist. There is the regular assume role, but according to documentation: *Grafana Assume Role is currently in private preview for Grafana Cloud* (only invited customers), and *only available for Amazon CloudWatch*[^14]. 

For simplicity sake, I create a IAM user with the IAM permissions listed here: [https://grafana.com/grafana/plugins/grafana-athena-datasource/](https://grafana.com/grafana/plugins/grafana-athena-datasource/) for the datasource AWS authentication. 

#### Pricing
* Free tier (woohoo!)
    * 10k series Prometheus metrics
    * 50GB logs, 50GB traces, 50GB profiles
    * 500VUh k6 testing
    * 20+ Enterprise data source plugins
    * 100+ pre-built solutions

#### Dashboard
* Add visualization
    * Last 7 days
    * Axis
        * Time zone: Coordinated Universal Time
        * Soft min: 0
    * Standard options:
        * Unit: Number
        * Decimals: 0

The file path format is `s3://jkuo-prod-us-east-1/trufflehog/date=2025-06-11/scan.json`. And the SQL call is:

```sql
SELECT
  date,
  COUNT(*) AS total_secrets
FROM trufflehog_table
WHERE date >= current_date - interval '7' day
GROUP BY date
ORDER BY date;
```

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/grafana-secrets.png){: width="600"}
{: refdef}
{:refdef: style="text-align: center;"}
\[Grafana Dashboard\]
{: refdef}

For a more advanced guide on Grafana see: [https://aws.amazon.com/blogs/mt/exploring-aws-config-data-using-amazon-athena-and-amazon-managed-grafana/](https://aws.amazon.com/blogs/mt/exploring-aws-config-data-using-amazon-athena-and-amazon-managed-grafana/)

# Future Improvements
* Fargate[^15]
    * VPC
        * Public VPC + security groups
        * Private VPC
        * S3 VPC Endpoints
    * AWS Identity Center
    * GitHub Action OIDC + IAM roles instead of IAM groups
    * Terraform CI/CD linter

# References
[^1]: [https://aws.amazon.com/lambda/faqs/#topic-1](https://aws.amazon.com/lambda/faqs/#topic-1)

[^2]: [https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/actions-limits](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/actions-limits)

[^3]: That said GH Actions are used for all sorts of workflow: [https://github.com/alex/nyt-2020-election-scraper](https://github.com/alex/nyt-2020-election-scraper). 

[^4]: [https://aws.amazon.com/s3/pricing/](https://aws.amazon.com/s3/pricing/)

[^5]: [https://github.com/settings/billing/usage](https://github.com/settings/billing/usage)

[^6]: [https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions)

[^7]: [https://github.com/trufflesecurity/trufflehog/blob/main/Dockerfile](https://github.com/trufflesecurity/trufflehog/blob/main/Dockerfile)

[^8]: [https://pkgs.alpinelinux.org/package/edge/community/x86_64/aws-cli](https://pkgs.alpinelinux.org/package/edge/community/x86_64/aws-cli)

[^9]: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_openid_connect_provider#thumbprint_list-1](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_openid_connect_provider#thumbprint_list-1)

[^10]: [https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect#adding-permissions-settings](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect#adding-permissions-settings)

[^11]: Partition: divides db into different files aka partitions in order to not have to pull all files at once: [https://docs.aws.amazon.com/athena/latest/ug/partitions.html](https://docs.aws.amazon.com/athena/latest/ug/partitions.html), [https://docs.aws.amazon.com/athena/latest/ug/partition-projection.html](https://docs.aws.amazon.com/athena/latest/ug/partition-projection.html)

[^12]: [https://aws.amazon.com/glue/pricing/](https://aws.amazon.com/glue/pricing/)

[^13]: [https://registry.terraform.io/providers/hashicorp/aws/6.0.0-beta3/docs/resources/athena_workgroup](https://registry.terraform.io/providers/hashicorp/aws/6.0.0-beta3/docs/resources/athena_workgroup)

[^14]: [https://grafana.com/docs/grafana/latest/datasources/aws-cloudwatch/aws-authentication/#use-grafana-assume-role](https://grafana.com/docs/grafana/latest/datasources/aws-cloudwatch/aws-authentication/#use-grafana-assume-role)

[^15]: [https://medium.com/@MUmarAmanat/setup-aws-vpc-for-fargate-246d1c515135](https://medium.com/@MUmarAmanat/setup-aws-vpc-for-fargate-246d1c515135)