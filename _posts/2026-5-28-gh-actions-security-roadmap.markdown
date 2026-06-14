---
layout: post
title: "GitHub Defense: Part III - Actions Security Roadmap"
date: 2026-5-28
tags: ["github"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Therew are some big changes to GitHub Actions this year. Breakdown the changes from the GitHub Actions 2026 Security Roadmap: [https://github.blog/news-insights/product-news/whats-coming-to-our-github-actions-2026-security-roadmap/](https://github.blog/news-insights/product-news/whats-coming-to-our-github-actions-2026-security-roadmap/) (Release date: March 26, 2026)

Also there some great discussions that were brought up here: [https://github.com/orgs/community/discussions/190621](https://github.com/orgs/community/discussions/190621)

Most of this is going to be rote copy and paste from the roadmap, btw. 

#### 1. Building a more secure Actions ecosystem
* Workflow-level dependency locking
    * We’re introducing a dependencies: section in workflow YAML that locks all direct and transitive dependencies with the commits SHA 
    * Think of it as Go’s go.mod + go.sum, but for your workflow with complete reproducibility and auditability

```yaml
dependencies:
    - github.com/actions/checkout@v6.0.1:sha-123
```

#### 2. Reducing attack surface with secure defaults
* Policy-driven execution
    * workflow execution protections built on GitHub’s ruleset framework
    * you define central policies that control:
        * who can trigger workflows
        * which events are allowed
    * core policy dimensions 
        * Actor rules specify who can trigger workflows such as: 
            * individual users
            * roles like repository admins
            * trusted automation like GitHub Apps, GitHub Copilot, or Dependabot
        * Event rules define which GitHub Actions events are permitted like push, pull_request, workflow_dispatch, and others.
    * could prohibit `pull_request_target` events entirely and only allow `pull_request`, ensuring workflows triggered by external contributions run without access to repository secrets or write permissions

#### 3. Scoped secrets and improved secret governance
* Scoped secrets
    * New scopes
        * Specific repositories or organizations
        * Branches or environments
        * Workflow identities or paths
        * Trusted reusable workflows without requiring callers to pass secrets explicitly
    * Effects
        * Secrets are no longer implicitly inherited
        * Access requires matching an explicit execution context
        * Modified or unexpected workflows won’t receive credentials
* Reusable workflow secret inheritance
    * Secrets are bound directly to trusted workflows
    * Callers don’t automatically pass credentials
    * Trust boundaries are explicit
* Permission model changes for Action Secrets
    * separating code contributions from credential management
    * write access to a repository will no longer grant secret management permissions and helps us move toward least privilege by default
    * capability will instead be available through a dedicated custom role and will remain part of the repository admin, organization admin, and enterprise admin roles

#### 4. Endpoint monitoring and control for CI/CD infrastructure
* Increased visibility with Actions Data Stream
    * Near real-time execution telemetry
    * Centralized delivery to your existing systems
        * Amazon S3 / Azure
    * Workflow and job execution details across repositories and organizations
    * Dependency resolution and action usage patterns

#### 5. Native egress firewall for GitHub-hosted runners
* Native egress firewall for GitHub-hosted runners
    * Allowed domains and IP ranges
    * Permitted HTTP methods
    * TLS and protocol requirements
    * Monitor / Enforce capabilities

# What does this mean and what can we do?

| Change | Initial Impression | Use Cases |
|---|---|---|
| Action lockfile | Nice | n/a |
| Rulesets for workflows execution | This is the big one. Very cool. <br><br> But seems kinda tricky because you might break existing functionality. And seems kinda hard to rightsize this for individuals | Could be helpful locking down GHApps with `pull_request` WRITE perms <br><br> For highly sensitive repos restricting workflow triggers for `internal` visiblity? <br><br> Prohibiting `pull_request_target` is a big one |
| Scoped secrets | There's a lot of changes here. <br><br> New scope: workflows identities, trusted reusable workflows <br><br> Secret inheritance: scoped secrets aren't auto-inherited <br><br> Repo write perm: Didn't even realize this gave secrets management | Not sure how big an impact this will be. Could an tie org secret to a reusable workflow. Maybe it's harder to dump the secret since it's not inheritable? |
| Better logging | Nice | n/a |
| Firewall | Nice | Maybe getting a list of approval domains. I doubt this would really stop exfiltration unless the list was actually locked down. And I don't know if applies to self-hosted runners |

I'll fleshout the attack surface of these new security improvement overtime and get a better understanding what can easily be incorporated and where we can further locked down the GitHub ecosystem.

And as far as i can tell none of this directly really helps with the org secret problem. Though it can decrease the attack surface from `pull_request_target` and maybe help with decreasing overscoped secret inheritance.