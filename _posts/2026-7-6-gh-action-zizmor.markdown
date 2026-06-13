---
layout: post
title: "GitHub: Actions - Zizmor"
date: 2026-8-7
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
How to fix zizmor template injection findings holistically

# Zizmor
Not going to explain zizmor, if ya know ya know.
* [https://docs.zizmor.sh/quickstart/](https://docs.zizmor.sh/quickstart/)
* [https://docs.zizmor.sh/usage/](https://docs.zizmor.sh/usage/)
* [https://docs.zizmor.sh/integrations/](https://docs.zizmor.sh/integrations/)

* [https://github.com/zizmorcore/zizmor](https://github.com/zizmorcore/zizmor)
* [https://github.com/zizmorcore/zizmor/blob/main/docs/audits.md#template-injection](https://github.com/zizmorcore/zizmor/blob/main/docs/audits.md#template-injection)

There are 4 output formats:
1. `plaintext` - cargo-style, easy to read, hard to parse
2. `json` - there seems to be multiple json versions, like `json-v1`. just use `json` which is the latest version 
3. `sarif` - super verbose, hard to read
4. `github` - very minimal, intended for annotations

There are 3 `personas`[^1]:
1. `regular` (default) actionable security findings
2. `pedantic` - code smell, non-actionable
3. `auditor` - everything


# Auto-Fix
* [https://docs.zizmor.sh/usage/#auto-fixing-results](https://docs.zizmor.sh/usage/#auto-fixing-results)

How does auto-fix work? Auto-fix has different modes:
* `--fix` (default safe)
* `--fix=all`
* `--fix=unsafe-only`

```yaml
name: zizmor-template-inject

on:
  pull_request_target:
    types: [opened, edited]

jobs:
  vuln-check:
    runs-on: ubuntu-latest
    steps:
      - name: 
        run: |
          # danger
          echo "PR Title: ${GITHUB_EVENT_PULL_REQUEST_TITLE}"
          echo "PR Ref: ${GITHUB_EVENT_PULL_REQUEST_HEAD_REF}"
          
          # safe
          echo "SHA: ${{ github.sha }}"
        env:
          GITHUB_EVENT_PULL_REQUEST_TITLE: ${{ github.event.pull_request.title }}
          GITHUB_EVENT_PULL_REQUEST_HEAD_REF: ${{ github.event.pull_request.head.ref }}

```

# Threat Model
`Approval for running fork pull request workflows from contributors`: 
Require approval for all external contributors

zizmor --offline . --format json --fix

# References
[^1]: [https://docs.zizmor.sh/usage/#using-personas](https://docs.zizmor.sh/usage/#using-personas)