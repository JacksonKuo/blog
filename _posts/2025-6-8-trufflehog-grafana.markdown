---
layout: post
title: "Trufflehog Scanner"
date: 2025-6-7
tags: ["scanner"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's build an trufflehog scanner that scans a github organization and pipes the data to Grafana.

# Building Blocks
The fundamental building blocks will be:

* OrgAdmin IAM user with YubiKey MFA
* Terraform IAM user
* Terraform
* S3 Bucket
* Trufflehog
    * Github Action
    * Fargate 
* Athena/Glue
* Grafana Cloud

The reason I'm not using AWS Lambda is that Lambdas have a maximum runtime of 15 minutes per execution[^1]. Github Actions have a longer runtime, i.e. 35 days[^2] and can natively pull code. It could be argued that Actions are more catered to CI/CD workflows such as PRs and are not intended to be used for regular security scans. Additionally, Actions have a rich history of outages. But Actions are much easier to setup. I'll be testing both GH Actions and Fargate.

# AWS S3 Bucket
* Storage:
    * S3 Standard, General Frequent Access, First 50 TB / Month, $0.023 per GB[^3]
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

#### Pricing Estimate
Based on the expected usage, I should only pay 2.3 cents per month. Requests are negligible and data transfer shouldn't surpass 100GB. Storage will never get close to 50 TB and all objects are auto-expired after 90 days. 

```yaml
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

# Github Action
My Actions linux usage for the month of May was 27 min with a $0.008 price per minute totaling $0.22. The Github Free tier provides 2000 free minutes per month[^4], so I'm not close to surpassing the limits.

# References
[^1]: [https://aws.amazon.com/lambda/faqs/#topic-1](https://aws.amazon.com/lambda/faqs/#topic-1)

[^2]: [https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/actions-limits](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/actions-limits)

[^3]: [https://aws.amazon.com/s3/pricing/](https://aws.amazon.com/s3/pricing/)

[^4]: [https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions)