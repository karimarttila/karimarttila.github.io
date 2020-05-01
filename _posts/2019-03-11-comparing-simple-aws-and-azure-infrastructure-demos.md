---
layout: post
title: "Comparing Simple AWS and Azure Infrastructure Demos"
category: [iac]
tags: [iac, aws, azure, cloud, terraform]
date:	2019-03-11
---

![](/img/2019-03-11-comparing-simple-aws-and-azure-infrastructure-demos_img_1.png)

*AWS and Azure Intro Demos in a Family Portrait.*

### Introduction

In my new unit I was asked to create some training material how to do infrastructure coding for both AWS and Azure. I considered a while what kind of infrastructure to use as an example and finally I decided that the infrastructure should be as simple as possible (not to intimidate new cloud learners) but comprising some basic elements that you can find in every infrastructure: a virtual network, a subnet, a firewall and some computing resource. So, I created two intro demonstrations, one for both AWS and Azure sides using this specification. The demonstrations can be found in Tieto / Public Cloud team's [Github account](https://github.com/tieto-pc):

* AWS: <https://github.com/tieto-pc/aws-intro-demo>
* Azure: <https://github.com/tieto-pc/azure-intro-demo>
I also created a repository which gives basic instructions how to start building cloud competence, you might also be interested about that if you are a new cloud learner: <https://github.com/tieto-pc/learning-cloud-skills>.

In this blog post I comment how incredibly similar the demonstrations are even though they are deployed into two different public clouds.

### Overall Infrastructure as Code Solutions

I used [Terraform](https://www.terraform.io/) in both AWS and Azure demonstrations as an IaC tool (see also my previous blog post [How to Create Infrastructure as Code for AWS and Azure]({% post_url 2019-02-28-how-to-create-infrastructure-as-code-for-aws-and-azure %}). Even though the demonstrations target different public clouds you can and should modularize the Terraform code according the same best practices:

**Environment parameters**. In the envs folders we host the various environments. In this demonstration we have only the dev environment, but this folder could have similar environment parameterizations for qa, perf, prod environments etc. If you look at both solutions you find that the dev.tf files in both sides are pretty similar: first there is the Terraform backend definition, then locals definitions and finally we call the env-def module injecting the parameter values of this environment.

**Environment definition**. In the env-def folders we define the modules that will be used in every environment. The environment injects the environment specific parameters to the env-def module which then creates the actual infra using those parameters by calling various infra modules and forwarding environment parameters to the infra modules. In AWS side we call modules: "resource-groups", "vpc" and "ec2". In Azure side we call modules: "main-resource-group", "vnet" and "vm". You can see the one-to-one relationships of these modules.

**Modules**. In the modules folder we have the terraform modules that are used by the environment definition (env-def, also a terraform module itself). Both sides host the (conceptually) same three modules:

| AWS | Azure | Explanation |  
| --------------------------------|  
| vpc | vnet | Virtual network |  
| rg | rg | Resource group(s) |  
| ec2 | vm | Virtual machine |  
|---------------------------------|

### Virtual Network

The virtual networks in both sides are pretty similar. First in both AWS and Azure sides the modules define the virtual networks themselves (vpc and vnet). Then the modules define a subnet and a security group for the subnet. Defining these entities using Terraform is incredibly similar in both AWS and Azure sides. Since we are defining a public subnet the AWS side requires some additional infrastructure boilerplate: an internet gateway and a route table.

### Resource Group

This is the main conceptual difference between the solutions. Azure has a concept of a concrete resource group into which you create your cloud resources. AWS has just recently added a resource group concept which is not a concrete resource group but a way to list certain resources based on certain tag keys and values you define for the resource group.

Therefore in the Azure side we define just one main resource group into which we create all cloud resources of the demonstration. In the AWS side we create 5 distinct resource groups for listing resources based on their most important tag metadata.

### Virtual Machine

And finally the virtual machine modules: ec2 (AWS) and vm (Azure).

The modules start with ssh key definitions — to make things easier for new cloud learners I have automated the creation of ssh key pairs in both demonstrations. There are separate Terraform functions for storing the private key part whether you are using Windows or *nix as your development machine (once again to make things a bit easier for new cloud learners; BTW thanks for Sami Huhtiniemi who kindly provided the solution how to store the private key in Windows side calling Powershell as terraform local-executor).

Both modules define a public ip that will be assigned for the virtual machine (Azure solution has some extra infra boilerplate — requires a virtual network interface (nic)).

The actual virtual machine definitions are also pretty similar. In the AWS side you tell the subnet and security group of the vm when defining it, in the Azure side you define the vm and tell its nic, which has the association to a subnet — a bit differently done but conceptually the same.

### The Longer Story

Both demonstrations provide a README.md file which gives a longer explanation regarding that demonstration — I encourage the reader to read those files to get a more detailed view regarding the demonstrations (e.g. more detailed explanations regarding Terraform backends in AWS and Azure sides etc.). At the end of the README files you can also find a detailed demonstration manuscript — you should be able to deploy the infrastructure to AWS and Azure clouds following the demonstration manuscripts (I tested the deployments of both demonstrations using both my Linux workstation and my virtual Windows 10 workstation).

### Conclusions

Using Terraform as an Infrastructure as Code tool you get synergy benefit you don't get by using cloud providers' native IaC tools: you can create infrastructure as code solutions in more than one public cloud using the same tool and therefore re-using your IaC knowledge— that's why Terraform is my choice of tool when creating infrastructure as code in both AWS and Azure.

*The writer has two AWS certifications and one Azure certification and is working at [Tieto Corporation](https://www.tieto.com/) in Application Services / Public Cloud team designing and implementing cloud native solutions. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
