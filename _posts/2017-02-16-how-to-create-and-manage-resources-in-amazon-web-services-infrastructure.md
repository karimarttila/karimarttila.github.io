---
layout:	post
title:	"How to Create and Manage Resources in Amazon Web Services Infrastructure?"
categories: [blog, aws]
tags: [aws]
date:	2017-02-16
---

  ![](/img/1*eQsT3OdHCbEGmmFx40LV3Q.png)Example Terraform Script to Create AWS Resources.We continue our AWS related blog series that we started in our first article “[**Use AWS Services as Building Blocks to Implement Your Enterprise System**](https://medium.com/tieto-developers/use-aws-services-as-building-blocks-to-implement-your-enterprise-system-598676a0ee49#.86dk7lu31)” . In this new article we talk about our experiences regarding how to create and manage resources in AWS.

### Introduction

There are many different ways and tools to create and manage resources in AWS. Let’s list here the most obvious ones:

* Manually using AWS Console
* Custom scripts using AWS CLI
* AWS CloudFormation
* Terraform
Next we’ll elaborate the solutions and tools in more detail and list some pros and cons regarding the options.

### 1. Manually Using AWS Console

[AWS Console](https://aws.amazon.com/console/) provides Web based interface to various AWS resources. You can create and manage any AWS resource using AWS Console. AWS Console is a good tool to browse the AWS resources and also to create certain temporary resources for experimentation or temporary testing. A typical way to use AWS Console is to use the [CloudWatch](https://aws.amazon.com/cloudwatch/) tool which provides excellent dashboard views to monitor any AWS resource and see correlations between different AWS resources using complex aggregated graphs.

But a typical AWS project may have hundreds of bigger (RDS, EC2…) and smaller (S3 lifecycle rules, metric alarms) resources. And if you have many environments (development, testing environments for the team, customer and stakeholders, performance testing environment, QA environment and production environment) you multiply the resources by the number of your environments. E.g. if you have 200 AWS resources in one environment and you have 6 environments: 6 x 200 = 1200 resources. The conclusion is: In no way it is viable to manage that many resources without strict automation.

### 2. Custom Scripts Using AWS CLI

![](/img/1*qlDvTHcUpZ2evPpIHRbA1w.png)AWS CLI code snippet example using Bash to reset autoscaling groups.[AWS CLI](https://aws.amazon.com/cli/) (command line interface) is a marvelous tool for implementing small custom scripts if you need to interact with AWS resources for some specific reason. E.g. We have used AWS CLI to implement various logging streaming tools, a tool to delete old AMIs (Amazon Machine Images), tools to shutdown and start resources etc. We have used Python as a glue to call AWS CLI and then process the returned JSON structures and make new calls based on those results. Another way is to use SDKs that AWS provides e.g. for Python and Java. For small tasks as examples given above I like to work directly with AWS CLI and Python since it is fast to test AWS CLI commands with Bash and then add the commands to Python scripts.

But is it viable to use AWS CLI to manage 1200 resources? The answer is no. It would be extremely stupid to make a custom solution to manage your whole AWS infrastructure with custom scripts since there are two excellent solutions just for this purpose.

### 3. AWS CloudFormation

[AWS CloudFormation](https://aws.amazon.com/cloudformation/) is AWS native way to manage AWS infrastructure. We have not used CloudFormation in our project, but the basic idea is the same as with Terraform: you maintain your configuration as code. In AWS CloudFormation you manage the resources as templates, which are basically JSON files. The good side using AWS CloudFormation is that AWS provides many ready-to-use CloudFormation templates for various topologies and services you want to use. The downside is that many complain the JSON file structure to be a bit awkward to use ([now also supports Yaml](https://aws.amazon.com/about-aws/whats-new/2016/09/aws-cloudformation-introduces-yaml-template-support-and-cross-stack-references/)). You can keep your templates in version control and have audit trail regarding the changes you have done in your environments. ([CloudTrail](https://aws.amazon.com/cloudtrail/) service also provides you good perspective to the changes done in your environments.) Templates provide a good way to parameterize your environments (e.g. smaller RDS in development env, bigger RDS with multi availability zone replica in production env) and make many environment copies of one environment template. Using CloudFormation doesn’t cost anything — you only pay for the actual resources you use after you create them.

### 4. Terraform

[Terraform](https://www.terraform.io) is a tool from HashiCorp, a company which has created a bunch of excellent products used in cloud based work. Terraform is a tool a bit like CloudFormation, but it uses its own DSL to define Cloud resources. The good side is that using Terraform you can create resources in a similar manner to Azure and AWS. Hashicorp made a good decision not to create an abstraction layer to Terraform but instead it provides a way to create either AWS or Azure resources depending on your target cloud (but the DSL and the process is basically the same). We haven’t used Terraform to create resources to Azure but we have extensive experience how to use Terraform to create resources to AWS. For AWS resource management Terraform has proved truly to be an excellent tool.

Using Terraform you can easily modularize your environment entities and create those entities and related sub-entities as one module. See the example Terraform script in the beginning of the article. The Terraform script could be in some AWS project in which one needs to create a common Auto-scaling Group module to be used for various EC2 instances. The script comprises AWS Launch configuration, App Instance Profile, Security Group configuration and the Auto-scaling group in one module. Then you can refer to this common module (and inject required parameters e.g. “application\_name”, instance type…) from all of your various Terraform environment scripts (dev, test, prod…). The solution provides good modularity and maintenance for your infrastructure code. The example script also provides one excellent Terraform/AWS best practice: Provide all your AWS entities with the “<environment>-<application>-” prefix — then it is a lot easier to search your entities in AWS Console when you can filter easily the resources based on some environment or application name they belong to (not to mention how much easier this makes creating Python/Bash scripts to use AWS CLI to refer AWS entities).

Using Terraform is quite safe since making changes to your AWS infrastructure is divided into two distinct Terraform steps: First you run a Terraform plan. When running Terraform plan Terraform compares its previous known state to the actual AWS state and to the current configuration state, it then calculates the delta between the current AWS state and the state you have defined in your new configuration, and provides you a pretty detailed description, which resources are to be deleted, which are to be created and which are to be changed. If you are new to Terraform the report can be a bit intimidating at first. But once you get more experience of AWS resources and their internal dependencies you learn to read Terraform reports and see if there are anything out of ordinary. If you are satisfied with the report you can run Terraform apply command which actually makes the changes.

In previous project we used Terraform to manage all our AWS resources in all of our 5 AWS environments which thanks to Terraform are pretty much exact copies of each other, and the changes in the environments are nicely moduralized and parameterized (e.g. smaller instances in dev environment, bigger and more instances in production environment).

### Conclusion

What ever tool you choose to manage your Cloud resources we give you one definite recommendation: Do not do it manually. You have to do it using some tool which sees your environment as code. Good candidates are CloudFormation and Terraform.

Both writers are AWS Certified Solutions Architects Associate, architecting and implementing AWS projects in Tieto CEM Finland. If you are interested about starting a new AWS project in Finland, you can contact us with firstname.lastname at tieto.com.

Kari Marttila & Timo Tapanainen

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
* Timo Tapanainen’s Home Page in LinkedIn: <https://www.linkedin.com/in/timo-tapanainen/>
  