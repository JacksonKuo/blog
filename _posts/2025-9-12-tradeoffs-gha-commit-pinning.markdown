---
layout: post
title: "Tradeoffs - GHA Commit Pinning"
date: 2025-9-1
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
What are the trade offs with following tools for GitHub Action auto-commit pinning?

* Dependabot
* Renovate
* suzuki-shunuske/pinact
* stacklok/frizbee
* sethvargo/ratchet

# Tooling

https://developerwithacat.com/blog/202508/github-actions-commit-hash-pinning-tradeoffs/

https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts#detection-of-insecure-dependencies

Dependabot will only create Dependabot alerts for vulnerable GitHub Actions that use semantic versioning. You will not receive alerts for a vulnerable action that uses SHA versioning. If you use GitHub Actions with SHA versioning, we recommend enabling Dependabot version updates for your repository or organization to keep the actions you use updated to the latest versions.

https://www.reddit.com/r/devops/comments/1n0tl0o/commit_hash_pinning_in_github_actions_secure_but/

# References
[^1]: []()