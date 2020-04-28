---
layout:	post
title:	"Setting Up AWS DevOps Tools"
categories: [blog, aws]
tags: [aws]
date:	2019-04-12
---

### Introduction

![](/img/1*0oCftRVnxpjoHA-kFabeOA.png)AWS DevOps Demonstration TopologyIn my new unit working as a Cloud Mentor I was thinking myself that sooner or later some AWS project asks me to help them to set up AWS DevOps tools for the project. So, I created a short AWS DevOps demonstration as a learning project and also to be used as a template and example for setting up these tools in a new AWS project. The diagram above depicts the components I used in the project and which I also describe in more detail in this blog post.

The demonstration was implemented using [Terraform](https://www.terraform.io/) as an infrastructure as code tool, and the source code is available in [Tieto / Public Cloud Github account](https://github.com/tieto-pc/aws-devops-intro-demo).

### The AWS DevOps Players

The main players of the AWS DevOps game are:

* [**CodeCommit**](https://aws.amazon.com/codecommit/) — an AWS hosted Git repository service.
* [**CodePipeline**](https://aws.amazon.com/codepipeline/) — an AWS hosted continuous delivery orchestrator.
* [**CodeBuild**](https://aws.amazon.com/codebuild/) — an AWS hosted continuous integration service.
I’m not going describe the services more than that — you can read more about the services in AWS documentation. Let’s next talk about how I used these services to build a demonstration continuous delivery pipeline.

### The DevOps Game Phases

Take a look at the diagram in the beginning of this blog article. I’ll describe the DevOps Game Phases of the demonstration next.

1. The process starts with a new commit into the **CodeCommit** repository.
2. The new commit triggers a **CloudWatch Event rule**.
3. The CloudWatch Event Rule starts a **CodePipeline project**.
4. The CodePipeline project has various **stages** (see in more detail in [codepipeline.tf](https://github.com/tieto-pc/aws-devops-intro-demo/blob/master/terraform/modules/codepipeline/codepipeline.tf) file). The **first stage** pulls the newest source code from the CodeCommit repository.
5. The **second stage** has two **actions**. The **first action** calls the **CodeBuild project 1** (codebuild\_build\_and\_test\_project) to build the application and run the unit tests. You can see the CodeBuild project in file [codebuild.tf](https://github.com/tieto-pc/aws-devops-intro-demo/blob/master/terraform/modules/codebuild/codebuild.tf). The build specification of this CodeBuild project is in file [buildspec\_build\_and\_test.yml](https://github.com/tieto-pc/java-simple-rest-demo-app/blob/master/codebuild/buildspec_build_and_test.yml "buildspec_build_and_test.yml").
6. The **second action** uploads the application jar file (that the first action built) into an **S3 bucket**.
7. The **third stage** finally calls the **CodeBuild project 2** (codebuild\_build\_docker\_image\_project) to build the Docker image. The CodeBuild project first pulls the application jar from S3 bucket and then bakes the jar into a new Docker image. Finally the project pushes the image to **ECR**. You can see the CodeBuild project in file [codebuild.tf](https://github.com/tieto-pc/aws-devops-intro-demo/blob/master/terraform/modules/codebuild/codebuild.tf). The build specification of this CodeBuild project is in file [buildspec\_build\_docker\_image.yml](https://github.com/tieto-pc/java-simple-rest-demo-app/blob/master/codebuild/buildspec_build_docker_image.yml "buildspec_build_docker_image.yml").
We can use our imagination what could happen next. Example. The upload of the new Docker image could trigger another CloudWatch Event Rule which then triggers another CodePipeline project — this project could deploy the new Docker image to some ECS or EKS environment and e.g. start a Robot framework project to end-to-end test the new Docker image in that AWS environment. Your imagination is the limit what you can automate using AWS.

### How to Develop Cloud Infra Using New Services?

It might be a bit intimidating for a junior cloud developer to start building a complex cloud system using infrastructure as code — and if you are new to most of the cloud services you are going to use it is even more intimidating. You might start building something and have no idea how to assemble everything together or even which pieces you are going to need. So, how to start?

It is usually a wise move to create the cloud entities first manually using the portal, then examine how the cloud provider’s wizards (behind the curtains) created the entities using the services. This way you have a working reference model — cloud infra that you know that works and you can examine what pieces the portal created and how they are glued together.

Ok. Now you have a working reference model of your cloud infra, created using the portal. But do not make the mistake that you are going to use this infra in production. You created the infra using portal — there is no way you can reproduce the infra automatically on demand. So, you need to re-create this infra using an IaC tool (e.g. Terraform or CloudFormation).

There are a couple of ways to examine the manually created entities and figure out how to re-create them using IaC. One way is to examine the entities in the portal and then create the equivalent entities in infrastructure code. Another way is to get the structural description of the entities e.g. using the cloud provider’s CLI (command line interface):

AWS\_PROFILE=YOUR-AWS-PROFILE aws codepipeline get-pipeline --name YOUR-MANUAL-CODEPIPELINE-PROJECT-NAME > manual-codepipeline-description.txt  
AWS\_PROFILE=YOUR-AWS-PROFILE aws codebuild batch-get-projects --name YOUR-MANUAL-CODEBUILD-PROJECT-NAME --output json > manual-codebuild-description.txtThen you can examine the contents of the files and re-create the same entities using an IaC tool.

Of course this was a happy day scenario. In real life there are always bits and pieces missing. The portal wizards create all kinds of stuff behind the scenes that you have to figure out yourself. One example. When I had the Terraform modules of CodePipeLine project and CodeBuild projects ready and I was testing the pipeline I was wondering why the CodePipeline that I created using the portal wizard (the reference model) got automatically triggered but my CodePipeline that I created using Terraform IaC was not. Consult the book of knowledge — Google — and I got the answer in an AWS document — [Start a Pipeline Execution in CodePipeline](https://docs.aws.amazon.com/codepipeline/latest/userguide/pipelines-about-starting.html):


> *”*When you use the console to create a pipeline that has a CodeCommit source repository or Amazon S3 source bucket, CodePipeline creates an Amazon CloudWatch Events rule that starts your pipeline when the source changes.*”*So, the portal wizards create all kinds of service roles, policies and triggering mechanisms to make your life easier when you are creating the service entities using the portal. Portal makes things often so easy that some cloud developers create the whole cloud native system using the portal — big mistake. I once audited a customer’s big data system that was created using the portal — no documentation how the system was created, no way to reproduce the equivalent system for development or testing automatically. The only way to make a reproducible cloud system is to use infrastructure as code.

So, the lesson of the story is: Use the portal to explore and learn new cloud services, but create the final system using infrastructure as code.

### Conclusion

AWS DevOps tools (CodeCommit, CodePipeline and CodeBuild) are powerful tools for building a DevOps pipeline in AWS that integrates excellently to other AWS services — no need to use Github or Jenkins anymore. The only downside of these tools is that they are a bit complex to set up for a junior cloud developer. A cloud development best practice helps in this road: first build a reference model using cloud provider’s portal wizards, examine the entities created by the wizards and create the same entities using infrastructure as code.

*The writer has three AWS certifications and one Azure certification and is working in *[*Tieto Corporation*](https://www.tieto.com/)* in Application Services / Public Cloud team as a Cloud Mentor designing and implementing cloud native solutions. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
  