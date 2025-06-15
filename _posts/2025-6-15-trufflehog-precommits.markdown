---
layout: post
title: "Trufflehog - Precommits Hooks"
date: 2025-6-15
tags: ["devsecops"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's prevent secrets before they're ever committed using Trufflehog pre-commit hooks.

# Overview
So there's actually two ways to prevent commits: pre-receive and pre-commit hooks.[^1] However pre-receive hooks are only if you control the git server. GitHub.com doesn't support pre-received hooks, but the self-hosted GitHub Enterprise Server does.[^2] Trufflehog has a great pre-commit hook guide: 
* [https://github.com/trufflesecurity/trufflehog/blob/main/PreCommit.md](https://github.com/trufflesecurity/trufflehog/blob/main/PreCommit.md)
* [https://docs.trufflesecurity.com/pre-commit-hooks](https://docs.trufflesecurity.com/pre-commit-hooks) 

Let's walk through `Githooks`/`hooksPath` and the pre-commit framework, woohoo!

# Githooks
A couple things for `githooks` - [https://git-scm.com/docs/githooks](https://git-scm.com/docs/githooks): 
* `$GIT_DIR/hooks` is not uploaded to GitHub.com
* *By default the hooks directory is `$GIT_DIR/hooks`, but that can be changed via the `core.hooksPath` configuration variable*
    * `$GIT_DIR/hooks` local to a git repo
* only runs this specific ``$GIT_DIR/hooks/pre-commit` file
* git config[^3] - [https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)
    * local - `git config core.hooksPath ~/.git-hooks`
    * global - `git config --global core.hooksPath ~/.git-hooks`
    * system - `git config --system core.hooksPath ~/.git-hooks`
* *can be bypassed with the `--no-verify` option*[^4]
* *non-zero status from this script causes the git commit command to abort before creating a commit*
* *The default pre-commit hook, when enabled, catches introduction of lines with trailing whitespaces and aborts the commit when such a line is found*
* *Hooks that don’t have the executable bit set are ignored*
* To quickly disable, `mv ~/.git-hooks/pre-commit ~/.git-hooks/pre-commit.disabled`

```bash
# Using Homebrew (macOS)
# brew install trufflehog
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
# Local trufflehog binary
# ~/.git-hooks/pre-commit
trufflehog git file://. --since-commit HEAD --results=verified,unknown --fail
```

```sh
#!/bin/sh
# Docker
# ~/.git-hooks/pre-commit
docker run --rm -v "$(pwd):/workdir" -i trufflesecurity/trufflehog:latest git file:///workdir --since-commit HEAD --results=verified,unknown --fail
```
A breakdown of the docker command: 
* `docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]`[^5]
    * `docker container run` = `docker run`
    * `-rm` = removes container when exits
    * `-v` = mount a volume
    * `-i` = *keeps the container's STDIN open, and lets you send input to the container through standard input*
    * `--fail` = return exit code 183 if results are found
    * `git file://` + `/workdir` = trufflehog git command that scans local git repo
        * note that `git` is preferred over `filesystem`, since `filesystem` can't look into packfiles. The one exception is if the repository is corrupted, then `filesystem` should be used.[^6]
    * exit code[^7]
        * 0 = no errors. when verified secrets are found or no results are found. tested with canaries and live AWS tokens
        * 1 = when an error occurs
        * 183 = `--fail` set and secrets are found

```bash
jacksonkuo@JacksonKuos-MacBook-Air repo-private % git commit -m "update"
Unable to find image 'trufflesecurity/trufflehog:latest' locally
latest: Pulling from trufflesecurity/trufflehog
5245f6332319: Download complete
dee434cacd1b: Download complete
4f4fb700ef54: Already exists
6b6bccb23c39: Download complete
Digest: sha256:496e024723167c6b749f067bb7f59244572deda65586a77058b54ef015576699
Status: Downloaded newer image for trufflesecurity/trufflehog:latest
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2025-06-15T08:18:16Z	info-0	trufflehog	running source	{"source_manager_worker_id": "3Moz4", "with_units": true}
2025-06-15T08:18:16Z	info-0	trufflehog	scanning repo	{"source_manager_worker_id": "3Moz4", "unit_kind": "dir", "unit": "/workdir", "repo": "https://github.com/jkuo-org/repo-private.git", "base": "9faaa8f4fb5e8e61d8cfd87ffb890b851257c583"}
✅ Found verified result 🐷🔑
Detector Type: AWS
Decoder Type: PLAIN
Raw result: xxx
Resource_type: Access key
Account: xxx
User_id: xxx
Arn: arn:aws:iam::xxx:user/trufflehog
Rotation_guide: https://howtorotate.com/docs/tutorials/aws/
Commit: Staged
File: canary.properties
Line: 14
Repository: https://github.com/jkuo-org/repo-private.git
Timestamp: 0001-01-01 00:00:00 +0000

2025-06-15T08:18:16Z	info-0	trufflehog	finished scanning	{"chunks": 1, "bytes": 153, "verified_secrets": 1, "unverified_secrets": 0, "scan_duration": "260.441209ms", "trufflehog_version": "3.89.1", "verification_caching": {"Hits":0,"Misses":1,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":244}}
```

```bash
jacksonkuo@JacksonKuos-MacBook-Air repo-private % git commit -m "update" --no-verify
[main 3086a6a] update
 1 file changed, 6 insertions(+)
```

# Pre-commit framework
A couple things for `pre-commit` framework - [https://pre-commit.com/](https://pre-commit.com/):
* *Every time you clone a project using pre-commit running pre-commit install should always be the first thing you do*
* *If you want to manually run all pre-commit hooks on a repository, run pre-commit run --all-files. To run individual hooks use pre-commit run \<hook_id\>*
* *new in 3.2.0: The values of stages match the hook names. Previously, commit, push, and merge-commit matched pre-commit, pre-push, and pre-merge-commit respectively.*

```sh
# .pre-commit-config.yaml in repo root
repos:
  - repo: local
    hooks:
      - id: trufflehog
        name: TruffleHog
        description: Detect secrets in your data.
        entry: bash -c 'docker run --rm -v "$(pwd):/workdir" -i --rm trufflesecurity/trufflehog:latest git file:///workdir --since-commit HEAD --results=verified,unknown --fail'
        language: system
        stages: ["pre-commit", "pre-push"]
```
* `repos` - *a list of repository mappings*
    * `repo` - *repository url to git clone from or one of the special sentinel values: local, meta.*[^8]
    * `hooks`[^9]
        * `id` - *which hook from the repository to use*
        * `name` - *(optional) override the name of the hook - shown during hook execution*
        * `entry` - *the entry point - the executable to run*
        * `language` - *the language of the hook - tells pre-commit how to install the hook*
            * `system` - *system hooks provide a way to write hooks for system-level executables which don't have a supported language*
        * `stages` - *(optional) selects which git hook(s) to run for*

```bash
brew install pre-commit
git config --global --unset-all core.hooksPath
pre-commit install
# pre-commit install --overwrite
# pre-commit run --all-files, only won't due to the --since-commit HEAD
```

```bash
jacksonkuo@JacksonKuos-MacBook-Air repo-private % git commit -m "labels"
TruffleHog...............................................................Failed
- hook id: trufflehog
- exit code: 183

🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2025-06-15T09:50:15Z	info-0	trufflehog	running source	{"source_manager_worker_id": "07asZ", "with_units": true}
2025-06-15T09:50:15Z	info-0	trufflehog	scanning repo	{"source_manager_worker_id": "07asZ", "unit_kind": "dir", "unit": "/workdir", "repo": "https://github.com/jkuo-org/repo-private.git", "base": "b6d22bf8ae018920e35647e49b4711bc3ddc96c9"}
✅ Found verified result 🐷🔑
Detector Type: AWS
Decoder Type: PLAIN
Raw result: xxx
Resource_type: Access key
Account: xxx
Rotation_guide: https://howtorotate.com/docs/tutorials/aws/
User_id: xxx
Arn: arn:aws:iam::xxx:user/trufflehog
Commit: Staged
File: canary.properties
Line: 2
Repository: https://github.com/jkuo-org/repo-private.git
Timestamp: 0001-01-01 00:00:00 +0000

2025-06-15T09:50:15Z	info-0	trufflehog	finished scanning	{"chunks": 1, "bytes": 179, "verified_secrets": 1, "unverified_secrets": 0, "scan_duration": "232.504083ms", "trufflehog_version": "3.89.1", "verification_caching": {"Hits":0,"Misses":1,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":216}}
```

```bash
jacksonkuo@JacksonKuos-MacBook-Air repo-private % git commit -m "pre-commit"
TruffleHog...............................................................Passed
[main df1914a] pre-commit
 1 file changed, 10 insertions(+)
 create mode 100644 .pre-commit-config.yaml
```

Something I noticed is that edits to the `canary.properties` file like newlines that move the existing secret around, do not cause Trufflehog to proc. Looks like Trufflehog works off the diff. This is good, since it means old secrets won't trigger unnecessarily. In order to trigger again, both lines for `aws_access_key_id` and `aws_secret_access_key` have to be changed at the same time or `--since-commit HEAD` has to be removed.

Also Trufflehog uses `AKIA` to find a AWS API token. If we want to explicitly ignore a credential then `#trufflehog:ignore` needs to be placed on the `AKIA` line. 

# Pros and Cons
So which one should be used githooks or the pre-commit framework? Note that I'm going to refer to the pre-commit framework as just pre-commits. Githooks are a better fit for individuals. For team projects, pre-commits is preferred. 
* Githooks require bootstrapped shell commands to set global pre-commits, and the per repo `$GIT_DIR/hooks` aren't included when pushing sourcecode to GitHub.com
* Githooks require a root script to properly call child scripts, whereas pre-commits allow for adding multiple hooks via easy built-ins
* Githooks require managing dependencies, where as pre-commit dependencies and virtual environments have built-ins
* Pre-commits also have built-in versioning

```bash
# pre-commit uninstall
git config --global init.templateDir ~/.git-template
pre-commit init-templatedir ~/.git-template
# pre-commit installed at /Users/jacksonkuo/.git-template/hooks/pre-commit
git clone https://github.com/jkuo-org/repo-private.git
cd repo-private

jacksonkuo@JacksonKuos-MacBook-Air repo-private % git commit -m "reorder"
TruffleHog...............................................................Failed
- hook id: trufflehog
- exit code: 183

🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

# git config --global --unset-all init.templatedir
# git config --global --list
# mv .git/hooks/pre-commit .git/hooks/pre-commit.disabled
```
Pre-commits require a unique `.pre-commit-config.yaml` file per repository. And pre-commits do require developers will need to explicitly install pre-commits and have `pre-commit install` executed. But developers can configure their `git` to point to a template that will auto call `.pre-commit-config.yaml` so that `pre-commit install` isn't required after every `git clone`.[^10] Alternatively, `pre-commit install` can just be part of the `Makefile` build process. 

# Performance
Performance wise, the first instance of downloading the Trufflehog container will be a bit slow, as well as any version updates. Running the Trufflehog binary may have better performance. The risk of supply chain attacks on Trufflehog warrant commit pinning along with frequent version updates from the security team. The best approach would be to have a version check inside the pre-commit and if an update is required call the installation script[^11] or download directly from Github Releases[^12]

#### Local binary vs docker container
```yml
repos:
  - repo: local
    hooks:
      - id: trufflehog-binary
        name: TruffleHog-Binary
        description: Detect secrets in your data.
        entry: bash -c 'trufflehog git file://. --since-commit HEAD --results=verified,unknown --fail'
        language: system
        stages: ["pre-commit", "pre-push"]
      - id: trufflehog-docker
        name: TruffleHog-Docker
        description: Detect secrets in your data.
        entry: bash -c 'docker run --rm -v "$(pwd):/workdir" -i --rm trufflesecurity/trufflehog:latest git file:///workdir --since-commit HEAD --results=verified,unknown --fail'
        language: system
        stages: ["manual"]
        #stages: ["pre-commit", "pre-push"]
```

* binary = `time git commit -m "update"`
    * *git commit -m "update"  0.84s user 0.17s system 73% cpu 1.374 total*
    * *git commit -m "update"  0.84s user 0.18s system 73% cpu 1.388 total*
* docker = `time git commit -m "update"`
    * *git commit -m "update"  0.12s user 0.09s system 5% cpu 3.606 total*
    * *git commit -m "update"  0.11s user 0.08s system 5% cpu 3.255 total*
    * *git commit -m "update"  0.12s user 0.08s system 8% cpu 2.367 total*

Based on the `time` results, looks like the docker container takes approximately 2-3x longer. 

# References
[^1]: [https://docs.trufflesecurity.com/block-secrets-from-leaking](https://docs.trufflesecurity.com/block-secrets-from-leaking)

[^2]: [https://docs.github.com/en/enterprise-server@3.17/admin/enforcing-policies/enforcing-policy-with-pre-receive-hooks/about-pre-receive-hooks](https://docs.github.com/en/enterprise-server@3.17/admin/enforcing-policies/enforcing-policy-with-pre-receive-hooks/about-pre-receive-hooks)

[^3]: [https://medium.com/@yadavprakhar1809/understanding-the-three-levels-of-git-config-local-global-and-system-e95c26aac8ee](https://medium.com/@yadavprakhar1809/understanding-the-three-levels-of-git-config-local-global-and-system-e95c26aac8ee)

[^4]: [https://git-scm.com/docs/githooks#_pre_commit](https://git-scm.com/docs/githooks#_pre_commit)

[^5]: [https://docs.docker.com/reference/cli/docker/container/run/](https://docs.docker.com/reference/cli/docker/container/run/) 

[^6]: [https://trufflesecurity.com/blog/trufflehog-commands-git-vs-filesystem](https://trufflesecurity.com/blog/trufflehog-commands-git-vs-filesystem)

[^7]: [https://github.com/trufflesecurity/trufflehog?tab=readme-ov-file#s3](https://github.com/trufflesecurity/trufflehog?tab=readme-ov-file#s3)

[^8]: [https://pre-commit.com/#pre-commit-configyaml---repos](https://pre-commit.com/#pre-commit-configyaml---repos)

[^9]: [https://pre-commit.com/#pre-commit-configyaml---hooks](https://pre-commit.com/#pre-commit-configyaml---hooks)

[^10]: [https://pre-commit.com/#automatically-enabling-pre-commit-on-repositories](https://pre-commit.com/#automatically-enabling-pre-commit-on-repositories)

[^11]: [https://github.com/trufflesecurity/trufflehog?tab=readme-ov-file#using-installation-script-to-install-a-specific-version](https://github.com/trufflesecurity/trufflehog?tab=readme-ov-file#using-installation-script-to-install-a-specific-version)

[^12]: [https://github.com/trufflesecurity/trufflehog/releases](https://github.com/trufflesecurity/trufflehog/releases)