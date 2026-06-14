---
layout: post
title: "GitHub: Owners - Restrict Org Deletion"
date: 2026-6-13
tags: ["github"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
How do we create a custom role that still gives access to most org and repo privileges but excludes super sensitive actions like org deletion.

Hmm, what's a good term for this type of account? Vice-owner or vice-admin could work.

# General
In a nutshell what we need is a custom role that excludes the following permissions
* Organization Deletion
* Role Management

* Default Roles
    * All repository read
    * All repository write
    * All repository triage
    * All repository maintain
    * All repository admin
    * Apps Manager
    * CI/CD Admin
    * Security Manager
* Organization Permissions[^1]
    * Apps and automation
    * CI/CD
    * Copilot
    * General
    * Identity and access management
        * [x] Manage custom organization roles
    * Repository
    * Security
* Repository Permissions
    * Base repository role: Admin

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/github-viceadmin.png){: width="250"}
{: refdef}
{:refdef: style="text-align: center;"}
\[ViceAdmin Access\]
{: refdef}

It's wild that include even with everything checked in role, there's a ton of functionality that isn't included. Note it's easier to manage assignment using a dedicated team. I created

#### Questions and Comments
* Many high-level permissions are only granted to Owners
* Even if `Manage custom organization roles` is enabled, this doesn't grant the ability to privilege escalation to owner, see point above
* I don't see Moderator or Billing Managers, where are they?
* What's the different between triage, maintain and admin?
* Can repos be undeleted? 
    A deleted repository can be restored within 90 days[^2]
* Can a org be undeleted?
    > Deleting your organization account permanently removes all repositories, forks of private repositories, wikis, issues, pull requests, and project or organization pages. This action is irreversible.[^3]
* Can a GHApp delete the org?
    * Yes, the Delete an organization[^4] requires `"Administration" organization permissions (write)`
* Note there's a maximum of 20 custom roles[^5][^6]
* What are the things that are still missing?


# References
[^1]: [https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/permissions-of-custom-organization-roles#permissions-for-organization-access](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/permissions-of-custom-organization-roles#permissions-for-organization-access)

[^2]: [https://docs.github.com/en/repositories/creating-and-managing-repositories/restoring-a-deleted-repository#about-repository-restoration](https://docs.github.com/en/repositories/creating-and-managing-repositories/restoring-a-deleted-repository#about-repository-restoration)

[^3]: [https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/deleting-an-organization-account#backing-up-your-organization-content](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/deleting-an-organization-account#backing-up-your-organization-content)

[^4]: [https://docs.github.com/en/rest/orgs/orgs?apiVersion=2026-03-10#delete-an-organization](https://docs.github.com/en/rest/orgs/orgs?apiVersion=2026-03-10#delete-an-organization)

[^5]: [https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/managing-custom-organization-roles#creating-a-custom-role](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/managing-custom-organization-roles#creating-a-custom-role)

[^6]: [https://github.blog/changelog/2025-08-21-enterprises-can-create-organization-roles-for-use-across-their-enterprise-and-custom-role-limits-have-been-increased/](https://github.blog/changelog/2025-08-21-enterprises-can-create-organization-roles-for-use-across-their-enterprise-and-custom-role-limits-have-been-increased/)