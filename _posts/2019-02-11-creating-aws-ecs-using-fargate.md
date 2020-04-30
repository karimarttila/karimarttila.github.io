---
layout: post
title: "Creating AWS ECS Using Fargate"
category: [aws]
tags: [aws, ecs, fargate, cloud, iac, terraform]
date:	2019-02-11
---

![](/img/2019-02-11-creating-aws-ecs-using-fargate_img_1.png)

*Main AWS Services Used in the Demonstration.*

### Introduction

I previously created infrastructure for running Docker containers in the AWS Kubernetes managed service ([EKS](https://aws.amazon.com/eks/) — see the blog article: [Creating AWS Elastic Container Service for Kubernetes (EKS) the Right Way]({% post_url 2019-01-18-creating-aws-elastic-container-service-for-kubernetes-eks-the-right-way %}). Now I wanted to see if creating a Docker runtime using AWS [ECS](https://aws.amazon.com/ecs/) — Elastic Container Service is going to be as simple. It turned out not to be quite that simple and let’s see why.

I created this demonstration for my new unit: Tieto / Application Services / Public Cloud, and therefore the demonstration can be found in the Public Cloud team’s [Github repository](https://github.com/tieto-pc/aws-ecs-fargate-demo).

### AWS Solution

The demonstration uses a dedicated [VPC](https://aws.amazon.com/vpc/). There are two public subnets for the [Application load balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) (ALB) and two private subnets for the ECS infrastructure in two [availability zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) (in the diagram [ECS](https://aws.amazon.com/ecs/) and [Fargate](https://aws.amazon.com/fargate/) are depicted in the bigger AZ just for diagram clarity). There is also a public subnet for the NAT infrastructure for ECS to pull public images. All subnets have a dedicated [security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) which allows only inbound traffic from those security groups that need them.

There is also an internet gateway for NAT, a [S3](https://aws.amazon.com/s3/) Bucket for ALB logs, an [ECR](https://aws.amazon.com/ecr/) for storing Docker images used by ECS and an [IAM](https://aws.amazon.com/iam/) role for running the ECS tasks.

You can use ECS using custom EC2 and Auto-scaling setup or you can leave the heavy lifting of managing EC2 to AWS — using Fargate. In this demonstration I’m using [Fargate](https://aws.amazon.com/fargate/).

### Terraform Code

I am using [Terraform](https://www.terraform.io/) as an [infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) (IaC) tool. Terraform is very much used both in the AWS and Azure side and one of Terraform’s strenghts compared to cloud native tools (AWS / [CloudFormation](https://aws.amazon.com/cloudformation) and Azure / [ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates)) is that you can use Terraform with many cloud providers, you have to learn just one infra language and syntax, and Terraform language (hcl) is pretty powerful and clear.

If you are new to infrastructure as code (IaC) and terraform specifically let’s explain the high level structure of the terraform code first. Project’s terraform code is hosted in the [terraform](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/terraform) folder.

It is a cloud best practice that you should modularize your infra code and also modularize it so that you can create many different (exact) copies of your infra as you like re-using the infra modules. I use a common practice to organize terraform code in three levels:

1. **Environment parameters**. In the [envs](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/terraform/envs) folder we host the various environments. In this demonstration we have only the dev environment, but this folder could have similar environment parameterizations for qa, perf, prod environments etc.
2. **Environment definition**. In the [env-def](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/terraform/modules/env-def) folder we define the modules that will be used in every environment. The environment injects the environment specific parameters to the env-def module which then creates the actual infra using those parameters by calling various infra modules and forwarding environment parameters to the infra modules.
3. **Modules**. In [modules](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/terraform/modules) folder we have the terraform modules that are used by the environment definition (env-def, also a terraform module itself). There are three modules for the main services used in this demonstration: [VPC](https://aws.amazon.com/vpc/), [ECR](https://aws.amazon.com/ecr/) and [ECS](https://aws.amazon.com/ecs/) (and a couple of other utility modules).
### Demo Application

The demo application used in this demonstration is a simple Java REST application that simulates a CRM system : [java-simple-rest-demo-app](https://github.com/tieto-pc/java-simple-rest-demo-app).

The demo application is dockerized — the Docker image is used by the ECS. The actual docker image is hosted in the ECR registry. See [docker](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/docker) folder for scripts how to build and tag/push the image to ECR.

### Terraform Modules

#### VPC Module

The [vpc](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/terraform/modules/vpc) module turned out to be much bigger than I originally thought I need. The reason mainly was that I wanted to make the demonstration a bit more real-like, e.g. putting ECS to private subnet, providing application load balancer and other security / redundancy features. Those decisions rippled to the VPC in that sense that I needed to add some extra infra boilerplate, e.g. needed to add a nat public subnet since ECS cannot pull images from the public repositories unless it has a route table to a nat in a public subnet which directs traffic to internet gateway etc (I could have provided a private link endpoint if I were only testing with ECR but I wanted to test infra using images in public repos as well).

#### ECR Module

The [ecr](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/terraform/modules/ecr) module is pretty simple: it defines the only repository we need in this demonstration, the “java-crm-demo” repository.

#### ECS Module

The [ecs](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/terraform/modules/ecs) module also turned out to be more challenging than I thought it would be before starting this exercise. There is quite a lot of stuff in the ecs module: IAM role for ECS task execution ( + role policy), ECS cluster, ECS task definition, S3 bucket for access logs, Application load balancer (+ listener and target group) and ECS service.

The ECS even with using Fargate is a bit complex to configure.

#### Resource Groups Module

The [resource-groups](https://github.com/tieto-pc/aws-ecs-fargate-demo/tree/master/terraform/modules/resource-groups) module defines a dedicated resource group for each tag key I use in all AWS resources that support tagging. The tag keys are:

- Name: <prefix>-<env>-<name-of-the-resource>, e.g. “aws-ecs-demo-dev-vpc” (not used in resource groups, of course)  
- Env: <env>, e.g. “dev”  
- Environment: <prefix>-<env>, e.g. “aws-ecs-demo-dev”  
- Prefix: <prefix>, e.g. “aws-ecs-demo”  
- Region: <region>, e.g. “eu-west-1  
- Terraform: “true” (fixed)This way you can pretty easily search the resources. Examples:

* Env = “dev” => All resources in all projects which have deployed as development (“dev”).
* Prefix = “aws-ecs-demo” => All AWS ECS demo resources in all envs (dev, perf, qa, prod…).
* Environment = “aws-ecs-demo-dev” => The resources of a specific terraform deployment (since each demo has dedicated deployments for all envs), i.e. this deployment.
After deployment open AWS Console => “Resource Groups” view => Saved Resource Groups => You see the 5 resource groups => Click one and you see all resources regarding that tag key and value (of resources that support tagging in AWS). This is a nice way to search what resources are in various envs (dev, qa, prod), for certain system in any environment (prefix) etc.

### Testing the Application Running in ECS

Run command ‘AWS\_PROFILE=YOUR-PROFILE terraform output -module=env-def.ecs’ => you get the application load balancer DNS. Use it to curl the ALB:

curl <http://ALB-DNS-HERE:5055/customer/1>  
# => Should return: {“ret”:”ok”,”customer”:{“id”:1,”email”:”[kari.karttinen@foo.com](mailto:kari.karttinen@foo.com)”,”firstName”:”Kari”,”lastName”:”Karttinen”}}### Conclusion

Configuring everything using ECS turned out to be a bigger task than I originally thought. When comparing experiences between ECS and EKS I think EKS is easier to setup from the infra coding point of view. On the other hand EKS might be a bit overkill to run just one docker container (with redundancy, of course) in which case ECS is a more streamlined way even though the initial infra setup might be a bit intimidating for the first time.

*The writer has two AWS certifications and one Azure certification and is working in *[*Tieto Corporation*](https://www.tieto.com/)* in Application Services / Application Development / Public Cloud team designing and implementing cloud native projects. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
  
