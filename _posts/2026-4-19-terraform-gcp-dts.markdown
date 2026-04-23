---
layout: post
title: "Terraform: GCP BigQuery DTS"
date: 2026-4-19
tags: ["IaC"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Remind myself how i setup the BigQuery Data Transfer Service (DTS) here: [https://github.com/JacksonKuo/terraform-gcp/blob/main/observability/sa.tf](https://github.com/JacksonKuo/terraform-gcp/blob/main/observability/sa.tf)

The GCP documentation for setting up DTS is horrendous. I think i only got this working after many hours working with agents and watching youtube videos. 

# Accounts
I've configured two accounts:

* Custom service account
    - `dts-gcs-loader@jkuo-security.iam.gserviceaccount.com`

| Type | Name | Role | 
|---|---|---|
| GCS Bucket | jkuo-security-hcp-demo-bucket| storage.objectViewer |
| BQ Dataset | github_org_scanner | bigquery.dataEditor |

* GCP default DTS service account
    - `service-{project_number}@gcp-sa-bigquerydatatransfer.iam.gserviceaccount.com`

| Type | Name | Role | 
|---|---|---|
| Service Account | dts-gcs-loader SA | iam.serviceAccountTokenCreator |
| BQ Table | scan_repos table | bigquery.dataEditor |
| BQ Table | scan_visibility table | bigquery.dataEditor |

# Explanation 
A few things to notes:
* When the BigQuery Data Transfer Service API is enabled, GCP creates the default service account `service-{project_number}@gcp-sa-bigquerydatatransfer.iam.gserviceaccount.com`. This account also known as the service agent[^1]
* The DTS default SA uses the `serviceAccountTokenCreator` permissions to impersonate the `dts-gcs-loader` custom SA.
* Impersonation generates a temp token that allows reading from GCS using the `dts-gcs-loader` permissions. 
* After reading from GCS, the DT default SA uses it's own identity to write to BigQuery
* Service accounts are used so that execution is not tied to an individual user
* The custom service account needs the following permissions:[^2]
    - `bigquery.datasets.get`
    - `bigquery.datasets.update`
* The role `bigquery.dataEditor` has both those permissions.
* Control Plane = custom SA, used to validate dataset
* Data Plane = default SA, used to write dataset[^3]
* Transfer Creator = sets up transfer config (Terraform account)
* Transfer Owner = `user identity that the BigQuery DTS uses to authorize the data transfer, specifically, for extracting the source data.`[^4]

# Setup Instructions:
1. Enable BQ DTS API, which creates default DTS SA
    * To see the account, IAM & Admin > click the Include Google-provided role grants
2. Running `terraform apply` creates the `dts-gcs-loader` SA
    * Note the account that creates the DTS transfer config, needs `bigquery.admin`. 
3. DTS schedule triggers
    * default DTS SA `gcp-sa-bigquerydatatransfer` impersonates the `dts-gcs-loader` SA and generates a temporary token
    * Temp token uses custom SA permissions to read from GCS bucket
    * The DTS default SA `gcp-sa-bigquerydatatransfer` and writes data to BQ tables

> Note: Starting March 17, 2026, the BigQuery Data Transfer Service will require the bigquery.datasets.getIamPolicy and bigquery.datasets.setIamPolicy permissions. For more information, see Changes to dataset-level access controls.[^5]

I'm not going to lie, i still don't fully understand how all this works... but some how it works in my instance

I might not need these:
* dts_bq_editor — custom SA doesn't need dataEditor, only datasets.get and datasets.update
* dts_agent_bq_editor_repos — DTS grants this automatically
* dts_agent_bq_editor_visibility — DTS grants this automatically

> If DTS automatically grants dataset-level dataEditor to the default DTS SA at transfer creation time, your table-level restrictions are effectively bypassed by that broader grant

> So this only impacts you if you're using a custom role with just bigquery.datasets.get or bigquery.datasets.update. In that case you'd need to add getIamPolicy and setIamPolicy to that custom role.

> If you're using predefined roles like bigquery.admin for the Terraform SA, you're already covered.

Apparently, the Schedule backfill will only run once on a GCS file, unless the file has been updated. 

# References
[^1]: [https://docs.cloud.google.com/bigquery/docs/enable-transfer-service#service_agent](https://docs.cloud.google.com/bigquery/docs/enable-transfer-service#service_agent)
[^2]: [https://docs.cloud.google.com/bigquery/docs/use-service-accounts](https://docs.cloud.google.com/bigquery/docs/use-service-accounts)
[^3]: [https://docs.cloud.google.com/bigquery/docs/dts-authentication-authorization](https://docs.cloud.google.com/bigquery/docs/dts-authentication-authorization)
[^4]: [https://docs.cloud.google.com/bigquery/docs/dts-authentication-authorization#transfer_creator_versus_transfer_owner](https://docs.cloud.google.com/bigquery/docs/dts-authentication-authorization#transfer_creator_versus_transfer_owner)
[^5]: [[https://docs.cloud.google.com/bigquery/docs/dataset-access-control](https://docs.cloud.google.com/bigquery/docs/dataset-access-control)
[^6]: [https://docs.cloud.google.com/iam/docs/roles-permissions/bigquery](https://docs.cloud.google.com/iam/docs/roles-permissions/bigquery)