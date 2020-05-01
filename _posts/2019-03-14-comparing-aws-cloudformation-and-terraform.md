---
layout: post
title: "Comparing AWS CloudFormation and Terraform"
category: [aws]
tags: [aws, cloud, iac, cloudformation, terraform]
date: 2019-03-14
---

![](/img/2019-03-14-comparing-aws-cloudformation-and-terraform_img_1.png)

*Terraform and CloudFormation in a Family Portrait.*

### Introduction

I created two simple infrastructure demos using [Terraform](https://www.terraform.io/) for training new cloud learners in my new unit: one [AWS demo](https://github.com/tieto-pc/aws-intro-demo) and one [Azure demo](https://github.com/tieto-pc/azure-intro-demo) and compared the demos in my previous blog post [Comparing Simple AWS and Azure Infrastructure Demos]({% post_url 2019-03-11-comparing-simple-aws-and-azure-infrastructure-demos %}). This time I implemented the same AWS demo using AWS native IaC tool, [CloudFormation](https://aws.amazon.com/cloudformation/): [AWS CloudFormation demo](https://github.com/tieto-pc/aws-intro-cloudformation-demo). In this blog post I compare these identical AWS simple intro demos created by two different IaC tools:

* **Terraform**: [aws-intro-demo](https://github.com/tieto-pc/aws-intro-demo)
* **CloudFormation**: [aws-intro-cloudformation-demo](https://github.com/tieto-pc/aws-intro-cloudformation-demo)


### Terraform vs CloudFormation

In this chapter I compare the two tools based on my experience when using them. I also give (very opinionated and personal) points (0–5, 5 highest) to tools regarding how I feel they excel in that category. At the end of the chapter I gather the results and give my personal recommendation which tool to use.

**Disclaimer**: I have used a lot more Terraform than CloudFormation so my opinions here are naturally pretty skewed.

#### 1. Cross Platform

Terraform's major benefit is that you can use it with many cloud providers: you just need to learn one language (hcl) and workflow and use it with AWS, Azure, GCP etc. This is a real benefit for developers like me who do both AWS and Azure — one of the reasons why Terraform is my choice of IaC tool.

Points: Terraform: 5, CloudFormation: 0.

#### 2. Language

I like Terraform's [hcl language](https://www.terraform.io/docs/configuration/syntax.html). Hcl's interpolation syntax is very powerful and it lets you to reference variables, attributes etc. Combined with a good IDE plugin this makes infrastructure coding a breeze. CloudFormation uses either JSON or Yaml which are as bare notation languages not as powerful as hcl. With CloudFormation you can also reference parameters, other stacks etc but in overall feeling I think hcl makes me more productive. CloudFormation on the other hand has good support for various [conditional functionality](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html). Terraform has the [count](https://www.terraform.io/intro/examples/count.html) parameter which lets you make certain things easy (like creating N identical resources) and certain things a bit harder (like if else type conditional structure — e.g: ```count = "${1 - var.my_workstation_is_linux}```. CloudFormation has also [WaitCondition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-waitcondition.html) and [CreationPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-creationpolicy.html) which can be important in certain deployment situations in which you have to wait before certain condition is satisfied.

Points: Terraform: 4, CloudFormation: 3.

#### 3. Modularity

Modularity is easier to implement to your infrastructure code using Terraform than using CloudFormation. For a software developer background guy like me Terraform's modules are a very natural way to modularize and reuse your infrastructure code. You can modularize CloudFormation code using [Nested stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html) but the mechanism to store nested stacks in S3 is much clumsier than Terraform modules. You cannot modularize your CloudFormation stack itself as easily as the Terraform code, but with CloudFormation you can use [cross stack reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/walkthrough-crossstackref.html), i.e. you can reference entities in one stack from another stack.

Points: Terraform: 4, CloudFormation: 3.

#### 4. Validation

Both tools provide a validation mechanism before you apply the infrastructure code, but a bit differently. Using CloudFormation you can ask to [validate the stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-validate-template.html) — but this means only checking the syntax of your stack; to validate if you are actually able to deploy the stack you need to deploy it. If you want to update some resources you can use [CloudFormation Change sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html) which lets you review the changes before you apply them. Terraform provides a similar plan phase which provides that same information — you get to see what resources are modified, deleted or created before you actually deploy the infrastructure code.

Points: Terraform: 4, CloudFormation: 4.

#### 5. Readability

Look at the family portrait screen shot at the beginning of this blog post. I think Terraform is a lot more readable for the following reasons: 
1. Terraform's hcl language is more readable than pure json/yaml that you need to use with CloudFormation. 
2. Terraform's modularity also makes the code more readable. Some CloudFormation developers may not agree with me regarding the curly braces and dollar signs but once you know the syntax those make perfect sense for readability.

Points: Terraform: 4, CloudFormation: 2.

#### 6. Maintainability

Personally I think Terraform is more maintainable due to its better modularity and how you can create resources like functions. CloudFormation's way to make JSON/Yaml sections for resources does not appeal to me the same way.

Points: Terraform: 4, CloudFormation: 2.

#### 7. Tooling

I really love IntelliJ IDEA's [Terraform / HCL plugin](https://plugins.jetbrains.com/plugin/7808-hashicorp-terraform--hcl-language-support). Using the plugin with IntelliJ idea makes the infrastructure coding a breeze — your IDE is aware of all available parameters of the resource you are editing, IDE suggests other modules, hcl variables and locals in your code. I also used IntelliJ IDEA's [CloudFormation plugin](https://plugins.jetbrains.com/plugin/7371-aws-cloudformation) when working with the CloudFormation aws intro demo version — it is pretty good as well: it can suggest the elements in the yaml code the same way as the Terraform plugin. One additional point using CloudFormation is its Designer — you get to see a visual diagram of the resources and their dependencies. For me as a developer that is not that important — the Terraform module structure gives me pretty much the same visualization.

Points: Terraform: 4, CloudFormation: 4.

#### 8. AWS Console Support

Since CloudFormation is a native AWS IaC tool it provides excellent support in the AWS Console, of course. You can use the AWS Console to monitor the progress of the deployment, see all deployment events, check various errors, integrate CloudFormation to CloudWatch etc. For Terraform there is no AWS Console support or integration to AWS services like CloudWatch.

Points: Terraform: 0, CloudFormation: 5.

#### 9. Workflow

In a powerpoint slide CloudFormation's rollback mechanism (if one service deployment of your stack fails CloudFormation rolls back the whole stack) may look good and in SysOps side that may be what you actually want. But in the development side that is actually not what you want in most cases — if you have a large CloudFormation stack code and try to deploy it and e.g. after 20 minutes some service fails and CloudFormation rolls back everything this is not that good workflow. Terraform's workflow — "create all resources you can and stop if some service deployment fails" is a lot more natural for a developer. Of course in development you can create your stack incrementally service by service and try the deployment every once in a while, so this is not that big issue in the CloudFormation side either. On the other hand in SysOps world the rollback feature could a life saver — if your deployment fails you don't get stuck in a non-functional infrastructure but you rollback to the previous working state. I give Terraform one point more anyway since I'm a developer and the "create all resources you can and stop if some service deployment fails" workflow suits better for developing new cloud native infrastructure projects.

Points: Terraform: 4, CloudFormation: 3.

#### 10. Error Message Understandability

Most of the error messages Terraform spits are pretty obvious. But if something weird happens Terraform error messages can be rather difficult to interpret, especially for novices. CloudFormation and using the Console on the other hand makes error messages more readable.

Points: Terraform: 3, CloudFormation: 4.

#### 11. Infrastructure State Management

Both tools provide good state management. Terraform provides a [backend state mechanism](https://www.terraform.io/docs/backends/state.html) and CloudFormation keeps track of your stack deployments.

Points: Terraform: 3, CloudFormation: 4.

#### 12. Infrastructure Changes Auditing Support

Since CloudFormation is a native AWS IaC tool it integrates with AWS CloudTrail which lets you see who did and what with CloudFormation. With Terraform you get the API calls regarding resource creation/modification/deletion requests in CloudTrail, of course, but not regarding Terraform usage itself.

Points: Terraform: 1, CloudFormation: 5.

### Summary

Here are the results of my opinionated and skewed comparison. I use a factor ("Fa": 1–2) to provide some emphasis how important this feature is for me as an infrastructure developer (a SysOps guy might choose different factors and different original points). So, the fields are: Fa = Factor (1–2), OP = Original points, FP = Factored Points.

![](/img/2019-03-14-comparing-aws-cloudformation-and-terraform_img_2.png)

So, both tools seem to be pretty good: the original points (40–39) are almost identical. When I multiply original results with my personal importance factors I get some differences (69–59) — but as I already mentioned many times: these are personal opinions.

Which tool to choose then? First ask your customer, of course. If your customer is using or requesting to use a specific IaC tool, then use that. Otherwise if you are a new cloud practitioner: learn both. At least in that way that you are fluent with one tool and know the basics of the other tool.

### Conclusions

Both Terraform and CloudFormation are pretty good when it comes to infrastructure coding. **The most important decision is to use an IaC tool** than which tool to use (I covered this decision a bit in my previous articles [How to Create Infrastructure as Code for AWS and Azure]({% post_url 2019-02-28-how-to-create-infrastructure-as-code-for-aws-and-azure %}) and [How to Create and Manage Resources in Amazon Web Services Infrastructure?]({% post_url 2017-02-16-how-to-create-and-manage-resources-in-amazon-web-services-infrastructure %}).

Make an infrastructure project using both [CloudFormation](https://aws.amazon.com/cloudformation) and [Terraform](https://www.terraform.io) (and why not evaluate also [Pulumi](https://www.pulumi.com/)) and make your decision based on your own priorities which tool made your life as an infrastructure coder more productive.

*The writer has two AWS certifications and one Azure certification and is working at [Tieto Corporation](https://www.tieto.com/) in Application Services / Public Cloud team designing and implementing cloud native solutions. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
