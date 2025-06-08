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
The fundamental building blocks will

* Option 
    * Fargate 
    * Github Action
* S3 Bucket
* Athena/Glue
* Grafana Cloud

The reason I'm using Fargate instead of AWS Lambda is that Lambdas have a maximum runtime of 15 minutes per execution[^1]. 

Github Actions have a longer runtime, i.e. 35 days[^2] and can natively pull code. It could be argued that Actions are more catered to CI/CD workflows such as PRs versus regular security scans. Additionally, Actions have a history of outages. 

# AWS Setup
Admin account
Terraform

# AWS S3 Bucket
I'll be leveraging S3 for storage. 

| Type | Description | Rate | Price |
|---|---|---|---|
| Standard | General Freq Access | First 50 TB / Month | $0.023 per GB |
| Standard - Infrequent Access | | | |
| Intelligent | | | |
| Express One Zone | | | |
| One Zone | | | |
| Glacier | | | |


S3 -> Glue -> Athena -> Grafana

Amazon Athena	$5.00 per TB scanned

Loki

# References
[^1]: [https://aws.amazon.com/lambda/faqs/#topic-1](https://aws.amazon.com/lambda/faqs/#topic-1)

[^2]: [https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/actions-limits](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/actions-limits)