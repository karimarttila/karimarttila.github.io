---
layout:	post
title:	"Comparing Simple GCP Infrastructure Demo to AWS and Azure"
categories: [blog, aws]
tags: [aws]
date:	2019-07-05
---

  ![](/img/2019-07-05-comparing-simple-gcp-infrastructure-demo-to-aws-and-azure_img_1.png)AWS, GCP and Azure Intro Demos in a Family Portrait

### Introduction

Working as a Cloud Mentor in my corporation one of my duties is to teach junior cloud developers to create infrastructure as code (IaC). I have created various demonstrations both in AWS and Azure using both Terraform and cloud native tools (CloudFormation and ARM). At the same time I was studying [Google Cloud Platform](https://cloud.google.com/gcp/) myself. Last month I did two GCP certifications: Associate Cloud Engineer and Professional Cloud Architect. After these studies I decided to create the same simple IaC demonstration using the Google Cloud Platform that I had earlier created for AWS and Azure platforms. I have already written a blog article in which I compared AWS and Azure Terraform implementations ([Comparing Simple AWS and Azure Infrastructure Demos](https://medium.com/@kari.marttila/comparing-simple-aws-and-azure-infrastructure-demos-cf756d1ef68b)). In this new blog post I compare the new GCP demonstration to those previous demonstrations in the AWS and Azure side. All three demonstrations can be found in Tieto / Public Cloud team’s [Github account](https://github.com/tieto-pc):

* AWS: <https://github.com/tieto-pc/aws-intro-demo>
* Azure: <https://github.com/tieto-pc/azure-intro-demo>
* GCP: <https://github.com/tieto-pc/gcp-intro-demo>
I try not to iterate the same information in this new blog post that I already wrote in the previous blog post “[Comparing Simple AWS and Azure Infrastructure Demos](https://medium.com/@kari.marttila/comparing-simple-aws-and-azure-infrastructure-demos-cf756d1ef68b)” — exception being e.g. some tables since I add the GCP related information in this new blog post. I recommend the reader to read first that previous blog post to have some context for reading this new blog post.

### Overall Infrastructure as Code Solutions

The overall Terraform solution structure is already explained in the previous blog post ([Comparing Simple AWS and Azure Infrastructure Demos](https://medium.com/@kari.marttila/comparing-simple-aws-and-azure-infrastructure-demos-cf756d1ef68b) — the same title as in this chapter), so I don’t iterate it here. Instead, let’s just briefly summarize the main structure of all these three IaC solutions:

* Environment parameters are hosted in the envs folder.
* Environment definition is hosted in the modules/env-def folder.
* The modules folder hosts the actual resource modules (in this GCP solution: project, vpc and vm).
![](/img/2019-07-05-comparing-simple-gcp-infrastructure-demo-to-aws-and-azure_img_2.png)### Virtual Network / Virtual Private Cloud

The virtual networks (“virtual private cloud” in AWS and GCP) in all solutions are pretty similar. All solutions define the virtual networks themselves (vpc and vnet). Then the modules define a subnet and a security group for the subnet (AWS/Azure) or a firewall rule for the VPC (GCP). Defining these entities using Terraform is incredibly similar in all solutions. The AWS side requires some additional infrastructure boilerplate: an internet gateway and a route table. In the GCP solution you don’t have to assign an address space for the [VPC](https://cloud.google.com/vpc/) itself since VPC is a global resource in GCP — you assign an address space just for the subnet.

### Gathering Resources

This is the main conceptual difference between the solutions. Azure has a concept of a concrete resource group into which you create your cloud resources. AWS has just recently added a resource group concept which is not a concrete resource group but a way to list certain resources based on certain tag keys and values you define for the resource group. GCP has a similar concept as Azure’s resource group: [Project](https://cloud.google.com/resource-manager/docs/creating-managing-projects).

### Virtual Machine

And finally the virtual machine modules: vm (GCP), ec2 (AWS) and vm (Azure).

The modules start with ssh key definitions (read the longer version in the previous blog post). All modules define a public ip that will be assigned for the virtual machine.

The actual virtual machine definitions are also pretty similar. In the AWS side you tell the subnet and security group of the vm when defining it. In the Azure and GCP sides you don’t have to link the security to the VM since in the Azure side the network security group is linked to the subnet and in the GCP side the firewall rule is linked to the VPC. But in overall all IaC code in all these three solutions are pretty similar.

### Infrastructure Code Comparisons

Now that we have three identical IaC solutions in three cloud platforms let’s see how many lines of code each solution comprises:

![](/img/2019-07-05-comparing-simple-gcp-infrastructure-demo-to-aws-and-azure_img_3.png)So, the number of files is pretty much the same. But there are some differences regarding the lines of code (LoC). I have provided an efficiency (Eff.) factor which is calculated dividing the LoC of each cloud with the lowest LoC (GCP: 345). This factor states that you need 40% more infrastructure code in the similar AWS solution and 10% more infrastructure code in the similar Azure solution as in the GCP solution. Word of caution — this is just one very small infrastructure demonstration and the lines of code comparison is done just for curiosity — I wouldn’t draw any general conclusions based on this comparison.

### The Longer Story

All three demonstrations provide README.md files which give a longer explanation regarding that particular demonstration — I encourage the reader to read those files to get a more detailed view regarding the demonstrations (e.g. more detailed explanations regarding Terraform backends in AWS, GCP and Azure sides etc.). At the end of the README files you can also find a detailed demonstration manuscript — you should be able to deploy the infrastructure to GCP, AWS and Azure clouds following the demonstration manuscripts. I tested the AWS and Azure deployments using both my Linux workstation and my virtual Windows 10 workstation, I tested the GCP deployment only using my Linux workstation — If someone wants to test the GCP demonstration manuscript (and possibly provide some powershell scripts instead of my bash scripts) I’m happy to receive a merge request.

### Conclusions

All three public clouds— AWS, GCP and Azure — are excellent cloud platforms and you can easily create any infrastructure solution using any of these cloud providers.

Using Terraform as an Infrastructure as Code tool you get synergy benefit you don’t get by using cloud providers’ native IaC tools: you can create infrastructure as code solutions in more than one public cloud using the same tool and therefore re-using your IaC knowledge — that’s why Terraform is my choice of tool when creating infrastructure as code in all “Big three”: GCP, AWS and Azure.

*The writer has three AWS certifications, two GCP certifications and one Azure certification and is working in *[*Tieto Corporation*](https://www.tieto.com/)* in Custom Software Development / Public Cloud team designing and implementing cloud native solutions in all three public cloud platforms. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
  