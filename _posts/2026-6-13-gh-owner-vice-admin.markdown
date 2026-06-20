---
layout: post
title: "GitHub Defense: Part VI - Vice Admin"
date: 2026-6-13
tags: ["github"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
How do we create a custom role that still gives access to most org and repo privileges but excludes super sensitive actions.

Hmm, what's a good term for this type of account? Vice-admin or vice-owner could work.

# General
In a nutshell what we need is a custom role that excludes the following permissions
* Organization Admin
* Role Management
* Member Management

The follow are the permissions available via roles:
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
    * Repository
    * Security
* Repository Permissions
    * Base repository role: Admin

It's wild that even with everything checked in Roles, there's still a ton of functionality that isn't included. Note it's easier to manage assignment using a dedicated team. I created a ViceAdmin team with the following permissions:
* Default Roles 
    * All repository admin
    * Apps Manager
    * CI/CD Admin
    * Security Manager
* Organization Permissions 
    * All enabled
* Repository Permissions
    * Base repository role: Admin

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/github-viceadmin.png){: width="250"}
{: refdef}
{:refdef: style="text-align: center;"}
\[ViceAdmin Access\]
{: refdef}

# Questions and Comments
* Many high-level permissions are only granted to Owners
* Custom role permissions analysis[^1]
    * `Manage custom organization roles`: even if this is enabled, because custom roles don't give the ability to change membership or add default roles, there's no way to privilege escalation to owner... i'm pretty sure. You can only create a new custom role or edit an existing custom role
    * `Manage organization credentials`: I honestly can't find documentation for this...
    * `Manage organization OAuth application policies`: Not sure if this is a privilege escalation mechanism, a future todo...
* I don't see Moderator or Billing Managers, where are they?
    * Settings > Billing and licensing[^2]
    * Settings > Access > Moderation > Moderators
* What's the difference between triage, maintain and admin?[^3]
    * > Triage: Recommended for contributors who need to proactively manage issues, discussions, and pull requests without write access
    * > Write: Recommended for contributors who actively push to your project
    * > Maintain: Recommended for project managers who need to manage the repository without access to sensitive or destructive actions
* Can repos be undeleted? 
    * A deleted repository can be restored within 90 days[^4]
    * Settings > Archive > Delete repositories
* Can orgs be undeleted?
    > Deleting your organization account permanently removes all repositories, forks of private repositories, wikis, issues, pull requests, and project or organization pages. This action is irreversible.[^5]
    * Terms and Services[^6]
* Can a GHApp delete the org?
    * Yes, the Delete an organization endpoint[^7] requires `"Administration" organization permissions (write)`
    * Which means that `Apps Manager` is super dangerous
* Note there's a maximum of 20 custom roles[^8] [^9]
* What permissions are needed to promote a member to owner?
    * Create an organization invitation - "Members" organization permissions (write)[^10]
    * Set organization membership for a user - "Members" organization permissions (write)[^11]
* Should maybe consider org-level backups[^12]

# What's Missing
Uh there's a lot. I don't really want to go through them all, but it's a lot just comparing visually. 

| Page 1 | Page 2 |
|:---:|:---:|
| ![]({{ site.baseurl }}/assets/images/github-orgadmin-1.png){: width="250"} | ![]({{ site.baseurl }}/assets/images/github-orgadmin-2.png){: width="250"} |

Just adding pre-defined role table which includes Owners: [https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/permissions-of-predefined-organization-roles#permissions-of-predefined-roles](https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/permissions-of-predefined-organization-roles#permissions-of-predefined-roles)

# Top Gun
* Default Role - Apps Manager
* PAT/GHApp - `"Members" organization permissions (write)`
* PAT/GHApp - `"Administration" organization permissions (write)`

# Lessons Learned
Maxing out a team with a custom role with all permissions and all default roles except App Manager should be safe. 

But does this setup provide me everything I need?
* Available
    * Rulesets
    * Custom properties
    * Secrets and variables
    * Audit logs
* Not available
    * Member privilege
    * Pages
    * Authentication security

# References
[^1]: [https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/permissions-of-custom-organization-roles#permissions-for-organization-access](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/permissions-of-custom-organization-roles#permissions-for-organization-access)
[^2]: [https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/adding-a-billing-manager-to-your-organization#inviting-a-billing-manager](https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/adding-a-billing-manager-to-your-organization#inviting-a-billing-manager)
[^3]: [https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/repository-roles-for-an-organization#repository-roles-for-organizations](https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/repository-roles-for-an-organization#repository-roles-for-organizations)
[^4]: [https://docs.github.com/en/repositories/creating-and-managing-repositories/restoring-a-deleted-repository#about-repository-restoration](https://docs.github.com/en/repositories/creating-and-managing-repositories/restoring-a-deleted-repository#about-repository-restoration)
[^5]: [https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/deleting-an-organization-account#backing-up-your-organization-content](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/deleting-an-organization-account#backing-up-your-organization-content)
[^6]: [https://docs.github.com/en/site-policy/github-terms/github-terms-of-service#2-upon-cancellation](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service#2-upon-cancellation)
[^7]: [https://docs.github.com/en/rest/orgs/orgs?apiVersion=2026-03-10#delete-an-organization](https://docs.github.com/en/rest/orgs/orgs?apiVersion=2026-03-10#delete-an-organization)
[^8]: [https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/managing-custom-organization-roles#creating-a-custom-role](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/managing-custom-organization-roles#creating-a-custom-role)
[^9]: [https://github.blog/changelog/2025-08-21-enterprises-can-create-organization-roles-for-use-across-their-enterprise-and-custom-role-limits-have-been-increased/](https://github.blog/changelog/2025-08-21-enterprises-can-create-organization-roles-for-use-across-their-enterprise-and-custom-role-limits-have-been-increased/)
[^10]: [https://docs.github.com/en/rest/orgs/members?apiVersion=2026-03-10#create-an-organization-invitation](https://docs.github.com/en/rest/orgs/members?apiVersion=2026-03-10#create-an-organization-invitation)
[^11]: [https://docs.github.com/en/rest/orgs/members?apiVersion=2026-03-10#set-organization-membership-for-a-user](https://docs.github.com/en/rest/orgs/members?apiVersion=2026-03-10#set-organization-membership-for-a-user)
[^12]: [https://docs.github.com/en/enterprise-cloud@latest/repositories/archiving-a-github-repository/backing-up-a-repository](https://docs.github.com/en/enterprise-cloud@latest/repositories/archiving-a-github-repository/backing-up-a-repository)