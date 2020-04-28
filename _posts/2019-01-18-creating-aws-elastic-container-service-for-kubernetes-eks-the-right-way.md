---
layout:	post
title:	"Creating AWS Elastic Container Service for Kubernetes (EKS) the Right Way"
categories: [blog, aws]
tags: [aws]
date:	2019-01-18
---

  ![](/img/2019-01-18-creating-aws-elastic-container-service-for-kubernetes-eks-the-right-way_img_1.png)IntelliJ IDEA with Terraform Plugin

### Introduction

I already did a small exercise how to create a managed Kubernetes service infra in the Azure side using Terraform — see my previous blog post “[Creating Azure Kubernetes Service (AKS) the Right Way](https://medium.com/@kari.marttila/creating-azure-kubernetes-service-aks-the-right-way-9b18c665a6fa)”. Now I wanted to see how the same Kubernetes managed service can be created using [AWS Elastic Container Service for Kubernetes](https://aws.amazon.com/eks/) (EKS). I describe in this blog post about my experiences regarding that.

You can find the project in my [Github account](https://github.com/karimarttila/aws/tree/master/simple-server-eks).

### Use an Example

The best way to setup a bit more complex cloud infra is to use an example from a reliable source — so that you can trust that the example can be used as a working reference when you mold it to your own purposes. I used document “[Terraform AWS EKS Introduction](https://learn.hashicorp.com/terraform/aws/eks-intro)” and the related git repo “[eks-getting-started](https://github.com/terraform-providers/terraform-provider-aws/tree/master/examples/eks-getting-started)” as an example and did some modifications like modularized the solution, used my own Terraform conventions etc. Using a working example is pretty important if you need to debug your own solution. One good cloud infra best practice is to deploy the working solution and your own non-working solution into the same cloud account (in different VPCs etc) and then compare the resources side by side.

### The Solution

The solution is pretty much the same as in the Azure side: create AWS S3 bucket to store the Terraform state, create AWS configuration in a modular manner using Terraform, and deploy the infra incrementally to AWS when you write new resource configurations.

#### 1. Create AWS S3 Bucket and DynamoDB Table for Terraform Backend

You don’t need this step for small pocs but I like to do things with best practices. So, the best practice is to store the Terraform state in a place where many developers can access the state (i.e. in the cloud, naturally). In the AWS side the natural place for the Terraform backend is S3. You should also create a DynamoDB table as a locking mechanism so that Terraform can take care of locking of the state so that many developers are not running cloud infra updates concurrently breaking the coherence of the infra.

#### 2. Create AWS Infra Code Using Terraform

The next step is to create the AWS cloud infra code using Terraform (see directory [terraform](https://github.com/karimarttila/aws/tree/master/simple-server-eks/terraform)). I usually create a “main” configuration part which defines what modules comprise the main configuration: file [env-def.tf](https://github.com/karimarttila/aws/blob/master/simple-server-eks/terraform/modules/env-def/env-def.tf). If you look at that file you see that we create here the **DynamoDB tables** for the application, a **VPC** (probably not necessary but I followed the original example here), **ECR repositories** for storing the Docker images, **EKS main infra** with all resources EKS needs (security group, security group rules, IAM role, some policies, and finally the actual EKS cluster itself), and the **EKS worker nodes** and their related infra (security group, IAM role, policies, launch configuration for EC2 images and auto scaling group).

You can look at the configuration files to get a more detailed description of the infra (or look at the original example). Let’s have a short discussion regarding the EKS infra anyway. From cloud infra developer’s point of view the EKS control plane is pretty well abstracted — you can create the control plane with one Terraform configuration (“aws\_eks\_cluster”). But the EKS worker nodes (into which you deploy your Kubernetes deployment configuration — the pods) are created pretty explicitly and transparently using [AWS Launch Configuration](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchConfiguration.html) (which provide a template for EC2 images) and [AWS Auto Scaling Group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html) (which provide the template how the elasticity, i.e. the scaling out/in, works). This is a kind of good and bad thing. Good in that sense that you are pretty much in control how the worker node plane works. Bad in that sense that it complicates the cloud infra for novice developers and you have to be pretty careful gluing everything and not make any mistakes in some tags, security group references etc (as I did a couple of times and had to spend a couple of hours debugging why the cluster didn’t work properly).

An interesting detail is the use of tags to glue EKS control plane and worker nodes together. As a typical engineer I first just glimpsed the example, hardly read the instructions and started to write code skipping most of the tags and thought that they are used just for telling developer some additional information about the resources (as usually is the case with tags). I didn’t realize that EKS actually uses some of these tags to glue things together (e.g. key = “kubernetes.io/cluster/${var.eks\_cluster\_name}” value = “owned”).

And finally there is the environments directory ([envs](https://github.com/karimarttila/aws/tree/master/simple-server-eks/terraform/envs/dev)) in which I create the different environment definitions and parameterize things (in this exercise just the resource prefix, env etc., but you could parameterize also different VM sizes for different environments).

#### 3. Deploy Cloud Infra!

As I already explained in the Azure blog post you don’t have to create the whole cloud infra in one shot before applying it to the cloud provider. I create the cloud infra incrementally, resource by resource. I try “terraform plan”, and if everything looks good I apply the new resources using command “terraform apply”. This way you can create the cloud infra incrementally which is pretty nice.

#### 4. Smoke Test with EKS

This time I did try to deploy the application (the single-node version anyway) to the EKS cluster before I wrote this blog post (see the experiences in the related Azure AKS blog post). For trying the EKS cluster you need the Kubernetes context. If you haven’t installed [aws-cli](https://github.com/aws/aws-cli) or [aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html) install them now. Then you can query the Kubernetes context and use it with kubectl tool

# First use terraform to print the cluster name:   
AWS\_PROFILE=YOUR-AWS-PROFILE terraform output -module=env-def.eks  
# Then get the Kubernetes cluster context:   
AWS\_PROFILE=YOUR-AWS-PROFILE aws eks update-kubeconfig --name YOUR-EKS-CLUSTER-NAME  
# Check the contexts:  
AWS\_PROFILE=YOUR-AWS-PROFILE kubectl config get-contexts # => You should find your new EKS context there.  
# Check initial setup of the cluster:  
AWS\_PROFILE=YOUR-AWS-PROFILE kubectl get all --all-namespaces # => prints the system pods...If you see all system pods up and running you are good to go (I didn’t, see my cloud infra debugging experiences in the [README.md](https://github.com/karimarttila/aws/blob/master/simple-server-eks/README.md) file).

You are not yet quite ready for the actual Kubernetes deployments since in EKS you need to join the worker nodes to the EKS control plane. For this you need a Kubernetes config map file — you can write it manually but the example provided nice mechanism based on Terraform output how to get the configuration:

# Store the config map that is needed in the next command.  
# NOTE: Store file outside your git repository.  
AWS\_PROFILE=YOUR-AWS-PROFILE terraform output -module=env-def.eks-worker-nodes > ../../../tmp/config\_map\_aws\_auth.yml   
emacs ../../../tmp/config\_map\_aws\_auth.yml # => Delete the first rows until "apiVersion: v1"   
AWS\_PROFILE=YOUR-AWS-PROFILE kubectl apply -f ../../../tmp/config\_map\_aws\_auth.yml  
# In terminal 2:  
while true; do echo "*****************" ; AWS\_PROFILE=YOUR-AWS-PROFILE kubectl get all --all-namespaces ; sleep 10; doneAll right! If you did everything properly you have a working Kubernetes cluster ready for your Kubernetes deployments! You can use the while indefinite loop to see how the worker nodes are getting created.

### Comparing the Azure AKS vs AWS EKS

Let’s make a short comparison between Azure AKS and AWS EKS regarding my experiences. Let’s first compare the cloud infra configuration files and lines of code:

| Cloud | Files | LoC |  
| - - - - - - - -| - - - | - - - |  
| Azure | 24 | 418 |  
| AWS | 23 | 674 |  
| AWS - no VPC | 20 | 589 |The comparison is not exactly honest since in the EKS side we are creating a VPC (and in the Azure side not). There are 3 VPC related files and 85 lines of code — the third row “AWS — no VPC” sums the results without comprising the VPC related infra code. 100*(589–418)/418 = 41%, i.e. we need 41% more code in the EKS side even though we don’t take the VPC code into consideration. Is this a good thing for Azure? Yes and no. The Azure infra code works more in a wizard way abstracting away things that are more explicit and transparent in the AWS side — a good thing if you are novice and just want the Kubernetes cluster up and running, but if you want more control under the hood the EKS solution might be better in that (the Azure solution just defines an “agent pool” as part of the “azurerm\_kubernetes\_cluster” — it doesn’t say what these entities actually are and e.g. how the autoscaling works; in the AWS side you explicitly create an autoscaling group, launch configuration and can even choose the AMI (Amazon Machine Image) which is used to run the worker nodes). So, both sides have pros and cons.

### Some Observations Regarding EKS Infra Development

There were a couple of observations that I’d like to emphasize here. The first one is that the AWS EKS infra code is more explicit and transparent but also more complex than the equivalent Azure AKS infra code. Especially the worker node configuration is a lot more complex in the AWS side (and almost non-existing in the Azure side). Since the solution is more complex there is also more room for errors (as I noticed — you have to be careful with the tags, security group references between the EKS control plane and worker nodes etc.).

The second observation is that creating the EKS control plane takes quite a lot of time — some 10 minutes. This doesn’t sound much but if you need to develop and test your Kubernetes deployment in the EKS environment you don’t want to start your development session waiting 10 minutes to create the environment and another 10 minutes to tear it down.

### Conclusions

Creating a Kubernetes cluster as a managed service is pretty straightforward using Terraform both in the Azure and AWS sides. The AKS and EKS solutions provide a bit different abstractions to the Kubernetes clusters but both seem to be solid environments to run your Kubernetes applications.

  