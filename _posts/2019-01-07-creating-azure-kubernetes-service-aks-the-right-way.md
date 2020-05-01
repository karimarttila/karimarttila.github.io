---
layout: post
title: "Creating Azure Kubernetes Service (AKS) the Right Way"
category: [azure]
tags: [azure, kubernetes, arm, terraform, iac, cloud]
date: 2019-01-07
---

![](/img/2019-01-07-creating-azure-kubernetes-service-aks-the-right-way_img_1.png)

*IntelliJ IDEA with Terraform Plugin.*

### Introduction

About 7 months ago I was asked to create a short Azure [AKS](https://docs.microsoft.com/en-us/azure/aks/) poc for one project using [ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) with a preview version of AKS (of that time), you can read more about that in my previous blog post [Running Azure Kubernetes Service (AKS)]({% post_url 2018-09-24-running-azure-kubernetes-service-aks %}). ARM was a requirement so I couldn't use my choice of Infrastructure as Code tool [Terraform](https://www.terraform.io/).

In my free-time study projects I decided to deploy my Simple Server both in AWS side using [EKS](https://aws.amazon.com/eks) and Azure side using AKS and compare the experiences. I already converted the Simple Server to use [AWS DynamoDB]({% post_url 2018-12-12-aws-dynamodb-with-clojure %}) and [Azure Table Storage]({% post_url 2019-01-01-azure-table-storage-with-clojure %}) databases for these future projects. In this blog post I write about my experiences when creating Azure AKS infra using Terraform.

You can find the project in my [Github account](https://github.com/karimarttila/azure/tree/master/simple-server-aks).

### Azure Infrastructure Using ARM vs. Terraform

I don't have that much experience in ARM but quite extensive experience how to use Terraform in the AWS side. When using ARM for creating Azure resources I always felt like this would be easier if I only could use Terraform. Now I had a chance to see how Terraform handles Azure resources and I must say that the experiences were pretty good. Terraform is really nice to work with also in the Azure side. With a good editor that understands [Hashicorp Configuration Language](https://github.com/hashicorp/hcl) (hcl) and also understands the semantics of hcl entities and their relationships (like IntelliJ IDEA with [Terraform plugin](https://plugins.jetbrains.com/plugin/7808-hashicorp-terraform--hcl-language-support)) writing Azure resources was a breeze. Just like in the AWS side you can make a modular configuration of your Azure cloud infra. Terraform provides a plan phase in which you can check what configuration changes (new resources to be created, some resources to be updated etc.) are to be made before you apply the changes.

I would say that the most compelling reason to learn Terraform and hcl when creating cloud infra is this: You don't have to learn cloud specific tools and syntax (like CloudFormation in the AWS side and ARM in the Azure side) but you can re-use your Terraform knowledge and quite effortlessly create cloud infra in both AWS and Azure side pretty much the same way using the same tool and language.

Let's have a short example. First a short code snippet from the ARM / json version:

![](/img/2019-01-07-creating-azure-kubernetes-service-aks-the-right-way_img_2.png)

*ARM / json.*

... and then about the same thing using Terraform / hcl:

![](/img/2019-01-07-creating-azure-kubernetes-service-aks-the-right-way_img_3.png)

*Terraform / hcl.*

I would say that using Terraform with its modules, variables etc. is nicer to work with than ARM / json.

### The Solution

The overall solution is pretty simple: create Azure storage account to store Terraform state, create Azure AKS configuration in a modular manner using Terraform, and deploy the infra incrementally to Azure when you write new resource configurations.

#### 1. Create Azure Storage Account for Terraform Backend

You don't need this step for small pocs but I like to do things with best practices. So, the best practice is to store the Terraform state in a place where many developers can access the state (i.e. in the cloud, naturally). Terraform takes care of locking of the state so that many developers are not running cloud infra updates concurrently breaking the coherence of the infra. In the AWS side the natural place for the Terraform backend is S3, in the Azure side it is [Blob Storage](https://azure.microsoft.com/en-us/services/storage/blobs/). I created a simple script to automate this part: [create-azure-storage-account.sh](https://github.com/karimarttila/azure/blob/master/simple-server-aks/scripts/create-azure-storage-account.sh).

#### 2. Create Azure Infra Code Using Terraform

The next step is to create the Azure cloud infra code using Terraform (see directory [terraform](https://github.com/karimarttila/azure/tree/master/simple-server-aks/terraform)). I usually create a "main" configuration part which defines what modules comprise the main configuration: file [env-def.tf](https://github.com/karimarttila/azure/blob/master/simple-server-aks/terraform/modules/env-def/env-def.tf). If you look at that file you see that we create here the resource group for the infra, [Azure Container Registry (ACR)](https://azure.microsoft.com/en-us/services/container-registry/), a couple of public ips and the Azure Kubernetes Service (AKS). The actual resources are provided as Terraform modules in [modules](https://github.com/karimarttila/azure/tree/master/simple-server-aks/terraform/modules) directory. When you have the main definition (env-def.tf) and the modules ready you can create the environments (e.g. development, integration-testing, performance-testing, production) separately just by injecting the environment specific values to the main configuration module, see example [dev.tf](https://github.com/karimarttila/azure/blob/master/simple-server-aks/terraform/envs/dev.tf) for development environment. If I later wanted to create e.g. a production environment I could then easily refactor certain parameters out of the env-def.tf to the environment files (e.g. vm_sizes etc.).

#### 3. Deploy Cloud Infra!

You don't have to create the whole cloud infra in one shot before applying it to the cloud provider. And actually you shouldn't. And actually I never do so. I create the cloud infra incrementally, resource by resource. I try "terraform plan", and if everything looks good I apply the new resources using command "terraform apply". This way you can create the cloud infra incrementally which is pretty nice. And once you have one environment ready (e.g. development environment) it is pretty easy to apply the same code for other environments as well (which are going to be as exact copies of your development environment as you wanted to be — probably you just want to use more inexpensive resource types (e.g. vm sizes) in your development environment for saving some project money).

### Experiences

#### Service Principal Hassle

I spent quite a lot of time trying to create a [Service principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals) for using with Terraform commands so that this Service principal would have authorization to create other Service principals (AKS needs a service principal to be able to create virtual machines for the Kubernetes cluster infra). This turned out not to be an easy task. I could have created the right kind of Service principal outside Terraform and inject the Service principal id and secret to the Terraform configuration but this would have been a bit of an ugly solution — best practice is not to divide your cloud infra to various scripts but try to make the whole cloud infra using one configuration which you can apply with one command.

Finally I gave up and just used my own credentials with terraform (I have owner role in my Azure subscription — owner role has right to create other Service principals). If you are interested about the hassle and what I tried read the chapter "Service Principle Hassle" in the [README.md](https://github.com/karimarttila/azure/tree/master/simple-server-aks) file.

#### Terraform Hassle

Not everything is dancing on a bed of roses with Terraform. Terraform state can corrupt which is a real hassle if you have a big environment (not to speak if that environment is production). Sometimes Terraform fails and gives cryptic error messages. Usually a standard procedure is to wait a couple of minutes and try to give "terraform apply" command again — if the second apply run is successful probably some previous cloud resource was not finalized before Terraform tried to create some other resource which had dependency to the previous one. This happened to me when creating AKS and Service principal. So, if you try the example in my Github account don't get frightened if the first "terraform apply" command fails.

#### Update 2019–01–09: Major Hassle with Authenticating AKS to Pull Images from ACR

I must admit now that I created the original version of this blog post a bit too early :-) . After the original version of this blog post I tried to use the terraform version of that time to deploy the [Simple Server single-node Kubernetes deployment](https://github.com/karimarttila/kubernetes/tree/master/simple-server) to that AKS. Didn't work. AKS didn't have authorization to pull images from ACR. When examining the failed pod (using kubectl describe command…) I saw that the image pull was failed: "Failed to pull image … unauthorized: authentication required…". It took me quite some time to figure out how to do this. In this kind of situation it is a good idea to create a working reference infra e.g. using Portal or command line tool. So, I created everything (AKS, ACR, Public IPs, Service principal, Role assignment etc.) manually step by step using az cli and deployed the Simple Server single-node version there and tested the deployment by curling the Simple Server API — the reference infra worked ok. Now I had a working reference infra for examining what was wrong with my Terraform configuration. I examined the created azure resources side by side (reference infra created by az cli and my terraform code). Finally I figured out the issues and was able to fix them. While spending some 8h with the terraform code I also quite extensively refactored it (e.g. put Service principal to aks module where it belongs, put role assignment to acr module where it belongs, introduced locals etc.).

The lesson of the story: If terraform apply goes smoothly it doesn't yet mean that the cloud infra is working properly — deploy what ever you are doing (Kubernetes, Docker, apps to VMs…) to the infra — if this part also works ok then you have a working cloud infra.

### Conclusions

Using Terraform it was pretty easy to create Azure AKS infra and related Azure resources. If I can choose I will use Terraform rather than ARM in my future Azure projects.

Stay tuned since in my next blog post I'm going to write about my experiences to deploy my Simple Server Azure Table Storage version to that Azure AKS Kubernetes cluster I wrote about in this current blog post.


*The writer has two AWS certifications and one Azure certification and is working at the [Tieto Corporation](https://www.tieto.com/) in Application Services / Application Development / Public Cloud team designing and implementing cloud native projects. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
