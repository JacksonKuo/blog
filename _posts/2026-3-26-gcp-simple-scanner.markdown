---
layout: post
title: "GCP Simple Scanner"
date: 2026-3-26
tags: ["devsecops"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Alrighty, it's been a while since I've thought about how this all works in GCP. Let's it all down for clarity. 

* Simple scanner
  - generate image
  - upload image
* terraform-gcp
  - scheduler
  - obserability
  - bigquery
* grafana
  - sql 

# Simple Scanner
I made a simple mock scanner

```python
def main():

    data = """{
    }"""
    with open("/tmp/output.json", "w") as f:
        f.write(data)

if __name__ == "__main__":
    main()
```

Build and load Dockerfile:

* Dockerfile
  - runs `git clone`
  - runs `entrypoint.sh`
* `gcloud auth login`
* `gcloud auth configure-docker us-central1-docker.pkg.dev`
  - should be already configured
* local `GITHUB_TOKEN` env var needs fine-grainted `contents:read`
* `./build.sh`
  - just runs `docker buildx` and pushes to artifact repo

# Terraform

* `module "github_org_scanner"` in [jobs/main.tf](https://github.com/JacksonKuo/terraform-gcp/blob/main/jobs/main.tf) creates an instance of `google_cloud_run_v2_job`
* `google_cloud_scheduler_job` also in in [jobs/main.tf](https://github.com/JacksonKuo/terraform-gcp/blob/main/jobs/main.tf) schedules the cron job

* Job Execution: [https://console.cloud.google.com/run/jobs/details/us-central1/github-org-scanner/executions](https://console.cloud.google.com/run/jobs/details/us-central1/github-org-scanner/executions)
* Cloud Storage: [https://console.cloud.google.com/storage/browser/jkuo-security-hcp-demo-bucket](https://console.cloud.google.com/storage/browser/jkuo-security-hcp-demo-bucket)

Big Query: [https://github.com/JacksonKuo/terraform-gcp/blob/main/observability/bigquery.tf](https://github.com/JacksonKuo/terraform-gcp/blob/main/observability/bigquery.tf)

# References
[^1]: []()