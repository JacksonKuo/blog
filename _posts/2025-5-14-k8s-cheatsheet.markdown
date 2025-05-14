---
layout: post
title: "K8s: Hardening Cheatsheet"
date: 2025-5-13
tags: ["k8s"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
The do and don't for k8 security. I always end up forgetting the various guidance off the top of my head, hence the cheatsheet.

# K8s Hardening
Ignore attacks against the control plane and assume control plane security is handled by the cloud provider. If we want a nice mnemonic to remember, we can use SCANPIL:

(S)ecrets, (C)ontrol plane, (A)uthN/authZ, (N)etwork, (P)od security, (I)mages, (L)ogging

* secrets
    * Don't store secrets in config maps [^1] [^2]
    * Don't store secrets in manifests
* control plane
* authN/authZ
    * rbac
        * namespaces
        * risky roles
            * system:anonymous user / system:unauthenticated
            * system:masters (bypasses RBAC)[^2]
        * risky permissions[^3]
            * Roles and RoleBindings instead of ClusterRoles and ClusterRoleBindings
            * Any verb or resource with '*'
            * Pod creation via Create/Update Deployment, Daemonsets, Statefulsets, Replicationcontrollers, Replicasets, Jobs and Cronjobs[^4]
            * need both `create:rolebindings` and `bind:clusterroles/roles` in order to assign a role[^5]
            * | Verb | Resource |
    |---|---|
    | list | secrets | 
    | create | pods | 
    | impersonate | users/groups | 
    | get | secrets | 
    | create | rolebindings | 
    | bind | clusterroles/roles |    
* network
    * Unrestricted egress from Kubernetes clusters
    * Kubernetes API accessible from the internet
* pod security context
    * Readonly filesystem
    * Resources no cpu or memory limits
    * seccomp
    * automountServiceAccountToken: false
    * non-root / rootless
    * volume mounts
    * capabilities
* images
    * unpatched containers
    * signed images
    * authorized images
* logging
      

Cluster administrator users without PIM

# Thoughts
Admission Controller
* Pod Security Standards
* Pod Security Policies > Pod Security Admission

* Authentication
* Authorization
* Admission Controller

# General Literature

* Raesene / Aquasec
    * [https://raesene.github.io/](https://raesene.github.io/)
    * [https://www.aquasec.com/authors/rory-mccune/](https://www.aquasec.com/authors/rory-mccune/)
    * [https://raesene.github.io/categories/index.html](https://raesene.github.io/categories/index.html)
* NCC Group
    * [https://www.nccgroup.com/sg/research-blog/auditing-k3s-clusters/](https://www.nccgroup.com/sg/research-blog/auditing-k3s-clusters/)
* Kubernetes
    * [https://kubernetes.io/docs/tasks/configure-pod-container/security-context/](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
    * [https://kubernetes.io/docs/concepts/security/security-checklist/](https://kubernetes.io/docs/concepts/security/security-checklist/)
* NSA
    * [https://www.nccgroup.com/us/research-blog/nsa-cisa-kubernetes-security-guidance-a-critical-review/](https://www.nccgroup.com/us/research-blog/nsa-cisa-kubernetes-security-guidance-a-critical-review/)
    * [NSA K8s Hardening Guidance 1.2](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
* Misc
    * [https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)
    * [https://github.com/jatrost/awesome-kubernetes-threat-detection](https://github.com/jatrost/awesome-kubernetes-threat-detection)
    * [https://www.secura.com/services/information-technology/vapt/what-can-be-pentested/crystal-box-kubernetes-pentesting/top-10-kubernetes-findings](https://www.secura.com/services/information-technology/vapt/what-can-be-pentested/crystal-box-kubernetes-pentesting/top-10-kubernetes-findings)

# References

[^1]: [https://www.aquasec.com/blog/kubernetes-configmap-secrets/](https://www.aquasec.com/blog/kubernetes-configmap-secrets/), [https://github.com/rancher/rke/issues/1024](https://github.com/rancher/rke/issues/1024)

[^2]: [https://www.aquasec.com/blog/kubernetes-authorization/](https://www.aquasec.com/blog/kubernetes-authorization/)

[^3]: [https://www.cyberark.com/resources/threat-research-blog/securing-kubernetes-clusters-by-eliminating-risky-permissions](https://www.cyberark.com/resources/threat-research-blog/)

[^4]: [https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-1](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-1)

[^5]: This two-check system can be a little confusing. `create:rolebinding` lets you create a rolebinding object. But just because you can create a rolebinding object, doesn't mean you assign any role to a user. Your user needs `bind` permissions declared on every specific role that you want to be able to attach out. 


