---
layout: post
title: "GCP Simple Scanner"
date: 2026-3-26
tags: ["devsecops"]
published: true
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

* 

# References
[^1]: []()