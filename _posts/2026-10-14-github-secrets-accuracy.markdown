---
layout: post
title: "GitHub Secrets - Workflow Types"
date: 2025-10-12
tags: ["IaC"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
My general question is how accurate is `gh workflow list`. 

Generate a list of different types of GitHub workflows

Renamed workflows? change name
branches
    workflow_dispatch, no trigger
    has to be an event that support branches
reusable workflows, only default branch

#### Workflow Types

| Name | Description | Actions Tab | Branch |
|---|---|---|
| on.schedule | normal events | appears | default |
| on.push | normal events | appears | default |
| on.pull_request | normal events | appears | default |
| on.pull_request_target | normal events | appears | default |
| on.repository_dispatch | webhook event for activity that happens outside of GitHub | appears | default |
| on.workflow_run | `execute a workflow based on execution or completion of another workflow`<br><br>`workflow started by the workflow_run event is able to access secrets and write tokens, even if the previous workflow was not.` | appears | default |
| on.workflow_call | reusable workflows | appears | default |
| on.workflow_dispatch | workflow to be triggered manually via API or UI | appears | default |

`gh workflow lists`
* invoked workflows
* dynamic/dependabot/dependabot-updates
* dynamic/github-code-scanning/codeql
* dynamic/pages/pages-build-deployment

actions

secrets

secret.secret_name

secrets:
    inherit:

do org secrets, get auto included in workflow_calls 2, 3, 4, without inherit
    what about intra vs inter workflows

workflow_call don't auto-load org_secrets, despite them being org secrets

recursion

combo of trigger types

according to gpt, can't combined with other triggers
    workflow_call
    workflow_run

    though looks like you can combine workflow_call with workflow_dispatch

# References
[^1]: [https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#on](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#on)

[^2]: [https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)

[^3]: [https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)
