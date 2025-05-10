---
layout: post
title: "Terraform: Multi-Cloud Module"
date: 2025-5-10
tags: ["IaC"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Create a terraform module for `app-springboot` deployment that allows me to easily swap cloud environments.

# Interface
My expected interface is below, effectively one main variable `cloud_provider` which will dictate which cloud platform the infrastructure will be created in. Note that when different cloud environments are introduced the CI/CD pipeline will need to be modified. Currently the pipeline works with a droplet.

```yaml
module springboot_app {
    source = "../../modules/multicloud/cluster"
    providers = {
        digitalocean = digitalocean
        github = github
    }
    cloud_provider = "digitalocean"
}
```

# Design
My previous terraform housed all resources in the same `main.tf` file: [https://jacksonkuo.github.io/blog/2025/04/03/terraform-droplet.html](https://jacksonkuo.github.io/blog/2025/04/03/terraform-droplet.html). The new module is composed of many modules, one root module called `multicloud` and multiple reusable modules. The conditional logic uses the `count` operator to determine if a resource should be created: `count = var.cloud_provider == "digitalocean" ? 1 : 0`.

```yaml
* live
    * prod
        * main.tf
* modules
    * digitaloceans
        * droplet
    * github
        * secrets
    * multicloud
        * cluster
```

A couple issues pointed out by the Terraform: Up & Running 3rd edition book.

> *If you’re using multiple clouds, you’re far better off managing each one in a separate module.*
> *you want each provider to be isolated in its own module so that you can manage it separately and limit the blast radius from mistakes or attackers*

In the real world, a multi-cloud module might be too much of a hassle to maintain, but for my use case, I still want to implement one for the hell of it.

> *Moreover, Terraform doesn’t have great support for dependency ordering between providers*

I'm not sure how much this applies to me. The `do_secrets` module does have a dependency of the `do_droplet` module, which means module call order should be automatically handled by terraform and a non-issue. However I added a `depends_on = [module.eks_cluster]` to be explicit. 

> *Defining provider blocks within reusable modules is an antipattern*

> *you should not define any provider blocks in your reusable modules and instead allow your users to create the provider blocks*

> *Every time you include a provider block in your code, Terraform spins up a new process to run that provider*

I'm only providing Provider blocks in my root module. Also note that `required_providers` is perfectly fine to include in reusable modules and is actually required.[^1] Normal aliases are not needed since each provider is a different cloud environment. However I am running into conflicts where the default `hashicorp` provider pattern is being expected. I suspect this is because I'm using custom providers. To mitigate the errors, I'm passing the `providers` explicitly using configuration aliases from `live/prod` > `multicloud/cluster` > `digitaloceans/droplet` and `github/secrets`. Lastly, keeping providers out of reusable modules is important for performance. Having the provider run once in the root module avoids inadvertently opening up hundreds of processes if/when reusable modules are frequently called.

> *key difference from normal provider aliases is that configuration aliases don’t create any providers themselves; instead, they force users of your module to explicitly pass in a provider for each of your configuration aliases using a providers map*

I do declared `configuration_aliases = [digitalocean.*]` to make passing the provider explicit. And I do label the parent (root) and child (reusable) config aliases just to show how the variables are being mapped and passed through.

```yaml
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
      configuration_aliases = [digitalocean.parent]
    }
    github = {
      source  = "integrations/github"
      version = "~> 6.0"
      configuration_aliases = [github.parent]
    }
  }
}
```

Deployment successful and my multi-cloud module gives me the extensibility for other cloud deployments. 

# References
[^1]: [https://developer.hashicorp.com/terraform/language/modules/develop/providers](https://developer.hashicorp.com/terraform/language/modules/develop/providers)