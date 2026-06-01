---
layout: post
title: "GitHub: Ruleset - Code Owners"
date: 2026-5-31
tags: ["github"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Breakdown the branch protection: `Require review from Code Owners`, what it does and what is needed to enable it. Also from time to time, I'm going to abbreviate CODEOWNERS/Code Owners as CO. 

# Breakdown
What is the `Require review from Code Owners` and what's the value. The description of the requirement is: *require an approving review in pull requests that modify files that have a designated code owner*. 

#### Pros
* Ensure not anyone in the org with WRITE perms can approve PRs
* Prevent automation with PR WRITE perms from approving PRs
    * GHApps
    * LLMs
    * Workflows
    * PATs

#### Cons
* Need code owners org-level enforcement 
* If you have base permission set to None, most entities won't have WRITE perms to start out with
* Users with READ-only perms can't approve anymore
* GHApps can't be used in CODEOWNERS[^1], unless the GHApp is acting on behalf of a user
    * GHApps have either installation access token or user access token[^2]

# CODEOWNERS file
* `CODEOWNERS` locations and order of operations: `.github/`, `root`, `docs/`
* `CODEOWNERS` pattern: `@username`, `@org/team-name`, `user@example.com`

> * The people you choose as code owners must have write permissions for the repository
> * Users and teams must have explicit write access to the repository, even if the team's members already have access
> * Each CODEOWNERS file assigns the code owners for a single branch in the repository. Thus, you can assign different code owners for different branches
> * CODEOWNERS paths are case sensitive
> * If any line in your CODEOWNERS file contains invalid syntax, that line will be skipped
> * To protect a repository fully against unauthorized changes, you also need to define an owner for the CODEOWNERS file itself. The most secure method is to define a CODEOWNERS file in the .github directory of the repository and define the repository owner as the owner of either the CODEOWNERS file (/.github/CODEOWNERS @owner_username
> * You cannot use an email address to refer to a managed user account

[CODEOWNERS syntax](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners#example-of-a-codeowners-file)

```yaml
# These owners will be the default owners for everything in the repo. Unless a later match takes precedence,
# @global-owner1 and @global-owner2 will be requested for review when someone opens a pull request.
*       @global-owner1 @global-owner2

# Order is important; the last matching pattern takes the most precedence.
*.js    @js-owner #This is an inline comment.
*.go    docs@example.com
*.txt   @octo-org/octocats

# The `docs/*` pattern will match files like `docs/b.md` but not further nested files like `docs/a/b.md`.
docs/* docs@example.com

# @octocat owns any file in an apps directory anywhere in your repository.
apps/ @octocat

# @doctocat owns any file in the `/docs` directory in the root of your repository and any of its subdirectories.
/docs/ @doctocat

# @octocat owns any file in a `/logs` directory such as `/a/logs`, `/b/logs`, and `/a/c/logs`. Any changes in a `/logs` directory will require approval from @octocat.
**/logs @octocat

# @octocat owns any file in the `/apps` directory in the root of your repository except for the `/apps/github` subdirectory, as its owners are left empty. 
# Without an owner, changes to `apps/github` can be made with the approval of any user who has write access to the repository.
/apps/ @octocat
/apps/github
```

# Scenarios

#### Missing
* No CO
* Invalid CO

First, if there's no CO file or if the file is invalid, then the `Require review from Code Owners` doesn't activate. There's not a working CO to reference, so the check doesn't proc. If a specific line in the file is malformed, according to the docs it only affects that line. 

#### Use Cases
* Monorepos
    * Monorepos are interesting. Paths that have a code owners are protected. Paths that don't have code owners, will accept an approver from anyone with WRITE perms. You could have a wildcard path as a fallthrough mechanism 
* Branches
    * The CO applies to the specific branch. What does this mean? It means the ruleset needs to point to a specific branch
    * `default branch/main/master` - target branches to set
    * Don't set CO Approvals on feature branches, since those don't need branch protections
* Single user repos
    * Ideally single user repos are tied to a GH team
    * Using `@username` wouldn't help since you need an approval that's different than the PR author
* Non-prod repos
    * Hmm, do non-prod repos need code owner approvals? Hmm, well non-prod doesn't need approvals in general, sooo no
* External contributors 
    * A `ext-<org>-<team>` team might work, if there are team structure available. The idea that if someone leaves, the team still owns the repo still makes sense in this case. Would need a mapping of external contributors to external orgs + teams. Optionally, external contributors could be added as a username. The main diff between the two is external-contributors may not be on SSO

#### Ruleset Construction
Assuming we're using custom properties + `Require review from Code Owners`, what would we need?

* Bypass list:
    * Organization admin
* Target repositories
    * Repositories matching a filter
        * `fork:false`
        * `props.codeowners:true`? `[#TODO]`
* Target branches
    * Include default branch
    * Include by pattern
        * `main`
        * `master`
* Branch rules
    * Require a pull request before merging
        * Required approvals: 1
        * Require review from Code Owners

#### Exceptions
* Exemptions
    * Ruleset bypass list: team, apps, roles
* Breakglass
    * Ruleset bypass list: org admin role[^3]
* Forks[^4]
    * *To trigger review requests, pull requests use the version of CODEOWNERS from the base branch of the pull request. The base branch is the branch that a pull request will modify if the pull request is merged.*
    * Fork Repo Filter - `fork:false`. Have the optional exclude
    * But should forks have a CO file? I think so... why would it not? It's public. Could be helpful against pwn requests... Also forks don't import secrets in PRs `[#TODO]`
* Dark forks
    * Dark forks aren't actual forks, so I don't think they need an exception

# Misc
What in the world is this: `Require review from specific teams`: [https://github.blog/changelog/2025-11-03-required-review-by-specific-teams-now-available-in-rulesets/](https://github.blog/changelog/2025-11-03-required-review-by-specific-teams-now-available-in-rulesets/).

> * You can now require approvals from specific teams to merge changes into protected branches based on files and folders
> * This new rule is designed to augment CODEOWNER files but not replace them

 I wonder if this sends an email out. This is probably most helpful for security teams for very specific filenames/paths. 

# References
[^1]: [https://github.com/orgs/community/discussions/23064](https://github.com/orgs/community/discussions/23064)

[^2]: [https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-user-access-token-for-a-github-app](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-user-access-token-for-a-github-app)

[^3]: I believe this is the same as an owner, bc i don't see an org admin role under organization roles > role management

[^4]: [https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners#codeowners-and-forks](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners#codeowners-and-forks)