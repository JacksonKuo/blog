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
Generate a list of different types of GitHub workflows

#### Workflow Types

* on

* on.<event_name>.types

    push
    pull_request
    pull_request_target

schedule

repository_dispatch
action

Call other workflows

workflow_run
workflow_call (reusable workflows) - 
workflow_dispatch

dynamic

| Name | Description | x | 
|---|---|---|
| x | x | x |
| x | x | x |
| x | x | x |
| workflow_call | x | x |
| workflow_dispatch | To enable a workflow to be triggered manually | x x |

# References
[^1]: [https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#on](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#on)

[^2]: [https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)

[^3]: [https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)
