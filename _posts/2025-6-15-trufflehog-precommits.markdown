---
layout: post
title: "Trufflehog - Precommits Hooks"
date: 2025-6-15
tags: ["devsecops"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's prevent secrets before they're ever committed using Trufflehog pre-commit hooks.

# Overview
So there's actually two ways to prevent commits[^1]: pre-receive and pre-commit hooks. However pre-receive hooks are only if you control the git server. Github.com doesn't support pre-received hooks, but the self-hosted Github Enterprise Server does[^2].  

Trufflehog has a great pre-commit hook guide: [https://docs.trufflesecurity.com/pre-commit-hooks](https://docs.trufflesecurity.com/pre-commit-hooks). Let's walk through it, woohoo!
* `githooks` / `hooksPath`
* Precommit framework

# githooks
A couple things for `githooks`: [https://git-scm.com/docs/githooks](https://git-scm.com/docs/githooks): 
* *Hooks that don’t have the executable bit set are ignored.*
* *By default the hooks directory is `$GIT_DIR/hooks`, but that can be changed via the `core.hooksPath` configuration variable*
    * `$GIT_DIR/hooks` local to a git repo
    * `core.hooksPath` = global
* *can be bypassed with the `--no-verify` option*[^3]
* *non-zero status from this script causes the git commit command to abort before creating a commit*
* *The default pre-commit hook, when enabled, catches introduction of lines with trailing whitespaces and aborts the commit when such a line is found.*

```bash
# Using Homebrew (macOS)
brew install trufflehog
# Using installation script for Linux, macOS, and Windows (and WSL)
# curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh | sh -s -- -b /usr/local/bin

mkdir -p ~/.git-hooks
touch ~/.git-hooks/pre-commit
chmod +x ~/.git-hooks/pre-commit
git config --global core.hooksPath ~/.git-hooks
git config --list
#core.hookspath=/Users/jacksonkuo/.git-hooks
```

```sh
#!/bin/sh
#~/.git-hooks/pre-commit
trufflehog git file://. --since-commit HEAD --results=verified,unknown --fail
```

```sh
#!/bin/sh
#~/.git-hooks/pre-commit
docker run --rm -v "$(pwd):/workdir" -i --rm trufflesecurity/trufflehog:latest git file:///workdir --since-commit HEAD --results=verified,unknown --fail
```

# Precommit framework

# Pros and Cons

Per repo, global?
performance




# References
[^1]: [https://docs.trufflesecurity.com/block-secrets-from-leaking](https://docs.trufflesecurity.com/block-secrets-from-leaking)

[^2]: [https://docs.github.com/en/enterprise-server@3.17/admin/enforcing-policies/enforcing-policy-with-pre-receive-hooks/about-pre-receive-hooks](https://docs.github.com/en/enterprise-server@3.17/admin/enforcing-policies/enforcing-policy-with-pre-receive-hooks/about-pre-receive-hooks)

[^3]: [https://git-scm.com/docs/githooks#_pre_commit](https://git-scm.com/docs/githooks#_pre_commit)