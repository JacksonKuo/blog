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
* Repository Deletion

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

What's the different between triage, maintain and admin?

Wild the amount of permissions that aren't include even with everything checked, is significant. Maybe if I ad

Note there's a maximum of 20 custom roles[^2][^3]

I wonder can a GHApp delete the org?

# References
[^1]: [https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/permissions-of-custom-organization-roles#permissions-for-organization-access](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/permissions-of-custom-organization-roles#permissions-for-organization-access)

[^2]: [https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/managing-custom-organization-roles#creating-a-custom-role](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-peoples-access-to-your-organization-with-roles/managing-custom-organization-roles#creating-a-custom-role)

[]