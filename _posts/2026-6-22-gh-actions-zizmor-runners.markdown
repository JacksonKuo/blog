---
layout: post
title: "GitHub Defense: Part VIII - GHA Runners"
date: 2026-6-22
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Thinking through GitHub runner security via zizmor

#### Zizmor Self-Hosted Runner Rule
The documentation is a little confusing. Zizmor has a rule for self-hosted runners. In the usage it says it requires `persona=auditor`[^1], but in the audit rules, it says `--pedantic`[^2]. I think `auditor` is the correct setting

The [PR #34](https://github.com/zizmorcore/zizmor/pull/34/changes) comments includes some useful info: 
> NOTE: GHA docs are unclear on whether runner groups always imply self-hosted runners or not. All examples suggest that they do, but I'm not sure.

> This audit is "pedantic" only, since zizmor can't detect! whether self-hosted runners are ephemeral or not.

Seems like zizmor parses the workflow file and looks for `self-hosted` or runner groups. And looks like zizmor doesn't call the REST API for runners, since the permissions are much higher: `only owner or administrator accounts can see the runner settings`.[^3] And the ephermerality can't easily be discovered via GitHub.

#### Self-Hosted Runners
How this works is there's self-hosted and GitHub-hosted runners. GitHub-hosted runners are pay-as-you-go. Runner groups are essentially permissions that can be assigned to each runner instance that restrict where and what the runner can execute.

* Runner Groups
  * Repository access - `All`/`Selected repositories`
    * Allow public repositories
      * > Runners can be used by public repositories. Allowing self-hosted or image-generation runners on public repositories and allowing workflows on public forks introduces a significant security risk.
  * Workflow access - `All`/`Selected workflows`

You can see if a runner is GitHub-hosted if it has a Github icon to the left of the runner name.  The runner name is also now a label that can be used in a `run-ons: <label>`. And then you can also call a group via:

```yaml
    runs-on: 
      group: ubuntu-runners
```

> The runs-on key sends the job to any available runner in the ubuntu-runners group

On that note, this statement seems incorrect

> > NOTE: GHA docs are unclear on whether runner groups always imply self-hosted runners or not. All examples suggest that they do, but I'm not sure.

I don't see why you couldn't do a runner group with a GitHub-hosted runner? I'm not sure if zizmor is handling this correctly...

Also you can combine the label and group:

```yaml
runs-on:
      group: ubuntu-runners
      labels: ubuntu-24.04-16core
```

And then self-hosted runner[^4] may have the label `self-hosted`:

> Self-hosted runners may have the self-hosted label. When setting up a self-hosted runner, by default we will include the label self-hosted. You may pass in the --no-default-labels flag to prevent the self-hosted label from being applied.

And then GitHub has a list of standard GitHub runners, such as: `ubuntu-latest`[^5]

Interestingly, theres a single CPU runners invoked via `ubuntu-slim` that are run in containers, not fresh VMs.

> ubuntu-slim runners execute Actions workflows in Ubuntu Linux, inside a container rather than a full VM instance. When the job begins, GitHub automatically provisions a new container for that job.

tktk how to determine from a workflow if self-hosted or not?

# Reference
[^1]: [https://docs.zizmor.sh/usage/#using-personas](https://docs.zizmor.sh/usage/#using-personas)

[^2]: [https://docs.zizmor.sh/audits/#self-hosted-runner](https://docs.zizmor.sh/audits/#self-hosted-runner)

[^3]: [https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners/use-larger-runners#running-jobs-on-your-runner](https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners/use-larger-runners#running-jobs-on-your-runner)

[^4]: [https://docs.github.com/en/enterprise-cloud@latest/actions/how-tos/write-workflows/choose-where-workflows-run/choose-the-runner-for-a-job#choosing-self-hosted-runners](https://docs.github.com/en/enterprise-cloud@latest/actions/how-tos/write-workflows/choose-where-workflows-run/choose-the-runner-for-a-job#choosing-self-hosted-runners)

[^5]: [https://docs.github.com/en/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-public-repositories](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-public-repositories)
