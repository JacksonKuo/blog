---
layout: post
title: "GitHub - Forks and Dark Forks"
date: 2026-6-7
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Breakdown forks, fork security, and dark forks. When do use them and why. And then how to develop around them with branch protections enabled. Most of GitHub fork documentation is here: [https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks)

# Terminology 
* Forks: *repository that shares code and visibility settings with the original “upstream” repository*[^1]
* Upstream: a common name used for a remote that points to repo that was forked
* Origin: the default name of remote[^3] when you git clone. similiar idea to `main` being the default branch name
* Remote: a URL that points to a remote repo. has an alias used to reference the url
* Fork network: *All repositories belong to a repository network. A repository network contains the upstream repository, the upstream repository's direct forks, and all forks of those forks.*[^4]
* Dark Forks: also called a mirror repo or manual fork. A repo that points to the upstream repo via `git remote add` but is not created through the `Create a new fork` button

# Security
Making a copy of the GitHub fork [security considerations](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-permissions-and-visibility-of-forks#important-security-considerations):
> If you work with forks, or if you're the owner of a repository or organization that allows forking, it's important to be aware of the following security considerations.
> * Forks have their own permissions separate from the upstream repository.
> * The owners of a repository that has been forked have read permission to all forks in the repository's network.
> * Organization owners of a repository that has been forked have admin permission to forks created in personal user namespaces, including the ability to delete the fork and its branches.
> Organization owners of a repository that has been forked have read permission to forks created in organizations, but do not have the ability to delete the fork or its branches.
> * Forks created in another organization will not be deleted when individual access is removed from the upstream repository.
> * Commits to any repository in a network can be accessed from any repository in the same network, including the upstream repository, even after a fork is deleted.

The last line is very important. Trufflehog had a great post about this: [https://trufflesecurity.com/blog/anyone-can-access-deleted-and-private-repo-data-github](https://trufflesecurity.com/blog/anyone-can-access-deleted-and-private-repo-data-github). The takeaway is:
> As long as one fork exists, any commit to that repository network (ie: commits on the “upstream” repo or “downstream” forks) will exist forever.

#### Tradeoffs

|  | Pro | Cons |
|---|---|---|---|
| Fork | No Secrets | Public<br>Fork Network | 
| Dark Fork | Non-Public | Secrets |

Dark forks are preferred because they're not on the internet. An attacker can't create a PR against a fork dark and/or attempt the typical tried and true github workflow attack vectors. And secrets committed to the dark fork aren't exposed via the fork network, aka the internet. 

The downsides is that dark forks don't the normal secret protection that regular forks have. Secrets are directly passed to dark forks, whereas they're not inherited for regular forks. So a dark fork workflow can dump secrets without a `pull_request_target`. That said, regular forks can still expose secrets via `pull_request_target`. And the dark fork would need to enumerate your companies secret names first and then get a malicious workflow into the upstream repository, which is a non-trivial barrier. And it would make the attack specific to a single company, which is not great value.

# Git Operations
clone - [https://git-scm.com/docs/git-clone](https://git-scm.com/docs/git-clone).
* `git clone --origin <name> <repo-url> <dir>`
    * `--origin = instead of using the remote name "origin", use <name> to keep track of the upstream repository`

branch - [https://git-scm.com/docs/git-branch](https://git-scm.com/docs/git-branch)
* `git branch --show-current`
    * show current branch, i.e. main
* `git branch <branch-name> [<start-point>]`
    * this creates a new branch called `<branch-name>` using the `<start-point>` commit
    * does not move your current `HEAD`
* `<start-point>`
    * *The new branch head will point to this commit. It may be given as a branch name, a commit-id, or a tag. If this option is omitted, the current HEAD will be used instead.*
* `HEAD`
    * *HEAD file is a symbolic reference to the branch you’re currently on*[^5]
* `CODEOWNERS` location
    * `/CODEOWNERS`
    * `.github/CODEOWNERS`
    * `docs/CODEOWNERS`

commit - [https://git-scm.com/docs/git-commit](https://git-scm.com/docs/git-commit)
* `git commit --allow-empty -s -m "message"`
    * `-s` - *Add a Signed-off-by trailer by the committer at the end of the commit log message.* common in open source

push - [https://git-scm.com/docs/git-push](https://git-scm.com/docs/git-push)
* `--follow-tags`
    * basically add tags to the fork
* `--all`
    * seems to push branches in alphabetical order

# Setup
We're working with the model that PR branch protections are enabled, but only on `main`,`master`, and `default_branch`. Feature branches don't require PR approvals. 

#### Dark Fork
The goal is two branches, with the `main` branch where you add custom code. And the `upstream` branch 

* `main` branch
* `upstream` branc

safety for updates

* branch -> main (default)
* branch -> upstream-main
* upstream remote -> original_repo
* origin remote -> owner_repo


#### Fork

# References
[^1]: [https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo)

[^2]: [https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/configuring-a-remote-repository-for-a-fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/configuring-a-remote-repository-for-a-fork)

[^3]: [https://sentry.io/answers/determine-the-origin-of-a-cloned-git-repository/](https://sentry.io/answers/determine-the-origin-of-a-cloned-git-repository/)

[^4]: [https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-permissions-and-visibility-of-forks#about-visibility-of-forks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-permissions-and-visibility-of-forks#about-visibility-of-forks)

[^5]: [https://git-scm.com/book/ms/v2/Git-Internals-Git-References](https://git-scm.com/book/ms/v2/Git-Internals-Git-References)

[^6]: [https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/detaching-a-fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/detaching-a-fork)