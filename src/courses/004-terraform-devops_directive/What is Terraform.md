

[TOC]



# What is Terraform?

Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently.

IaC tools allow you to define your entire cloud infrastructe as a set of config files which then the tool terraform can go off and interact with the cloud provider api and provision and manage on our behalf.

# Course Overview

1. Evolution of Cloud + Infrastructure as Code
  
2. Terraform overview + Setup
  
3. Basic Terraform Usage
  
4. Variables and Outputs
  
5. Language Features
  
6. Project Organization + Modules
  
7. Managing Multiple Environments
  
8. Testing Terraform Code
  
9. Developer Workflows


# 1: Evolution of Cloud + Infrastructure as Code

## 1.1 Historical Overview

### 1.1.1 Pre-Cloud (1990s --> 2000s)

- Buy bunch of servers (physical servers)
  
- Setting up datacenter (handle all power management and networking and ...)
  

### 1.1.2 Cloud (2010s --> Now)

- Rather than provision your server, deploy your app to the cloud
  
- the cloud it's become pretty much the de facto standard
  
- On-demand resource
  

## 1.2 What has changed?

- Infrastructure provisioned via APIs
  
- Servers created & destroyed in seconds
  
- Long-lived + mutable --> Short-lived + Immutable

each individual unit is the short-lived immutable thing that we're never gonna change. (It's a paradigm shift of thinking about infrastructre of apps)

## 1.3 Provisioning Cloud Resources

Three Approaches:

* GUI: By clicking
* API/CLI: `aws ec2 new`, its more programmatically
* IaC: Define infrastructure in codebase

## 1.4 What is Infrastructure as Code (IaC)?

Categories of IaC tools:

1. **Ad hoc scripts**: like a bash script that calls AWS to provision five ec2 instances
2. **Configuration Management tools**: like Ansible, Puppet or chef, they are really positioned to manage the software that is running and configuration of infrastructure, and these are more suited for on-perm setups where you're provisioning some hardware and then you need to manage what software is installed and how those are configured
3. **Server Templating tools:** This category is building out a template for what you're going to provision onto a server. `AMI`, Amazon Machine Image or any virtual machine image is provisioned from some template and you can build in all your dependencies into that template. You can build that template and you can spawn multiple copies of the same server over and over.
4. **Orchestration tools:** The most popular these days in terms of orchestration tools is Kubernetes which is for orchestrating containers. These are for how define your application deployment.
5. **Provisioning tools:** Provisioning those cloud resources to begin with, and important thing to call out here is the concept of declarative versus imperative. 

### 1.4.1 Declarative vs. Imperative

* Declarative tools: You define the end state of what you want, example: 5 servers 1 load balancer 1 s3 bucket etc, and the tool manages what API calls need to be made and how to actually make that happen.
* Imperative tools: You tell the system what you want to happen and the sequence in which you want them to happen.

A lot of configuration management tools for fall more on the imperative side, they do offer some utilities to make them more declarative and make those scripts idempotent, so you can run them multiple times. But a lot of provisioning tools which terraform falls into they are primarily on the declarative side so you specify the end state you want your infrastructure to take, and then you let the tools handle the details of how to actually get there

## 1.5 IaC Provisioning Tools Landscape

### 1.5.1 Cloud Specific

* AWS Cloud Formation
* Azure Resource Management
* Google Cloud Deployment Manager

Provided by major cloud provider and focusing on provisioning infrastructure within that cloud. You are out of luck if you want provision something out of that cloud or another cloud if you use one of them like Cloud Formation.

### 1.5.2 Cloud Agnostic

They can use across any cloud provider. They can interact with almost anything with and API online. Use cases are like if you have your application deployed across multiple clouds or you want to be able to use auxiliary services like Cloudflare or Atlas for MongoDB

* Terraform
* Pulumi

# 2: Terraform Overview + Setup

## 2.1 What is Terraform

* Terraform is a tool building, changing, and versioning infrastructure safely and efficiently
* Enables application software best practices to infrastructure
* Compatible with many clouds and services (Cloud-agnostic)

## 2.2 Common Patterns

<img src="./images/terraform_ansible.png" alt="terraform_ansible" style="zoom: 25%;" />

We have our setup and terraform is going to provision a number of virtual machines for us, then we could take a tool like ansible and install all the necessary dependencies inside of those virtual machines.

<img src="./images/terraform_packer.png" alt="terraform_packer" style="zoom:25%;" />

In this case Terraform provisions the servers and Packer is used to build the image from those virtual machines are created, rather than provisioning and then installing and configuring like we were with ansible, now we can pre-package all of that into a machine image that we can provision copies of with Terraform. 

We can actually even build our application code into that server template; so that once it's provisioned, not only does it have all the dependencies, it also has our application bundled right in.

<img src="./images/terraform_kubernetes.png" alt="terraform_kubernetes" style="zoom:25%;" />

In this case we using Terraform to provision our Kubernetes clusters, maybe it's a managed cluster like EKS in AWS or maybe a self-managed cluster where we're provisioning a bunch of virtual machines and installing Kubernetes onto it, but we're using Terraform to define the cloud resources and then we're using Kubernetes to define how our application is deployed and managed on those cloud resources.

## 2.3 Terraform Architecture

### 2.3.1 **Terraform Core**

At the very center of Terraform we have what's called `Terraform Core`. This is kind of engine that takes our configuration files (Terraform State & Terraform Config), so that Terraform config there on the bottom in conjunction with our current state of the world. 

Terraform Core takes those two inputs then it needs to figure out how to interact with the cloud provider APIs to make that state match the config that we want it to.

<img src="./images/terrform_core.png" alt="terrform_core" style="zoom: 33%;" />

