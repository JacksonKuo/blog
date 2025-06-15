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
So there's actually two ways to prevent commits[^1]: pre-receive and pre-commit hooks. However pre-receive hooks are only if you control the git server. Github.com doesn't support pre-received hooks, but the self-hosted Github Enterprise Server does[^2]. Trufflehog has a great pre-commit hook guide: 
* [https://github.com/trufflesecurity/trufflehog/blob/main/PreCommit.md](https://github.com/trufflesecurity/trufflehog/blob/main/PreCommit.md)
* [https://docs.trufflesecurity.com/pre-commit-hooks](https://docs.trufflesecurity.com/pre-commit-hooks). 

Let's walk through `githooks`/`hooksPath` and the precommit framework, woohoo!

# githooks
A couple things for `githooks`: [https://git-scm.com/docs/githooks](https://git-scm.com/docs/githooks): 
* *Hooks that don’t have the executable bit set are ignored.*
* *By default the hooks directory is `$GIT_DIR/hooks`, but that can be changed via the `core.hooksPath` configuration variable*
    * `$GIT_DIR/hooks` local to a git repo
    * `core.hooksPath` = global
* *can be bypassed with the `--no-verify` option*[^3]
* *non-zero status from this script causes the git commit command to abort before creating a commit*
* *The default pre-commit hook, when enabled, catches introduction of lines with trailing whitespaces and aborts the commit when such a line is found.*
* `$GIT_DIR/hooks` is not uploaded to Github.com

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
docker run --rm -v "$(pwd):/workdir" -i trufflesecurity/trufflehog:latest git file:///workdir --since-commit HEAD --results=verified,unknown --fail
```
A breakdown of the docker command: 
* `docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]`[^4]
    * `docker container run` = `docker run`
    * `-rm` = removes container when exits
    * `-v` = mount a volume
    * `-i` = *keeps the container's STDIN open, and lets you send input to the container through standard input*
    * `--fail` = return exit code 183 if results are found
    * `git file://` + `/workdir` = trufflehog git command that scans local git repo
        * note that `git` is preferred over `filesystem`, since `filesystem` can't look into packfiles. The one exception is if the repository is corrupted, then `filesystem` should be used.[^4]
    * exit code[^6]
        * 0 = no errors. when verified secrets are found or no results are found. tested with canaries and live AWS tokens
        * 1 = when an error occurs
        * 183 = `--fail` set and secrets are found

```bash
jacksonkuo@Mac blog % git commit -m "update"
Unable to find image 'trufflesecurity/trufflehog:latest' locally
latest: Pulling from trufflesecurity/trufflehog
5245f6332319: Download complete
dee434cacd1b: Download complete
4f4fb700ef54: Already exists
6b6bccb23c39: Download complete
Digest: sha256:496e024723167c6b749f067bb7f59244572deda65586a77058b54ef015576699
Status: Downloaded newer image for trufflesecurity/trufflehog:latest
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2025-06-15T06:50:51Z	info-0	trufflehog	running source	{"source_manager_worker_id": "QqqSp", "with_units": true}
2025-06-15T06:50:51Z	info-0	trufflehog	scanning repo	{"source_manager_worker_id": "QqqSp", "unit_kind": "dir", "unit": "/workdir", "repo": "https://github.com/JacksonKuo/blog.git", "base": "75661e167240e900cc3946f19a910aa163484601"}
2025-06-15T06:50:51Z	info-0	trufflehog	finished scanning	{"chunks": 1, "bytes": 2823, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "95.806792ms", "trufflehog_version": "3.89.1", "verification_caching": {"Hits":0,"Misses":0,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":0}}
[main eb8726a] update
 6 files changed, 77 insertions(+), 17 deletions(-)
...
(99%)
```

# Precommit framework

# Pros and Cons

Per repo, global?
performance




# References
[^1]: [https://docs.trufflesecurity.com/block-secrets-from-leaking](https://docs.trufflesecurity.com/block-secrets-from-leaking)

[^2]: [https://docs.github.com/en/enterprise-server@3.17/admin/enforcing-policies/enforcing-policy-with-pre-receive-hooks/about-pre-receive-hooks](https://docs.github.com/en/enterprise-server@3.17/admin/enforcing-policies/enforcing-policy-with-pre-receive-hooks/about-pre-receive-hooks)

[^3]: [https://git-scm.com/docs/githooks#_pre_commit](https://git-scm.com/docs/githooks#_pre_commit)

[^4]: [https://docs.docker.com/reference/cli/docker/container/run/](https://docs.docker.com/reference/cli/docker/container/run/) 

[^5]: [https://trufflesecurity.com/blog/trufflehog-commands-git-vs-filesystem](https://trufflesecurity.com/blog/trufflehog-commands-git-vs-filesystem)

[^6]: [https://github.com/trufflesecurity/trufflehog?tab=readme-ov-file#s3](https://github.com/trufflesecurity/trufflehog?tab=readme-ov-file#s3)