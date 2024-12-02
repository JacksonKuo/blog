---
layout: post
title: "Trufflehog Shallow Clone Insecurity - Part II"
date: 2024-11-20
tags: ["vulnerability"]
#published: false
---

# Problem Statement

Following a compromised Github Action via shell injection, what post-exploitation is possible? Saying this more explicitly, can the following occur:

* Can a compromised GHA on a public repo lead to compromise of internal/private repositories?
* What configurations need to be set for escalation?

# zzz

| Type | Secrets | Trigger | Token |
|---|---|---|---|
| pull_request | none | branch/fork | none |
| pull_request_target | base repo | x | read/write |
| issue_ | x | x | x |

* https://medium.com/tinder/exploiting-github-actions-on-open-source-projects-5d93936d189f

# GITHUB_TOKEN Permissions

```
permissions:
  actions: read|write|none
  attestations: read|write|none
  checks: read|write|none
  contents: read|write|none
  deployments: read|write|none
  id-token: write|none
  issues: read|write|none
  discussions: read|write|none
  packages: read|write|none
  pages: read|write|none
  pull-requests: read|write|none
  repository-projects: read|write|none
  security-events: read|write|none
  statuses: read|write|none
```

* `permissions: read-all`
* `permissions: write-all`

Details: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token#overview

# Secrets

secrets

# Checkout

`persist-credentials` determines if the Github Token is saved to the local git config.The default setting is true

# Notes

* Trufflehog typically is run with status checks and a Github App

# Trufflehog Permissions

```
on: 
  push: 
    branches: 
      - main 
  pull_request:
```

```
permissions: 
  contents: read 
  id-token: write 
  issues: write 
  pull-requests: write
```

# Repo Access / Repo Visibility

zz

# Token Types

* Default
* PAT
* Github App

### References:
[^1]: [https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token#defining-access-for-the-github_token-permissions](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token#defining-access-for-the-github_token-permissions)

[^2]: [https://github.com/actions/checkout?tab=readme-ov-file#checkout-v4](https://github.com/actions/checkout?tab=readme-ov-file#checkout-v4)

[^3]: [https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication#permissions-for-the-github_token](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)

[^4]: [https://trufflesecurity.com/blog/running-trufflehog-in-a-github-action](https://trufflesecurity.com/blog/running-trufflehog-in-a-github-action)