---
layout:	post
title:	"Comparing GCP Deployment Manager and Terraform"
categories: [blog, aws]
tags: [aws]
date:	2019-07-13
---

### Introduction

Last week I wrote a simple GCP demonstration using Terraform and wrote a blog post to compare GCP, AWS and Azure when using Terraform: “[Comparing Simple GCP Infrastructure Demo to AWS and Azure](https://medium.com/@kari.marttila/comparing-simple-gcp-infrastructure-demo-to-aws-and-azure-abbbe8496d31)”. This week I wrote the same GCP demonstration again but this time using [GCP Deployment Manager](https://cloud.google.com/deployment-manager/). In this new blog post I compare GCP Deployment manager and [Terraform](https://www.terraform.io/) when implementing cloud systems on the [Google Cloud Platform](https://cloud.google.com/). Both demonstrations can be found in Tieto / Public Cloud team’s [Github account](https://github.com/tieto-pc):

* Terraform: <https://github.com/tieto-pc/gcp-intro-demo>
* DM: <https://github.com/tieto-pc/gcp-intro-dm-demo>
There is a lot of introduction type material found regarding these tools in the net so I don’t provide any basic information about them — the reader is recommended to use Google search to find this kind of information. To keep this blog post short I just briefly provide my personal experiences and developer feelings regarding these two tools.

![](/img/2019-07-13-comparing-gcp-deployment-manager-and-terraform_img_1.png)

*GCP Intro Demonstration.*

### Terraform

[Terraform](https://www.terraform.io/) is a widely used tool to create cloud infrastructure to public clouds — and this is Terraform’s most compelling pro: If you do multi-cloud development (as I do — AWS, GCP and Azure) it is a really powerful benefit to have one tool to create IaC for all three cloud platforms. Terraform also provides powerful declarative language ([HCL](https://github.com/hashicorp/hcl)) for creating IaC solutions. All major cloud services are supported in Terraform and usually new major services are supported pretty soon they are launched by the cloud provider.

Terraform is a declarative language but compared to other declarative languages used by some public cloud provider native IaC tools (like JSON or YAML) Terraform provides nice constructs to modularize your cloud resources into logical re-usable entities. Terraform also provides nice ways to refer previously created entities and there are good Terraform plugins to widely used IDEs (e.g. I use IntelliJ IDEA and its Terraform plugin really makes coding Terraform solutions a joy).

I have already in my previous blog posts opened the Terraform solution structure I like to use in my Terraform based IaC solutions. You can also read more about that in the [README.md](https://github.com/tieto-pc/gcp-intro-demo/blob/master/README.md) file of the Terraform based GCP demonstration.

### GCP Deployment Manager

[GCP Deployment Manager](https://cloud.google.com/deployment-manager/) is the GCP cloud native IaC tool. When comparing it to AWS CloudFormation and Azure ARM I must say that I liked GCP Deployment Manager most. The most important reason for this is that Google recommends to create GCP resource templates using Python. So, using Python you have all the power of a full-fledged programming language to modularize and process your IaC solution the way you want.

### GCP DM Demonstration Solution Structure

I have provided a more complete documentation regarding the solution structure of my GCP Deployment Manager demonstration in its [README.md](https://github.com/tieto-pc/gcp-intro-dm-demo/blob/master/README.md) file, but let’s briefly provide here the most important parts of the solution.

The GCP Deployment Manager expects to find a GenerateConfig(context) function that it calls to get the JSON representation of the cloud resources you want to deploy to the GCP platform. So, basically you just need to create a Python file, implement the GenerateConfig function and in that function either directly write the JSON representation of your cloud resources, or modularize the solution so that this function just calls your internal modules and returns the list of resources that have been created in other modules. Even though this demonstration is rather simple I wanted to experiment how to modularize my solution a bit.

So, you first need to create a YAML file which tells GCP Deployment Manager which Python files are needed for your deployment. I provided a template for that YAML file in [deployment-template.yaml](https://github.com/tieto-pc/gcp-intro-dm-demo/blob/master/dm/deployment-template.yaml). That file also provides the values for various parameters that you are using in your Python files — a nice way to parameterize your cloud solution.

The file [deployment.py](https://github.com/tieto-pc/gcp-intro-dm-demo/blob/master/dm/deployment.py) is the “main” Python file for providing the cloud resources JSON representation and it provides the GenerateConfig function that Deployment Manager calls. As you can see, in that file I just call [vpc.py](https://github.com/tieto-pc/gcp-intro-dm-demo/blob/master/dm/vpc.py) and [vm.py](https://github.com/tieto-pc/gcp-intro-dm-demo/blob/master/dm/vm.py) files that do the heavy lifting of actually providing the JSON representations of respective cloud resources.

### Developing and Debugging a DM Solution

It would be a bit stupid to make some changes to the Python code, try to deploy it to the GCP using gcloud, wait and find out that your deployment failed for a simple syntax error. Therefore I created [mymain.py](https://github.com/tieto-pc/gcp-intro-dm-demo/blob/master/dm/mymain.py) module which I used to check that there were no syntax errors and that the final JSON representation looked the same as [Supported resource types](https://cloud.google.com/deployment-manager/docs/configuration/supported-resource-types) Google documentation.

I used [PyCharm](https://www.jetbrains.com/pycharm/) which is my favourite Python IDE. It is easy to create a Run Configuration in PyCharm and run the main module. In that file I first created a GCP context simulator simulateContext() which I used to simulate how GCP would inject certain parameters to the deployment (actually I read the exact same Yaml configuration that I use in the actual deployment). So, running the main() function you can simulate how GCP would create the JSON representation of the infrastructure resources and use this output as a debugging tool.

### Comparing Terraform and Deployment Manager

Now that I implemented the exact same GCP demonstration using both Terraform and Deployment Manager it is easy to compare the two solutions and the tools.

As I already mentioned the major pro with Terraform is that you can learn one tool and language and use it with all “Big Three” cloud providers: AWS, Azure and GCP. This is a real advantage if you do multi-cloud development. Terraform has also proved in the past years to be really vibrant tool and Hashicorp continuously adds new cloud services to Terraform and also develops the HCL language itself (see new [0.12 version features](https://www.hashicorp.com/resources/introducing-terraform-0-12)).

But GCP Deployment Manager is not bad either, on the contrary, I really liked it. I had read about the Deployment Manager when studying for the two GCP certifications I did this summer, and also did some simple lab exercises / personal experimentations using it (e.g. deploy simple VM resource with a basic YAML configuration), but only now I tried to create a bit more complete demonstration with a VPC, subnet, firewall and a VM. And basically with very little Deployment Manager experience I could create a complete solution with just a couple of days — which proves that you can start to be very productive with the Deployment Manager right away. And using a real programming language — Python — you can modularize the solution the way you want. It is also very easy to create in-house development tools to create Deployment Manager solutions (the way I did with the simple Deployment Manager simulator I explained earlier).

There was one issue with Deployment Manager, though. In my corporate GCP organization it is not possible to use one admin project to host the deployment and do the deployment to another infra project so that also the infra project is created as part of the IaC solution (and use a service account in the admin project to make the deployment). I had a conversation regarding this with our GCP organization admin and the answer was that this is restricted due security policy — they want to be rather restrictive who can create GCP projects. Therefore in my DM demonstration I created the infra project using gcloud and also used the infra project to host the DM deployment (read more about that in the [README.md](https://github.com/tieto-pc/gcp-intro-dm-demo/blob/master/README.md)). With Terraform I could create an admin project to host the terraform state and in the IaC solution to create the infra project as part of the IaC solution.

With AWS CloudFormation and Azure ARM I always had a bit of a discomfortable feeling compared using Terraform. But GCP Deployment Manager was a delightful surprise among these cloud native IaC tools.

### Conclusions

GCP Deployment Manager is a powerful tool to create cloud solutions to the Google Cloud Platform. So, you have two excellent tools to choose when working with GCP: Terraform and GCP Deployment Manager. I encourage you to create a simple cloud demonstration using both tools and then decide which one to use.

*The writer has three AWS certifications, two GCP certifications and one Azure certification and is working in *[*Tieto Corporation*](https://www.tieto.com/)* in Custom Software Development / Public Cloud team designing and implementing cloud native solutions in all three public cloud platforms. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
  
