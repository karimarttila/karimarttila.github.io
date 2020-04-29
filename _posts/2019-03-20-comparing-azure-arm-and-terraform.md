---
layout: post
title: "Comparing Azure ARM and Terraform"
category: [azure]
tags: [azure, arm, terraform, iac]
date:	2019-03-20
---

![](/img/2019-03-20-comparing-azure-arm-and-terraform_img_1.png)

*Terraform and ARM infrastructure code in a family portrait.*

### Introduction

This is the final blog post regarding the AWS and Azure intro demonstrations that I created using native tools (AWS: CloudFormation and Azure: ARM) and Terraform. The previous articles in this series are:

* [Comparing AWS CloudFormation and Terraform](https://medium.com/@kari.marttila/comparing-aws-cloudformation-and-terraform-c4251c88df96)
* [Comparing Simple AWS and Azure Infrastructure Demos](https://medium.com/@kari.marttila/comparing-simple-aws-and-azure-infrastructure-demos-cf756d1ef68b) (compared creating the same infra in AWS and Azure using Terraform)
In this current blog article I compare two IaC tools for creating infrastructure code for [Azure](https://azure.microsoft.com/): cloud native tool [ARM (Azure Resource Manager)](https://azure.microsoft.com/en-us/features/resource-manager/) and [Terraform](https://www.terraform.io/).

All intro demonstrations are available in [Tieto / Public Cloud Github account](https://github.com/tieto-pc). The demonstrations that are compared in this blog article are:

* **Terraform**: [azure-intro-demo](https://github.com/tieto-pc/azure-intro-demo)
* **ARM**: [azure-intro-arm-demo](https://github.com/tieto-pc/azure-intro-arm-demo)
### Terraform vs ARM

In this chapter I compare the two tools based on my experience when using them. I also give (very opinionated and personal) points (0–5, 5 highest) to tools regarding how I feel they excel in that category. At the end of the chapter I gather the results and give my personal recommendation which tool to use. The evaluation process is therefore pretty similar that I already used in my previous blog post: “[Comparing AWS CloudFormation and Terraform](https://medium.com/@kari.marttila/comparing-aws-cloudformation-and-terraform-c4251c88df96)” (and actually most of Terraform’s pros are pretty similar in these two articles).

Disclaimer: I have used a lot more Terraform than ARM so my opinions here are naturally pretty skewed.

#### 1. Cross Platform

Terraform’s major benefit is that you can use it with many cloud providers: you just need to learn one language (hcl) and workflow and use it with AWS, Azure, GCP etc. This is a real benefit for developers like me who do both AWS and Azure (and studying GCP at the moment) — one of the reasons why Terraform is my choice of IaC tool.

Points: Terraform: 5, ARM: 0.

#### 2. Language

I like Terraform’s [hcl language](https://github.com/hashicorp/hcl). Hcl’s interpolation syntax is very powerful and it lets you to reference variables, attributes etc. Combined with a good IDE plugin this makes infrastructure coding a breeze.

ARM uses JSON which as a notation language is a bit clumsy. You can put variables in a separate file and refer to the variables in the main configuration file. The ARM notation is relatively powerful, though — supporting conditionals, nested templates etc, you can read more about it in the [Microsoft ARM templates documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates).

Points: Terraform: 4, ARM: 3.

#### 3. Modularity

Modularity is a bit more natural using Terraform than using ARM. For a software developer background guy like me Terraform’s modules are a very natural way to modularize and reuse your infrastructure code.

You can modularize ARM templates using “[nested ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-linked-templates)” which are (relatively) flexible also — you can import nested templates from files, urls etc. — but the file needs to be accessible by Azure Resource Manager during deployment (which means you cannot have the nested template just locally — it has to be stored somewhere ARM can fetch it, e.g. in the Storage account).

Points: Terraform: 4, ARM: 2.

#### 4. Validation

Both tools provide a validation mechanism before you apply the infrastructure code, but a bit differently. Using ARM you can use command “az group deployment validate” to validate the syntax of the ARM template — if you want to validate the deployment you actually have to deploy the ARM template. Terraform provides a real cloud validation with the plan phase which actually checks the current deployment in the terraform state file, refreshes the status of the actual cloud configuration and calculates the delta between the current configuration and the target configuration (your Terraform infrastructure code).

Points: Terraform: 4, ARM: 2.

#### 5. Readability

Look at the family portrait screen shot at the beginning of this blog post. I think Terraform is a lot more readable than the bare JSON format of the ARM template. Terraform’s modularity also makes the code more readable. Some ARM template developers may not agree with me regarding the curly braces and dollar signs but once you know the syntax those make perfect sense for readability. Comments help readability — you can use comments in Terraform code but not in JSON.

Points: Terraform: 4, ARM: 2.

#### 6. Maintainability

Personally I think Terraform is more maintainable due to its better modularity and how you can create resources like functions. ARM templates are not that easy to read and also the fact that you cannot store nested templates in your local workstation is a bit clumsy.

Points: Terraform: 4, ARM: 2.

#### 7. Tooling

I really love [IntelliJ IDEA](https://www.jetbrains.com/idea/)’s [Terraform / HCL plugin](https://plugins.jetbrains.com/plugin/7808-hashicorp-terraform--hcl-language-support). Using the plugin with IntelliJ idea makes the infrastructure coding a breeze — your IDE is aware of all available parameters of the resource you are editing, IDE suggests other modules, hcl variables and locals in your code. I also installed the Azure plugin to IntelliJ but I had interesting instability issues with it — the first time I actually had to contact IntelliJ support. I therefore used [Visual Studio Code](https://code.visualstudio.com/) as an IDE when working with ARM templates, and installed [Azure Resource Manager Tools plugin](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools).

Points: Terraform: 4, ARM: 4.

#### 8. Azure Portal Support

Since ARM templating is a native Azure IaC tool it provides good support in the Azure Portal, of course. You can use the Azure Portal to monitor the progress of the deployment, see all deployment events, check various errors etc. For Terraform there is no Azure Portal support.

Points: Terraform: 0, ARM: 5.

#### 9. Workflow

I already covered the Terraform workflow quite well in my previous blog post so I’m not iterating it here. Let’s just say that Terrform’s workflow — “create all resources you can and stop if some service deployment fails” is a natural fit for a developer. ARM templates actually provide the same workflow and also are capable to rollback to the previous successful deployment (as CloudFormation) using the “-- rollback-on-error” option.

One minor advantage using Terraform is that you can create the main resource group you put your other resources also using Terraform. In ARM you must have created the main resource group beforehand since the azure cli command to deploy the ARM configuration requests you to supply the resource group that is used in the deployment.

Points: Terraform: 4, ARM: 4.

#### 10. Error Message Understandability

Most of the error messages Terraform spits are pretty obvious. But if something weird happens Terraform error messages can be rather difficult to interpret, especially for novices. ARM and using the Azure Portal on the other hand makes error messages more readable.

Points: Terraform: 3, ARM: 4.

#### 11. Infrastructure State Management

Both tools provide good state management. Terraform provides a [backend state mechanism](https://www.terraform.io/docs/backends/state.html). ARM is capable to rollback to a previous successful deployment (I interpret this to be a kind of state aware thing even though in some ARM documentation they said that ARM is “stateless”).

Points: Terraform: 3, ARM: 3.

#### 12. Infrastructure Changes Auditing Support

Since ARM is a native Azure IaC tool I would have assumed that it integrates with Azure Log Analytics (as CloudTrail in the AWS side) to let you see who did and what with ARM deployments. However I didn’t quickly find an exact match to audit the deployments in the Log Analytics the same way I can use CloudTrail to audit deployments in the AWS side — most probably just because I have used AWS much more than Azure.

Points: Terraform: 1, ARM: 1.

### Summary

Here are the results of my opinionated and skewed comparison. I use a factor (“Fa”: 1–2) to provide some emphasis how important this feature is for me as an infrastructure developer (a SysOps guy might choose different factors and different original points). So, the fields are: Fa = Factor (1–2), OP = Original points, FP = Factored Points. I see one major problem with this comparison and that is the fact that I haven’t used that much ARM — a seasoned ARM user most probably has found a standard way or at least some workaround for those issues I complained in this blog post. Maybe I update this blog post after some ARM project when I have gathered more ARM experience.

![](/img/2019-03-20-comparing-azure-arm-and-terraform_img_2.png)

So, both tools seem to be pretty good. I must emphasize that my personal points here for ARM are pretty skewed for the fact that I don’t have that much experience using ARM than using Terraform — so the reader should evaluate both tools and then make his/her own opinion.

Which tool to choose then? First ask your customer, of course. If your customer is using or requesting to use a specific IaC tool, then use that. Otherwise if you are a new cloud practitioner: learn both. At least in that way that you are fluent with one tool and know the basics of the other tool. If you do multi-cloud: learn Terraform.

One final observation. Microsoft openly advertises that you can use Terraform with Azure: [Terraform on Azure documentation](https://docs.microsoft.com/en-us/azure/terraform/). I couldn’t find the same support for Terraform in the AWS side — there were some mentions regarding Terraform in the [AWS blogs](https://aws.amazon.com/blogs/apn/terraform-beyond-the-basics-with-aws/), though.

### Conclusions

Both Terraform and ARM are pretty good when it comes to infrastructure coding for Azure. **The most important decision is to use an IaC tool **than which tool to use (I covered this decision a bit in my previous articles “[How to Create Infrastructure as Code for AWS and Azure](https://medium.com/@kari.marttila/how-to-create-infrastructure-as-code-for-aws-and-azure-ab0a5ddecc06)” and “[How to Create and Manage Resources in Amazon Web Services Infrastructure?](https://medium.com/tieto-developers/how-to-create-and-manage-resources-in-amazon-web-services-infrastructure-f9af85b77c4a)”).

Make an infrastructure project using both [ARM](https://azure.microsoft.com/en-us/features/resource-manager/) and [Terraform](https://www.terraform.io) (and why not evaluate also [Pulumi](https://www.pulumi.com/)) and make your decision based on your own priorities which tool made your life as an infrastructure coder more productive.

*The writer has two AWS certifications and one Azure certification and is working in *[*Tieto Corporation*](https://www.tieto.com/)* in Application Services / Public Cloud team designing and implementing cloud native solutions. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
  
