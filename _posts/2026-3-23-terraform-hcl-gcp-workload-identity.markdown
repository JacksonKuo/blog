---
layout: post
title: "Terraform: HCL and GCP Workload Identity"
date: 2026-3-23
tags: ["IaC"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Remind myself how to add a GH repo to auto sync with HCL Terraform. And also include a refresher on how I setup GCP federated workload identity originally. 

# HCL Terraform
I believe I mostly followed this documentation and that it was quite tedious to setup:
* [https://developer.hashicorp.com/terraform/cloud-docs/dynamic-provider-credentials/workload-identity-tokens](https://developer.hashicorp.com/terraform/cloud-docs/dynamic-provider-credentials/workload-identity-tokens)
* [https://developer.hashicorp.com/terraform/cloud-docs/dynamic-provider-credentials/workload-identity-tokens](https://developer.hashicorp.com/terraform/cloud-docs/dynamic-provider-credentials/workload-identity-tokens)

I have HCL Terraform synced to specific repos in [https://github.com/JacksonKuo/terraform-gcp](https://github.com/JacksonKuo/terraform-gcp). Right now I have three folders in GitHub:

* deploys
* jobs
* observability

To login, put in your email and then click GitHub SSO.

Under `Workspace > Settings > Version Control`:

* Make sure VCS is connected
* Terraform Working Directory is filled out

Under `Workspace > Variables`:

* `TFC_GCP_PROVIDER_AUTH` = `true`
* `TFC_GCP_RUN_SERVICE_ACCOUNT_EMAIL` = `hcp-tf-workload-identity@jkuo-security.iam.gserviceaccount.com`
* `TFC_GCP_WORKLOAD_PROVIDER_NAME` = `projects/xxx/locations/global/workloadIdentityPools/hcp-terraform-pool/providers/hcp-terraform-provider`

Under `GCP > IAM & Admin > Workload Identity Federation`

* Provider name: HCP Terraform Provider
* Issue: `https://app.terraform.io`
* Audience: default audience
* Enabled provider: checked
* Attribute mapping: `Google1: google.subject`
* Attribute mapping: `OIDC1: assertion.sub`
* Attribute conditions: `assertion.sub.startsWith("organization:jkuo-tf-org:project:security:workspace:appsec-")`

Under IAM & Admin > Service Accounts > View by Principals

* `principal://iam.googleapis.com/projects/xxx/locations/global/workloadIdentityPools/hcp-terraform-pool/subject/organization:jkuo-tf-org:project:security:workspace:appsec-deploys:run_phase:apply` to `Workload Identity User`

And ensure there's one for apply, destroy, and plan. 

# References
[^1]: 
