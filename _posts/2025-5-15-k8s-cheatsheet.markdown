---
layout: post
title: "K8s: Hardening Cheatsheet"
date: 2025-5-15
tags: ["k8s"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
The do and don't for k8 security. I always end up forgetting the exact k8s security issues I usually look for during a pentest, hence the cheatsheet.

# K8s Hardening
I'm ignoring attacks against the control plane and assuming control plane security is handled by the cloud provider. If we want a nice mnemonic to help remember the various hardening areas, we can use the acronym: `SCAN-PIL`:
* `(S)ecrets, (C)ontrol plane, (A)uthN/authZ, (N)etwork`
* `(P)od security, (I)mages, (L)ogging`

#### Cheatsheet
* secrets
    * Don't store secrets in config maps [^1]
    * Don't store secrets in manifests
* control plane
* authN/authZ
    * rbac
        * namespaces
        * risky roles
            * system:anonymous user / system:unauthenticated
            * system:masters (bypasses RBAC)[^2]
            * default:view
        * risky permissions[^3] [^4]
            * Roles and RoleBindings instead of ClusterRoles and ClusterRoleBindings
            * Any verb or resource with `*`
            * Pod creation via Create/Update Deployment, Daemonsets, Statefulsets, Replicationcontrollers, Replicasets, Jobs and Cronjobs[^5]
            * need both `create:rolebindings` and `bind:clusterroles/roles` in order to assign a role[^6]
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
    * Default network policies within each namespace[^7]
* pod security context[^8]
    * non-root *(runAsUser/runAsGroup as non-0 value)*
        ```
        securityContext:
            runAsUser: 10001
            runAsGroup: 10001
        ```
    * privileged setting *(removes the container security isolation)*
        ```
        securityContext:
            privileged: false
        ```
    * capabilities *(container)*
        ```
        securityContext:
            capabilities:
                drop:
                    - all
                add:
                    - CHOWN
        ``` 
        * `CAP_SYS_ADMIN` is equivalent to root
    * read-only filesystem
        ```
        securityContext:
            readOnlyRootFilesystem: true
        ```
        * if you need to write temp files then: *mount an emptyDir volume into the container, which will allow files to be written to a location and then removed automatically when the container is destroyed*[^9]
    * privilege escalation *(can child process gain more privileges than its parent)*
        ```
        securityContext:
            allowPrivilegeEscalation: false
        ```
    * seccomp *(locks down specific Linux syscalls)*[^10]
        ```
        securityContext:
            seccompProfile: 
                type: RuntimeDefault
        ```        
    * Resources no cpu or memory limits *(Mi = millicpus, 250m/1000m = 25%)*
        ```
        resources:
            requests:
                memory:"64Mi"
                cpu:"250m"
            limits:
                memory:"128Mi"
                cpu: "500m"
        ```
    * imageTag *(declare a explicit version via tag or hash, else latest can change anytime)*
        
        ```
        containers:
        - name: app
          image: ubuntu:20.04
        ```
    * Pod Security Standards[^11]
        * Privileged - no restrictions
        * Baseline - weak restrictions
        * Restricted - strong restrictions
    * `automountServiceAccountToken: false`
        * Should be set on service accounts and pods
    * volume mounts
    * pod execution *(kubectl exec)*
        * remove the `create` verb for the `pods/exec` resource
* images
    * unpatched containers
    * signed images
    * authorized images
* logging

# General Literature
* Raesene / Aquasec
    * [https://raesene.github.io/](https://raesene.github.io/)
    * [https://raesene.github.io/categories/index.html](https://raesene.github.io/categories/index.html)
    * [https://www.aquasec.com/authors/rory-mccune/](https://www.aquasec.com/authors/rory-mccune/)
* NCC Group
    * [https://www.nccgroup.com/sg/research-blog/auditing-k3s-clusters/](https://www.nccgroup.com/sg/research-blog/auditing-k3s-clusters/)
* Kubernetes
    * [https://kubernetes.io/docs/tasks/configure-pod-container/security-context/](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
    * [https://kubernetes.io/docs/concepts/security/security-checklist/](https://kubernetes.io/docs/concepts/security/security-checklist/)
    * [https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/)
    * [https://kubernetes.io/docs/concepts/security/rbac-good-practices/](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)
* NSA
    * [https://www.nccgroup.com/us/research-blog/nsa-cisa-kubernetes-security-guidance-a-critical-review/](https://www.nccgroup.com/us/research-blog/nsa-cisa-kubernetes-security-guidance-a-critical-review/)
    * [https://www.nsa.gov/Press-Room/News-Highlights/Article/Article/2716980/nsa-cisa-release-kubernetes-hardening-guidance/](https://www.nsa.gov/Press-Room/News-Highlights/Article/Article/2716980/nsa-cisa-release-kubernetes-hardening-guidance/)
* Misc
    * [https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Kubernetes_Security_Cheat_Sheet.html)
    * [https://github.com/jatrost/awesome-kubernetes-threat-detection](https://github.com/jatrost/awesome-kubernetes-threat-detection)

# Helpful kubectl commands
```
kubectl get clusterroles
kubectl get clusterrolebindings
kubectl get roles -n default
kubectl get rolebindings

kubectl describe clusterrolebinding 
```

# Thoughts and Future Ideas
Note this list is just the tip of the iceberg, is incomplete in some areas, and is fairly vendor agnostic. There are cloud vendor specific hardening that is not included, like: Azure Cluster administrator users without PIM[^12]. But regardless, this is a good base template that'll be more helpful than me trying to remember everything off the cuff. 

Some future projects I plan on doing, include:
* practical privilege escalation and attack chains
* writing a custom controller or operator
* hardening my k3/k8s infrastructure

# References
[^1]: [https://www.aquasec.com/blog/kubernetes-configmap-secrets/](https://www.aquasec.com/blog/kubernetes-configmap-secrets/), [rke](https://github.com/rancher/rke/issues/1024)

[^2]: [https://www.aquasec.com/blog/kubernetes-authorization/](https://www.aquasec.com/blog/kubernetes-authorization/)

[^3]: [https://www.cyberark.com/resources/threat-research-blog/securing-kubernetes-clusters-by-eliminating-risky-permissions](https://www.cyberark.com/resources/threat-research-blog/)

[^4]: API groups explanation: if you want to `get` or `list` pods, you need to include the API group that has pods in it, which is the `""` core group: [https://medium.com/@Vishwa22/kubernetes-api-groups-explained-like-youre-5-why-they-matter-with-real-examples-e2d4338b91b4](https://medium.com/@Vishwa22/kubernetes-api-groups-explained-like-youre-5-why-they-matter-with-real-examples-e2d4338b91b4)

[^5]: [https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-1](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-1)

[^6]: This two-check system can be a little confusing. `create:rolebinding` lets you create a rolebinding object. But just because you can create a rolebinding object, doesn't mean you assign any role to a user. Your user needs `bind` permissions declared on every specific role that you want to be able to attach out. 

[^7]: NetworkPolicies are scoped by namespaces: [https://kubernetes.io/docs/concepts/security/security-checklist/#network-security](https://kubernetes.io/docs/concepts/security/security-checklist/#network-security)

[^8]: [https://www.aquasec.com/blog/kubernetes-hardening-techniques/](https://www.aquasec.com/blog/kubernetes-hardening-techniques/)

[^9]: [https://kubernetes.io/docs/concepts/storage/volumes/#emptydir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

[^10]: [https://medium.com/@rifewang/kubernetes-seccomp-apparmor-selinux-pod-security-standards-and-admission-control-c2d9b7c56031](https://medium.com/@rifewang/kubernetes-seccomp-apparmor-selinux-pod-security-standards-and-admission-control-c2d9b7c56031). Seccomp is for syscalls, while AppArmor (path-based ACL) and SELinux (label-based ACLs) are for files and resources. Ideally all three should be used. 

[^11]: Pod Security Standards are enforced by PodSecurity a K8s Admission controller and not Pod Security Policies (PSP) any more

[^12]: [https://www.secura.com/services/information-technology/vapt/what-can-be-pentested/crystal-box-kubernetes-pentesting/top-10-kubernetes-findings](https://www.secura.com/services/information-technology/vapt/what-can-be-pentested/crystal-box-kubernetes-pentesting/top-10-kubernetes-findings)