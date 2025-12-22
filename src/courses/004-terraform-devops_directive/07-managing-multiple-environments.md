[TOC]

# 07: Managing Multiple Environments

<img src="./images/multiple_environments.png" alt="Multiple Environments" style="zoom:60%;" />

We want to take our single config or module and deploy it multiple times and there are two main approaches:

## 7.1 Two Main Approaches

### 7.1.1 Workspaces

Multiple named sections within a single backend

```shell
$ terraform workspace new dev
Created and switched to workspace "dev"!

You're now on a new, empty workspace. Workspaces isolate their state, so if you run "terraform plan" Terraform will not see any existing state for this configuration.

$ terraform workspace list
default
* dev
production
staging
```

#### 7.1.1.1 Pros

- Easy to get started
- Convenient `terraform.workspace` expression
- Minimizes Code Duplication

#### 7.1.1.2 Cons

- Prone to human error (If you're manually interacting, it can be dangerous; automated: less danger!)
- State stored within same backend (problem: isolation of different environments state files)
- Codebase doesn't unambiguously show deployment configuration

### 7.1.2 File Structure

Directory layout provides separation, modules provide reuse

```shell
$ tree
.
├── _modules
│   ├── module-1
|   |   ├── main.tf
|   |   └── variables.tf
│   └── module-2
|       ├── main.tf
|       └── variables.tf
├── dev
│   ├── main.tf
│   └── terraform.tfvars
├── production
│   ├── main.tf
│   └── terraform.tfvars
└── staging
    ├── main.tf
    └── terraform.tfvars

6 directories, 10 files

```

#### 7.1.2.1 Pros

- Isolation of backends
  - Improved Security
  - Decreased potential for human error
- Codebase fully represents deployed state

#### 7.1.2.2 Cons

- Multiple `terraform apply` required to provision environments
- More code duplication, but can be minimized with modules

## 7.2 File Structure (environments + components)

```shell
.
├── _modules
│   ├── compute-module
|   |   ├── main.tf
|   |   └── variables.tf
│   └── networking-module
|       ├── main.tf
|       └── variables.tf
├── dev
│   ├── compute
|   |   ├── main.tf
|   |   └── terraform.tfvars
│   └── networking
|       ├── main.tf
|       └── terraform.tfvars
├── production
│   ├── compute
|   |   ├── main.tf
|   |   └── terraform.tfvars
│   └── networking
|       ├── main.tf
|       └── terraform.tfvars
└── staging
    ├── compute
    |   ├── main.tf
    |   └── terraform.tfvars
    └── networking
        ├── main.tf
        └── terraform.tfvars

```

Depending how complex infrastructure is, we probably want to start separating things out into not just having a single Terraform config for all of our infrastructure.

- Further separation (at logical component groups) useful for larger projects
  - Isolate things that change frequently from those which don't
- Referencing resources across configurations is possible using `terraform_remote_state`

## 7.3 Terragrunt

- Tool by gruntwork.io that provides utilities to make certain Terraform use cases easier
  - Keeping Terraform code DRY
  - Executing commands across multiple TF configs
  - Working with multiple cloud accounts

## 7.4 Examples

### 7.4.1 Workspaces

> main.tf

```terraform
terraform {
  # Assumes s3 bucket and dynamo DB table already set up
  backend "s3" {
    bucket         = "devops-directive-tf-state"
    key            = "07-managing-multiple-environments/workspaces/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locking"
    encrypt        = true
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

variable "db_pass" {
  description = "password for database"
  type        = string
  sensitive   = true
}

locals {
  environment_name = terraform.workspace
}

module "web_app" {
  source = "../../06-organization-and-modules/web-app-module"

  # Input Variables
  bucket_prefix    = "web-app-data-${local.environment_name}"
  domain           = "devopsdeployed.com"
  environment_name = local.environment_name
  instance_type    = "t2.micro"
  create_dns_zone  = terraform.workspace == "production" ? true : false
    # Because the DNS zone is going to be global across the two environments
    # If i'm in production go ahead and provision that zone
    # If i'm not in production don't use it
    # In staging and development i'll use the DNS zone that already exists
  db_name          = "${local.environment_name}mydb"
  db_user          = "foo"
  db_pass          = var.db_pass
}
```

```bash
terraform init
terraform workspace list
terraform workspace new production
terraform workspace list
terraform apply
terraform workspace new staging
terraform apply
terraform destroy
terraform workspace select production
terraform destroy
```

### 7.4.2 File Structure

> tree of project:
>
> .
> ├── README.md
> ├── global
> │   └── main.tf
> ├── production
> │   └── main.tf
> └── staging
>     └── main.tf
> global/main.tf

```terraform
terraform {
  # Assumes s3 bucket and dynamo DB table already set up
  backend "s3" {
    bucket         = "devops-directive-tf-state"
    key            = "07-managing-multiple-environments/global/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locking"
    encrypt        = true
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# Route53 zone is shared across staging and production
resource "aws_route53_zone" "primary" {
  name = "devopsdeployed.com"
}
```

> production/main.tf

```terraform
terraform {
  # Assumes s3 bucket and dynamo DB table already set up
  # See /code/03-basics/aws-backend
  backend "s3" {
    bucket         = "devops-directive-tf-state"
    key            = "07-managing-multiple-environments/production/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locking"
    encrypt        = true
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

variable "db_pass" {
  description = "password for database"
  type        = string
  sensitive   = true
}

locals {
  environment_name = "production"
}

module "web_app" {
  source = "../../../06-organization-and-modules/web-app-module"

  # Input Variables
  bucket_prefix    = "web-app-data-${local.environment_name}"
  domain           = "devopsdeployed.com"
  environment_name = local.environment_name
  instance_type    = "t2.micro"
  create_dns_zone  = false
  db_name          = "${local.environment_name}mydb"
  db_user          = "foo"
  db_pass          = var.db_pass
}
```

> staging/main.tf

```terraform
terraform {
  # Assumes s3 bucket and dynamo DB table already set up
  # See /code/03-basics/aws-backend
  backend "s3" {
    bucket         = "devops-directive-tf-state"
    key            = "07-managing-multiple-environments/staging/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locking"
    encrypt        = true
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

variable "db_pass" {
  description = "password for database"
  type        = string
  sensitive   = true
}

locals {
  environment_name = "staging"
}

module "web_app" {
  source = "../../../06-organization-and-modules/web-app-module"

  # Input Variables
  bucket_prefix    = "web-app-data-${local.environment_name}"
  domain           = "devopsdeployed.com"
  environment_name = local.environment_name
  instance_type    = "t2.micro"
  create_dns_zone  = false
  db_name          = "${local.environment_name}mydb"
  db_user          = "foo"
  db_pass          = var.db_pass
}
```

commands to run:

```bash
terraform init
terraform apply # provisioning shared resources
cd production
terraform init
terraform apply
cd ../staging
terraform init
terraform apply
```

**Why would this approach be potentially beneficial over the workspaces approach?**

1. I can very clearly look at my directory file structure. I could store my `.tfvars` files within there and very easily see what environments I have deployed and how my configuration maps onto those. With the workspaces approach all we have is that `main.tf` until we actually have Terraform installed and start issuing `terraform workspace list`, we won't know what is deployed, so it's much easier to look at the codebase and reason about the actual infrastructure we have deployed using filesystem-based approach.
2. One downside is that we do have a little bit more code repetition right instead of just that single `main.tf` we have a `main.tf` in each of our environments.
