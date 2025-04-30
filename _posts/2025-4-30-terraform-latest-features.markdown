---
layout: post
title: "Terraform: Latest Features"
date: 2025-4-30
tags: ["IaC"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
A few years I read the [Terraform: Up & Running 2nd edition book](https://www.amazon.com/Terraform-Running-Writing-Infrastructure-Code/dp/1098116747). The 2nd edition came out September 2019 and covers update to Terraform 0.12. A third edition came out September 26, 2022 with significant updates covering up to Terraform 1.2. And the current version of Terraform is 1.11.4. My understanding of the latest Terraform could use a refresher, let's figure out what the new features are.

# Latest Features
The 3rd edition has a nice section covering new features: *Changes from the Second Edition to the Third Edition*. Some significant changes include[^1]:
* `count` and `for_each` with modules
* `moved` blocks
* secret mangement
* multiple providers

The following will be rote walkthrough of the new sections of Terraform: Up & Running 3rd edition, along with any definitions or interesting points that standout.

#### Chapter 5 -  Moved Blocks

Relevant section located in the *Refactoring Can Be Tricky* > *Refactoring may require changing state section*. 

* `terraform state mv <ORIGINAL_REFERENCE> <NEW_REFERENCE>`
* `terraform state mv aws_security_group.instance aws_security_group.cluster_instance`
* *If you rename an identifier and run this command, you’ll know you did it right if the subsequent terraform plan shows no changes*
* `moved` blocks, to update state automatically

```js
moved {
    from = aws_security_group.old_name
    to = aws_security_group.new_name
}
```

#### Chapter 6 - Managing Secrets with Terraform

* Secret Management Tools with Terraform
    * Providers
        * Environment Variables
        * *1Password offers a CLI tool called op*
            * *secret manager that supports a CLI interface*
            * `eval $(op signin my)`
            * `export AWS_ACCESS_KEY_ID=$(op get item 'aws-dev' --fields 'id')`
            * `export AWS_SECRET_ACCESS_KEY=$(op get item 'aws-dev' --fields 'secret')`
        * aws-vault
            * `aws-vault add dev`
            * `aws-vault exec dev -- terraform apply`
            * *exec command automatically uses AWS STS to fetch temporary credentials*
        * CircleCI
            * *store the credentials for that machine user in CircleCI in what’s called a CircleCI Context*
            * 
                ```yaml
                # Expose secrets in the CircleCI context as environment variables
                context:
                    - example-context
                ```
        * Github Actions
            * OIDC
            * *first step is to create an IAM OIDC identity provider*
            * `aws_iam_openid_connect_provider` resource
            * *trust the GitHub Actions thumbprint, fetched via the `tls_certificate` data source*
            * *create IAM roles*
            * *don’t accidentally allow all GitHub repos to authenticate to your AWS account!*
            * 
                ```yaml
                permissions:
                    id-token: write
                ```
            * *`configure-aws-credentials` action*
    * Resources and data sources
        * `export commands shown here have a leading space: if you start your command with a space, most shells will skip writing that command to the history file.`
        * Environment variables
            * 
                ```yaml
                variable "db_username" {
                    description = "The username for the database"
                    type = string
                    sensitive = true
                }
                ```
            * *`sensitive = true` to indicate they contain secrets (so Terraform won’t log the values when you run plan or apply)* (still stores in state file)
        * Encrypted files
            * *create a KMS Customer Managed Key (CMK), which is an encryption key that AWS manages for you*
            * *first have to define a key policy, which is an IAM Policy that defines who can use that CMK*
            * 
                ```yaml
                data "aws_caller_identity" "self" {}
                
                data "aws_iam_policy_document" "cmk_admin_policy" {
                    ...
                    identifiers = [data.aws_caller_identity.self.arn]
                    ...
                }

                resource "aws_kms_key" "cmk" {
                    policy = data.aws_iam_policy_document.cmk_admin_policy.json
                }

                resource "aws_kms_alias" "cmk" {
                    name = "alias/kms-cmk-example"
                    target_key_id = aws_kms_key.cmk.id
                }

                data "aws_kms_secrets" "creds" {
                    secret {
                    name = "db"
                    payload = file("${path.module}/db-creds.yml.encrypted")
                    }
                }

                locals {
                    db_creds =
                    yamldecode(data.aws_kms_secrets.creds.plaintext["db"])
                }

                local.db_creds.username
                ```
            * *good practice to also create a human-friendly alias for your CMK using the aws_kms_alias resource*
            * *Once you’ve created the CMK, you can start using it to encrypt and decrypt data... you’ll never be able to see... the underlying encryption key.*
            * *`aws kms encrypt` command and write to... `db-creds.yml.encrypted`*
            * sops
                * *`sops <FILE>`, sops will decrypt FILE... and open your text editor...on exit, sops will automatically encrypt the contents.*
                * *Terraform doesn’t yet have native support for... sops... third-party provider such as carlpett/sops*
            * drawbacks
                *Storing secrets is harder... have to run lots of commands or use an external tool*
                * *Integrating with automated tests is harder*
                * *rotating and revoking secrets is hard*
                * *ability to audit who accessed secrets is minimal*
        * Secret stores
            * AWS Secrets Manager
                * *secrets are in a JSON format*
                * 
                ```yaml
                    data "aws_secretsmanager_secret_version" "creds" {
                        secret_id = "db-creds"
                    }

                    locals {
                        db_creds = jsondecode(
                            data.aws_secretsmanager_secret_version.creds.secret_string
                        )
                    }
                ```
    * State files and plan files
        * state files
            * *any secrets you pass into your Terraform resources and data sources will end up in plain text in your Terraform state file*
            * *Store Terraform state in a backend that supports encryption*
            * *Strictly control who can access your Terraform backend*
        * plan files
            * *can store the output of the plan command (the “diff”) in a file*
            * `terraform plan -out=example.plan`
            * *apply command on this saved plan file to ensure that Terraform applies exactly the changes you saw originally*
            * `terraform apply example.plan`
            * *any secrets you pass into your Terraform resources and data sources will end up in plain text in your Terraform plan files*
            * *encrypt your Terraform plan files*

#### Chapter 7 - Working with Multiple Providers

* 
    ```yaml
    provider "aws" {
        region = "us-east-2"
    }
    ```
* Working with one provider
    * What is a provider?
        * *Terraform providers are plugins for the Terraform core. Each plugin is
written in Go to implement a specific interface... designed to work with some platform in the outside world... Terraform core communicates with plugins via remote procedure calls (RPCs)*
        * *Each provider claims a specific prefix... `aws_` prefix*
    * How do you install providers?
        * `terraform init`, Terraform automatically downloads the code for the provider
        * `required_providers` block
        * 
            ```yaml
            terraform {
                required_providers {
                    <LOCAL_NAME> = {
                    source = "<URL>"
                    version = "<VERSION>"
                    }
                }
            }
            ```
        * *local name to use for the provider in this module*
        * `URL = [<HOSTNAME>/]<NAMESPACE>/<TYPE>`
        * *typically only include HOSTNAME for custom providers that you’re downloading from private Terraform Registries*
        * *important to control the version of the provider... recommend always including a `required_providers`*
    * How do you use providers?
        * *`required_providers` block to your code tospecify which provider*
        * *add a `provider` block to configure that provider*
        * *Once you’ve configured a provider, all the resources and data sources from that provider...will automatically use that configuration*
* Working with multiple copies of the same provider
    * Working with multiple AWS regions
        * 
            ```yaml
            provider "aws" {
                region = "us-east-2"
                alias = "region_1"
            }
            provider "aws" {
                region = "us-west-1"
                alias = "region_1"
            }
            ```
        * *alias is a custom name for the provider... can explicitly pass to individual resources, data sources, and modules*
        * 
            ```yaml
            data "aws_region" "region_1" {
                provider = aws.region_1
            }
            output "region_1" {
                value = data.aws_region.region_1.name
                description = "The name of the first region"
            }
            resource "aws_instance" "region_1" {
                provider = aws.region_1
                ...
            }
            ```
        * *manage these AMI IDs manually is tedious... use the `aws_ami` data source... find AMI IDs for you automatically*
        * *To check... really deploying into different regions, add output variables that show you which availability zone, `aws_instance.region_1.availability_zone`*
        * *Notice that with modules, the `providers` (plural) parameter is a map, whereas with resources and data sources, the `provider` (singular) parameter is a single value.*
        * *the providers map you pass to a module, the key must match the local name of the provider in the `required_providers` map within the module*
        * *Use aliases sparingly*
        * *aliases are a good fit when the infrastructure you’re deploying across several aliased providers is truly coupled and you want to always deploy it together*
    * Working with multiple AWS accounts
        * *Cross-account IAM roles are double opt-in*
        * *Use aliases sparingly*
        * *modules that deploy across multiple accounts, if something goes wrong in one account, it affect the other*
    * Creating modules that can work with multiple providers
        * *Reusable modules: low-level modules that are not meant to be deployed directly*
        * *Root modules: high-level modules that combine multiple reusable modules*
        * *Defining provider blocks within reusable modules is an antipattern*
        * *Configuration problems: expose 50 extra variables in a module will make that module very cumbersome to maintain and use*
        * *Performance problems: Every... provider block... spins up a new process... and communicates with that process via RPC.*
        * *should not define any provider blocks in your reusable modules and instead allow your users to create the provider blocks they need solely in their root modules*
        * *configuration aliases: define them in a required_providers*
        * 
            ```yaml
                terraform {
                    required_providers {
                        aws = {
                            source = "hashicorp/aws"
                            version = "~> 4.0"
                            configuration_aliases = [aws.parent, aws.child]
                        }
                    }
                }

                provider "aws" {
                    region = "us-east-2"
                    alias = "parent"
                }
                
                module "multi_account_example" {
                    source = "../../modules/multi-account"
                    providers = {
                        aws.parent = aws.parent
                        aws.child = aws.child
                    }
                }
            ```
        * *key difference from normal provider aliases is that configuration aliases don’t create any providers themselves; instead, they force users of your module to explicitly pass in a provider for each of your configuration aliases using a providers map.*
* Working with multiple different providers
    * *use the AWS Provider with the Kubernetes provider to deploy Dockerized apps*
    * *Elastic Kubernetes Service (EKS) in AWS*
    * *Use multiple providers sparingly*
    * *you want each provider to be isolated in its own module so that you can manage it separately and limit the blast radius from mistakes*
    * *Terraform doesn’t have great support for dependency ordering between providers.*
    * *The root issue lies with the order in which Terraform itself evaluates the provider blocks vs. actual resources.*



# References
[^1]: [https://www.gruntwork.io/blog/terraform-up-running-3rd-edition-early-release-is-now-available](https://www.gruntwork.io/blog/terraform-up-running-3rd-edition-early-release-is-now-available). The blog series is made up of 2 parts, however the link for part 2 no longer seems to work. 