:spiral_notepad: Terraform State file is managed by Terraform itself and it contains references to all the infrastructure that we've already provisioned 

### 2.3.2 Providers

Providers are kind of like plugins to the core that tell Terraform how to map a specific configuration (example for AWS) onto the current state of AWS's API, or if we're provisioning something in Cloudflare we need to take that configuration and map it onto the specific set of API calls to achieve the desired state.

There are many providers that are available

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

* official tags give us a higher level of confidence and trust in a provider being up-to-date and high quality 

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

When w run the `init` command, actually goes off and downloads the associated providers that we defined in that terraform block. So it's going to get code for the AWS provider from the Terraform registry. It actually downloads that and puts it into our working directory, so  if we run the `tree` again after running `terraform init` we have:

```shell
$ tree -a .
.
├── .terraform
│   └── providers
│       └── registry.terraform.io
│           └── hashicorp
│               └── aws
│                   └── 3.76.1
│                       └── linux_amd64
│                           └── terraform-provider-aws_v3.76.1_x5
├── .terraform.lock.hcl
└── main.tf

8 directories, 3 files
```

Now we have a `.terraform` hidden directory with a providers subdirectory and registry.terraform.io  subdirectory and ... . registry.terraform.io is official registry but you can have additional custom Terraform registry or third party registries where those providers actually stored. 

And when we go all the way down we see that final directory we're seeing the version, the architecture and then the actual code for provider lives in that final subdirectory.

We also have that `lockfile` which contains information about the specific dependencies and providers that are installed within this workspace.

And if you use `modules`, it will download and pulls those into working directory. Modules are located in `.terraform/modules/[MODULE_NAME]` 

## 3.5 State File

* State file is Terraform's representation of the world
* JSON file containing information about every single resource and data object that we have deployed using Terraform
* Contains Sensitive Info (e.g database password), so you need to protect state file (permissions and encryption)
* Can be stored locally or remotely (By default: locally)

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
                        <MANY MORE ARREIBUTES>
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

data object: there are blocks which correspond to resources, there is also resources which correspond to data, maybe we're pulling some information form a third-party API or we could have some fixed data within our code, and those could be used to referencing things that were not provisioned and are not managed by Terraform but we want to pull those in to influence how the infrastructure is actually provisioned.

### 3.5.1 Local Backend

:heavy_plus_sign: Simple to get started!

:heavy_minus_sign: Sensitive values in plain text

:heavy_minus_sign: Uncollaborative

:heavy_minus_sign: Manual

### 3.5.2 Remote Backend

State file store in a remote server.

One option is **Terraform Cloud** , we can also self-managed a remote backend to store those state files using something like Amazon Simple Storage Service (S3), Google Cloud Storage or etc.

:heavy_plus_sign: Sensitive data encrypted

:heavy_plus_sign: Collaboration possible

:heavy_plus_sign: Automation possible

:heavy_minus_sign: Increased complexity

## 3.6 `terraform plan`

This command is taking our Terraform config which we're defining on our system (That is desired state: what we want our infrastructure look like) and it compares it with the Terraform state which is the actual state of the world. 

> Also this is slight misnomer because you could have gone in and modified something within the GUI or sort of out of band of our Terraform workflow, as long as you haven't done that the Terraform state should represent the actual state of the world. If you have done that you can actually get yourself into trouble so you should always try to avoid making modifications to your infrastructure outside of Terraform workflow.

## 3.7 `terraform plan`

When we have a mismatch output from comparing our desired state and state file, the apply command comes into play and it can figure out the sequence of API calls that are necessary to actually provision what we want which wrote it our desired state.

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

https://app.terraform.io

## 3.10 Remote Backend (AWS)

```terraform
terraform {
    backend "s3" {
        bucket          = "devops-directive-tf-state"
        key			   = "tf-infra/terraform.tfstate"
        region          = "us-east-1"
        dynamodb_table  = "terraform-state-locking"
        encrypt         = true
        }
    }
```

* S3 Bucket used for storage
* DynamoDB used for locking

Why use DynamoDB? Because we could have multiple people working on the same project at once, you want to prevent two people from trying to apply different changes at the same time, so we use that atomic guarantees that the DynamoDB offers. If I issue a command, I can lock the terraform configuration and if someone else, one of colleagues issues another `terraform apply` command, they will get reject and that `apply` command would be rejected until mine has finished. 

:question:How to provision these S3 bucket and DynamoDB tables? (chicken and egg problem)

### 3.10.1 Bootstrapping

Bootstrapping process will enable us to provision those resources and then import them into our configuration and then import them into our configuration, so even the remote backend resources can be managed by Terraform as well.

:one: At first, we specify our Terraform configuration with no remote backend, :two: so it will default to a local backend, and in second step, we then define the resources that we need S3 bucket and DynamoDB table, :three: then we would go through our normal apply process and then within our Terraform state file we have those two resources within our AWS account, we also have those resources provisioned  :four: we can then change our backend so before we didn't specify backend, now we specifying we want to use remote with proper configuration :five: then rerun `init` command, it recognizes before you were using a local backend, now you have a remote backend, then it will ask: *do you want to import that state into new backend*; then with choosing *yes* we have our state in that remote backend. 

# 04: Variables and Outputs

## 4.1 Variables Types

* Input Variables
  * var.\<name\>
  * You can think of input variables input parameters or arguments for a function.
* Local Variables
  * local.\<name\>
  * When we declare them we use `locals` plural.
  * These are like temporary variables within the scope of a function.
  * These just allow me to take a value which is repeated a few times throughout my configuration and pull it into  variable that can be reused, so i might say this service name is `x` or I am the owner of these resources.
* Output Variables
  * These are the return value of the function
  * These are what allow us to take multiple Terraform configurations and bundle them together 

