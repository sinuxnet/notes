[TOC]

# 2: Terraform Overview + Setup

## 2.1 What is Terraform

- Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently
- Enables application software best practices to infrastructure
- Compatible with many clouds and services (Cloud-agnostic)

## 2.2 Common Patterns

<img src="./images/terraform_ansible.png" alt="terraform_ansible" style="zoom: 25%;" />

We have our setup and Terraform is going to provision a number of virtual machines for us, then we could take a tool like Ansible and install all the necessary dependencies inside of those virtual machines.

<img src="./images/terraform_packer.png" alt="terraform_packer" style="zoom:25%;" />

In this case Terraform provisions the servers and Packer is used to build the image from which those virtual machines are created, rather than provisioning and then installing and configuring like we were with Ansible, now we can pre-package all of that into a machine image that we can provision copies of with Terraform.

We can actually even build our application code into that server template; so that once it's provisioned, not only does it have all the dependencies, it also has our application bundled right in.

<img src="./images/terraform_kubernetes.png" alt="terraform_kubernetes" style="zoom:25%;" />

In this case we are using Terraform to provision our Kubernetes clusters, maybe it's a managed cluster like EKS in AWS or maybe a self-managed cluster where we're provisioning a bunch of virtual machines and installing Kubernetes onto them, but we're using Terraform to define the cloud resources and then we're using Kubernetes to define how our application is deployed and managed on those cloud resources.

## 2.3 Terraform Architecture

### 2.3.1 **Terraform Core**

At the very center of Terraform we have what's called `Terraform Core`. This is the engine that takes our configuration files (Terraform State & Terraform Config), so that Terraform config there on the bottom in conjunction with our current state of the world.

Terraform Core takes those two inputs then it needs to figure out how to interact with the cloud provider APIs to make that state match the config that we want it to.

<img src="./images/terraform_core.png" alt="terraform_core" style="zoom: 33%;" />

:spiral_notepad: Terraform State file is managed by Terraform itself and it contains references to all the infrastructure that we've already provisioned

### 2.3.2 Providers

Providers are kind of like plugins to the core that tell Terraform how to map a specific configuration (example for AWS) onto the current state of AWS's API, or if we're provisioning something in Cloudflare we need to take that configuration and map it onto the specific set of API calls to achieve the desired state.

There are many providers that are available.
