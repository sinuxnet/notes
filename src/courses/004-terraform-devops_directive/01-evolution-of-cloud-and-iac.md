[TOC]

# 1: Evolution of Cloud + Infrastructure as Code

## 1.1 Historical Overview

### 1.1.1 Pre-Cloud (1990s --> 2000s)

- Buy bunch of servers (physical servers)
- Setting up datacenter (handle all power management and networking and ...)

### 1.1.2 Cloud (2010s --> Now)

- Rather than provision your server, deploy your app to the cloud
- The cloud has become pretty much the de facto standard
- On-demand resource

## 1.2 What has changed?

- Infrastructure provisioned via APIs
- Servers created & destroyed in seconds
- Long-lived + mutable --> Short-lived + Immutable

Each individual unit is the short-lived immutable thing that we're never going to change. (It's a paradigm shift of thinking about infrastructure of apps)

## 1.3 Provisioning Cloud Resources

Three Approaches:

- GUI: By clicking
- API/CLI: `aws ec2 new`, it's more programmatic
- IaC: Define infrastructure in codebase

## 1.4 What is Infrastructure as Code (IaC)?

Categories of IaC tools:

1. **Ad hoc scripts**: like a bash script that calls AWS to provision five ec2 instances
2. **Configuration Management tools**: like Ansible, Puppet or Chef, they are really positioned to manage the software that is running and configuration of infrastructure, and these are more suited for on-prem setups where you're provisioning some hardware and then you need to manage what software is installed and how those are configured
3. **Server Templating tools:** This category is building out a template for what you're going to provision onto a server. `AMI`, Amazon Machine Image or any virtual machine image is provisioned from some template and you can build in all your dependencies into that template. You can build that template and you can spawn multiple copies of the same server over and over.
4. **Orchestration tools:** The most popular these days in terms of orchestration tools is Kubernetes which is for orchestrating containers. These are for defining how your application is deployed.
5. **Provisioning tools:** Provisioning those cloud resources to begin with, and important thing to call out here is the concept of declarative versus imperative.

### 1.4.1 Declarative vs. Imperative

- Declarative tools: You define the end state of what you want, example: 5 servers 1 load balancer 1 s3 bucket etc, and the tool manages what API calls need to be made and how to actually make that happen.
- Imperative tools: You tell the system what you want to happen and the sequence in which you want them to happen.

A lot of configuration management tools fall more on the imperative side, they do offer some utilities to make them more declarative and make those scripts idempotent, so you can run them multiple times. But a lot of provisioning tools which Terraform falls into are primarily on the declarative side, so you specify the end state you want your infrastructure to take, and then you let the tools handle the details of how to actually get there.

## 1.5 IaC Provisioning Tools Landscape

### 1.5.1 Cloud Specific

- AWS Cloud Formation
- Azure Resource Management
- Google Cloud Deployment Manager

Provided by major cloud providers and focusing on provisioning infrastructure within that cloud. You are out of luck if you want to provision something outside of that cloud or another cloud if you use one of them like CloudFormation.

### 1.5.2 Cloud Agnostic

They can be used across any cloud provider. They can interact with almost anything with an API online. Use cases are like if you have your application deployed across multiple clouds or you want to be able to use auxiliary services like Cloudflare or Atlas for MongoDB.

- Terraform
- Pulumi
