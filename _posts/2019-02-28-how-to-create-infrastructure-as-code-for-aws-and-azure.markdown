---
layout:	post
title:	"How to Create Infrastructure as Code for AWS and Azure"
date:	2019-02-28
---

  ### Introduction

In this blog article I compare various ways and tools to create infrastructure as code for AWS and Azure. I try to give some instructions for new cloud learners regarding how to create cloud infra and typical pitfalls to avoid.

### How to Create Cloud Infra?

In most cases you should create cloud infra using some infrastructure as code (IaC) tool. I have covered various ways to create cloud infra (using cloud provider’s **portal**, **cli** and **IaC tool**) in my earlier blog post “[How to Create and Manage Resources in Amazon Web Services Infrastructure?](https://medium.com/tieto-developers/how-to-create-and-manage-resources-in-amazon-web-services-infrastructure-f9af85b77c4a)” — the instructions of that article to use IaC tool applies to Azure or any cloud provider as well.

### What Infrastructure as Code Tool to Use?

Both AWS and Azure provide a native tool for creating infrastructure code:

* AWS: [CloudFormation](https://aws.amazon.com/cloudformation/)
* Azure: [Azure Resource Manager — ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/)
There are also some widely used generic cloud infrastructure tools that you can use with most big cloud providers, e.g.:

* [Terraform](https://www.terraform.io/)
* [Pulumi](https://pulumi.io/)
If there are so many choices — which infrastructure as code tool to choose? My recommendation is: Learn to use the cloud provider’s native tool (in AWS: CloudFormation, and in Azure: ARM), and also learn to use one generic tool. My choice for a generic tool is Terraform.

So, I have used CloudFormation, ARM and Terraform and they all have their strengths and weaknesses. Let’s briefly introduce the tools.

### CloudFormation

![](/img/1*M_AXnXK2hRrf9ClOeO1BCw.png)CloudFormation code using yaml.[CloudFormation](https://aws.amazon.com/cloudformation/) uses a concept of a stack. You create CloudFormation stacks and deploy the stacks to AWS. AWS nicely shows the CloudFormation deployment in AWS Console and you can also examine if some part of the deployment failed. The main strength (or weakness — point of view) with CloudFormation is that you can use it as a transactional infrastructure tool — if some part of your deployment fails CloudFormation knows how to roll back the changes — i.e. you deploy everything or nothing regarding the stack. This is a nice feature if it is what you want. It may sound that it is what you usually want (e.g. comparing to ACID in the database world) but in the cloud environment that is not necessarily what you want when developing a new cloud infrastructure (more about that when introducing Terraform).

AWS provides various [quickstart CloudFormation templates](https://aws.amazon.com/cloudformation/aws-cloudformation-templates/) that you can use when starting your own CloudFormation project.

### Azure Resource Manager — ARM

![](/img/1*XFowWm_iWbUUwuX7EWcn2g.png)ARM code using yaml.[Azure Resource Manager — ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/) uses json format. ARM files tend to be quite verbose and therefore long. That is the main disadvantage regarding ARM templates and I guess one of the reasons why many developers want to create Azure infra using shell scripts and [azure command line interface](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest). I would’t go that road, however. Either stick with ARM or choose some other infrastructure as code tool like Terraform.

Microsoft provides various [quickstart ARM templates](https://azure.microsoft.com/en-us/resources/templates/) that you can use when starting your own ARM project.

### Terraform

![](/img/1*eX9U0mdsWTqkS5GsSnSwpA.png)Terraform code using hcl.The biggest strength of [Terraform](https://www.terraform.io/) is that you learn one tool and you can use the tool with any big cloud provider. That is pretty strong argument especially for a developer like me who does both AWS and Azure. Terraform code also is pretty clear and the hcl language is concise and readable — you quite soon realize what a certain Terraform module does when you look e.g. how the variables are used to create certain cloud resource (like the EC2 instance in the example above). When you deploy your Terraform infra code to some cloud provider Terraform creates the resources that it can and stops when some particular resource fails — this usually is the case what you want in development compared to CloudFormation which rolls back everything. I.e. you just fix that part of the deployment code and re-deploy and Terraform happily continues where it left the infra in the previous round.

You can find many terraform code templates in Github (use google query: site:github.com terraform templates).

### When to Use Portal or CLI?

So, I believe you as a reader now realize that you should use some infrastructure as code tool when creating cloud infra and not create the infra using the cloud provider’s portal or cli. However, there are cases when I use the portal or cli on purpose. When learning some new cloud service I usually first create a test instance using portal. Then I create the same entity using infrastructure as code tool. If I have any issues with the infra code I can check the working example created by the portal wizard and compare the entities.

Another use case for using the portal to create some infrastructure is a simple static infra that you create once and you don’t have to modify and maintain the infra after that, and automating the infra would cost considerably more than the benefits you get from automation. However, personally I would use IaC tool even for those purposes since if you are fluent with the tool it doesn’t take that much time. There are exceptions for this rule (as any), though. E.g. if you have to create a very complex VM with lots of proprietary software installations and configurations and you need the infra created only once — automating the VM image for one deployment wouldn’t be viable (but even in that case I would create the infra around the VM (virtual network, subnets, firewalls etc) using IaC tool).

The portal is tempting in that sense that it is so easy to create cloud infra using it. But do not start to go that path — the beginning is easy but soon you get sucked into a dark swamp when your cloud infra grows and you have to maintain everything manually using the portal, you have to document all your resources in fine detail (because they were done manually) and you have no means to automate anything. Remember the Star Wars proverb: “*The dark side of the force is tempting*”. Using an infrastructure as code tool the beginning is more difficult and time consuming. But when your infra grows you will start to get the rewards: automation and the infra and all alarms and monitoring documented in infrastructure code to the finest detail.

### Conclusions

Learn to use the cloud provider’s portal so that you can effectively navigate and search your resources. Use the cloud provider’s portal and cli for creating resources for ad hoc purposes (e.g. experimenting with a new service) but create all production infrastructure with a real infrastructure as code tool. My recommendation is to learn the cloud provider’s cloud native tool (AWS: CloudFormation, Azure: ARM) and one generic tool (like Terraform).

*The writer has two AWS certifications and one Azure certification and is working in *[*Tieto Corporation*](https://www.tieto.com/)* in Application Services / Public Cloud team designing and implementing cloud native projects. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

**Kari Marttila**

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
  