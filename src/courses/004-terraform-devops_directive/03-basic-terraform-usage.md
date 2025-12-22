[TOC]

# 3. Basic Terraform Usage

## 3.1 Basic Usage Sequence

1. **`terraform init`**: Initializes project
2. **`terraform plan`**
   1. Takes configuration
   2. Checks it against the currently deployed state of the world and state file
   3. Figures out the set of the sequence of things that need to happen to provision that infrastructure
3. **`terraform apply`**: Takes that set of commands and applies them
4. **`terraform destroy`**: Cleans up resources after doing example or takes down infrastructure that was being used previously but no longer needed

## 3.2 Providers

https://registry.terraform.io

- Official tags give us a higher level of confidence and trust in a provider being up-to-date and high quality.

## 3.3 First Example

```terraform
terraform {
    required_providers {
        aws = {
            source = "hashicorp/aws"
            version = "~> 3.0"
            }
        }
    }

provider "aws" {
    region = "us-east-1"
    }
```

## 3.4 `terraform init`

```shell
$ tree -a
.
└── main.tf

1 directory, 1 file

$ terraform init
```

When we run the `init` command, it actually goes off and downloads the associated providers that we defined in that Terraform block. So it's going to get code for the AWS provider from the Terraform registry. It actually downloads that and puts it into our working directory, so if we run the `tree` again after running `terraform init` we have:

```shell
$ tree -a .
.
├── .terraform
│   └── providers
│       └── registry.terraform.io
│           └── hashicorp
│               └── aws
│                   └── 3.76.1
│                       └── linux_amd64
│                           └── terraform-provider-aws_v3.76.1_x5
├── .terraform.lock.hcl
└── main.tf

8 directories, 3 files
```

Now we have a `.terraform` hidden directory with a providers subdirectory and registry.terraform.io subdirectory and so on. registry.terraform.io is the official registry but you can have additional custom Terraform registries or third-party registries where those providers are actually stored.

And when we go all the way down we see that final directory we're seeing the version, the architecture and then the actual code for provider lives in that final subdirectory.

We also have that `lockfile` which contains information about the specific dependencies and providers that are installed within this workspace.

And if you use `modules`, it will download and pull those into the working directory. Modules are located in `.terraform/modules/[MODULE_NAME]`

## 3.5 State File

- State file is Terraform's representation of the world
- JSON file containing information about every single resource and data object that we have deployed using Terraform
- Contains Sensitive Info (e.g database password), so you need to protect state file (permissions and encryption)
- Can be stored locally or remotely (By default: locally)

```json
{
    "version": 4,
    "terraform_version": "0.14.4",
    "serial": 5
    "lineage": "adqeb0b0-c0a3-e454-falke993en984",
    "outputs": {},
    "resources": [
        {
            "mode": "managed",
            "type": "aws_instance",
            "name": "example",
            "provider": "providerp[\"registry.terraform.io/hashicorp/aws\"]",
            "instances": [
                {
                    "schema_version": 1,
                    "attributes": {
                        "ami": "ami-9118992242bb809215",
                        "arn": "arn:aws:ec2:us-east-1:917774923557:instance/i-9d8df8348g3238",
                         "public_ip": "3.99.252.30",
                        ...
                        <MANY MORE ATTRIBUTES>
                        ...
                    },
                    "sensitive_attributes": [],
                    "private": <SENSITIVE INFO>
                }
            ]
        }
    ]
}
```

Data object: there are blocks which correspond to resources, there are also resources which correspond to data, maybe we're pulling some information from a third-party API or we could have some fixed data within our code, and those could be used to reference things that were not provisioned and are not managed by Terraform but we want to pull those in to influence how the infrastructure is actually provisioned.

### 3.5.1 Local Backend

:heavy_plus_sign: Simple to get started!

:heavy_minus_sign: Sensitive values in plain text

:heavy_minus_sign: Not collaborative

:heavy_minus_sign: Manual

### 3.5.2 Remote Backend

State file is stored in a remote server.

One option is **Terraform Cloud**, we can also self-manage a remote backend to store those state files using something like Amazon Simple Storage Service (S3), Google Cloud Storage, etc.

:heavy_plus_sign: Sensitive data encrypted

:heavy_plus_sign: Collaboration possible

:heavy_plus_sign: Automation possible

:heavy_minus_sign: Increased complexity

## 3.6 `terraform plan`

This command takes our Terraform config which we're defining on our system (that is desired state: what we want our infrastructure to look like) and it compares it with the Terraform state which is the actual state of the world.

> Also this is slight misnomer because you could have gone in and modified something within the GUI or sort of out of band of our Terraform workflow, as long as you haven't done that the Terraform state should represent the actual state of the world. If you have done that you can actually get yourself into trouble so you should always try to avoid making modifications to your infrastructure outside of Terraform workflow.

## 3.7 `terraform apply`

When we have a mismatch output from comparing our desired state and state file, the apply command comes into play and it can figure out the sequence of API calls that are necessary to actually provision what we want which we wrote as our desired state.

## 3.8 `terraform destroy`

To clean up all the resources in Terraform config file.

## 3.9 Remote Backend (Terraform Cloud)

```terraform
terraform {
    backend "remote" {
        organization = "devops-directive"

        workspaces {
            name = "terraform-course"
            }
        }
    }
```

Visit: https://app.terraform.io

## 3.10 Remote Backend (AWS)

```terraform
terraform {
    backend "s3" {
        bucket          = "devops-directive-tf-state"
        key             = "tf-infra/terraform.tfstate"
        region          = "us-east-1"
        dynamodb_table  = "terraform-state-locking"
        encrypt         = true
        }
    }
```

- S3 Bucket used for storage
- DynamoDB used for locking

Why use DynamoDB? Because we could have multiple people working on the same project at once, you want to prevent two people from trying to apply different changes at the same time, so we use the atomic guarantees that DynamoDB offers. If I issue a command, I can lock the Terraform configuration and if someone else, one of my colleagues issues another `terraform apply` command, they will get rejected and that `apply` command would be rejected until mine has finished.

:question: How to provision these S3 buckets and DynamoDB tables? (chicken and egg problem)

### 3.10.1 Bootstrapping

Bootstrapping process will enable us to provision those resources and then import them into our configuration, so even the remote backend resources can be managed by Terraform as well.

:one: At first, we specify our Terraform configuration with no remote backend, :two: so it will default to a local backend, and in second step, we then define the resources that we need S3 bucket and DynamoDB table, :three: then we would go through our normal apply process and then within our Terraform state file we have those two resources within our AWS account, we also have those resources provisioned :four: we can then change our backend so before we didn't specify backend, now we specifying we want to use remote with proper configuration :five: then rerun `init` command, it recognizes before you were using a local backend, now you have a remote backend, then it will ask: _do you want to import that state into new backend_; then with choosing _yes_ we have our state in that remote backend.
