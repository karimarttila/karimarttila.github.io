---
layout:	post
title:	"AWS Services as Building Blocks to Your Enterprise System"
category: [aws]
tags: [aws, iac, cloud]
date:	2017-01-27
---

  We are going to share some experiences regarding how to build enterprise-level applications using the AWS platform. In this blog post we are talking about how to use AWS services as building blocks…   ![](/img/2017-01-27-aws-services-as-building-blocks-to-your-enterprise-system_img_1.jpeg)The real Amazon ecosystem. [© Big Blue Marble](https://www.flickr.com/photos/bbmexplorer/)We are going to share some experiences regarding how to build enterprise-level applications using the AWS platform. In this blog post we are talking about how to use AWS services as building blocks to implement your enterprise system.

The diagram below depicts some of the typical AWS services that you are going to use when implementing your enterprise system in AWS infrastructure. We are going to introduce these services in this article.

![](/img/2017-01-27-aws-services-as-building-blocks-to-your-enterprise-system_img_2.jpeg)A concept diagram of AWS services used in an enterprise system

### 1. Virtual Private Cloud

The first task implementing [AWS](https://aws.amazon.com/) infrastructure is to create a [VPC](https://aws.amazon.com/vpc/). A best practice is to create at least two AWS accounts, one for production and another for any other purposes (development, test etc). This way you completely separate production environment from other activities and have e.g. different [IAM](https://aws.amazon.com/iam/) users for production environment.

You can create VPC using [AWS CloudFormation](https://aws.amazon.com/cloudformation/) or e.g. [Terraform](https://www.terraform.io). We recommend to choose either tool and then create and manage all AWS resources consistently with that tool. We have used Terraform to create all AWS resources. We'll write later another post to describe in more detail how to utilize Terraform to manage AWS resources.

AWS CloudFormation provides many ready to use [templates](https://aws.amazon.com/cloudformation/aws-cloudformation-templates/) for creating VPCs for different purposes. Even if you used Terraform you can browse and create exploratory VPCs with CloudFormation templates to see what kind of configuration certain templates create.

Most important entities with VPC are: IGW (Internet Gateway), NAT, public and private subnets at least in two different AZs (AWS Availability Zones). We recommend you to read more about these entities in AWS documentation and tutorials. You create subnets for each environment you need (in production account for production, in development account for each environment you need: development, customer testing, partner testing, QA, performance testing…). Using AWS it is pretty easy to create and decommission entities. Therefore you can and should create as many environments you need — the best practice is: one environment for one purpose. With CloudFormation / Terraform you can parameterize your environments and make them basically exact copies of each other. Read the next article "[How to Create and Manage Resources in Amazon Web Services Infrastructure?](https://medium.com/@kari.marttila/how-to-create-and-manage-resources-in-amazon-web-services-infrastructure-f9af85b77c4a#.gt6f4tkrj)" for more details.

### 2. Virtual Servers, Auto-scaling and Firewalls

The next step is to start creating virtual servers to your new VPCs. Create a CloudFormation / Terraform configuration so that you create the same entities and services the same way in all of your environments (to make the environments as copies of each other).

Create an AWS Launch configuration and AWS Auto-scaling group for each of your application. You can create golden images or dedicated Docker images per application — do not install more than one application per image. At least in Terraform you can create modules in which you can encapsulate the creation of many AWS resources per one entity (e.g. for application node, you can inject parameters to a module which creates Launch configuration, Auto-scaling group etc.).

You can create a dedicated bastion server for each environment. Configure your [AWS Security Groups](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html).

### 3. Databases as Services

AWS provides Relational Database as a Service - [RDS](https://aws.amazon.com/rds/). AWS also provides NoSQL database as a service — [DynamoDB](https://aws.amazon.com/dynamodb/). You should create these resources with CloudFormation / Terraform as all AWS resources. E.g. using Terraform you can create a RDS module into which you inject the instance size and HA as parameters — smaller instance with no HA in development to minimize costs, bigger instance and HA in performance and production environments). You don't have to do maintenance operations to databases — maintenance is provided as a service by AWS.

### 4. Queues and Notifications as Services

You can use AWS SQS — [Simple Queue Service](https://aws.amazon.com/sqs/) as a service without any installation hassle. You just configure what queues you need for your application and configure which roles can do what with these queues. We have used AWS queues e.g. storing certain units for further multi-threading processing — one process parses big XML file and distributes units of work to an AWS queue, then many threads are requesting those units from the queue and processes them in parallel for performance reasons. Another example to use queues is to use a queue as a triggering mechanism. An application listens to a queue and if there is a message, it parses the message and starts processing. In development environment you can send messages to the queue manually using AWS [CLI](https://aws.amazon.com/cli/), in production you configure an AWS [CloudWatch](https://aws.amazon.com/cloudwatch/) [schedule rule](http://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) to send the message to the queue.

You can use AWS SNS — [Simple Notification Service](https://aws.amazon.com/sns/) for gluing various AWS and your custom alarms to various notifications (email, sms…). We have used SNS extensively for monitoring certain critical alarms (logs, RDS queues, EC2 cpu, S3 file uploads…).

### 5. Storage as a Service

AWS provides excellent long term storage options. One of the most used storage services is AWS S3 — [Simple Storage Service](https://aws.amazon.com/s3/). You can use S3 e.g. for various file base integrations. You can publish certain S3 buckets to your stakeholders who then can upload asynchronously files to those buckets on their own pace. You can add various triggers to S3 buckets so, that e.g. when a new file appears in a bucket, it automatically triggers an AWS [Lambda](https://aws.amazon.com/lambda) which then starts to process the file (and possibly stores it to DynamoDB, RDS, or notifies some application asynchronously using AWS queues).

### 6. Logging and Monitoring as Services

AWS provides excellent logging tools. You can configure aws logging package to your golden images and send application logs automatically to AWS CloudWatch [Logs](https://aws.amazon.com/about-aws/whats-new/2014/07/10/introducing-amazon-cloudwatch-logs/). You can categorize your log streams to various log groups (e.g. you could have different log groups for your different environments). You can set various AWS alarms to monitor log files (and ask alarms to send notifications to SNS… and you get email/sms regarding errors in the logs).

### 7. The Rest of AWS Services

There are dozens of various new AWS services, go and check the latest situation in [here](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/). You can use all these services as building blocks when implementing your enterprise system.

### Conclusion

Amazon Web Services is so much more than just a platform for virtual servers. AWS is a rich ecosystem providing dozens of services that you can use as building blocks when you implement your enterprise system. Your imagination is the limit how to combine those services into a working enterprise system. There are good AWS [Reference Architectures](https://aws.amazon.com/architecture/) - browse down to the chapter "AWS Reference Architectures" and open the PDF files: for each reference architecture there is a descriptive picture (with AWS services used) and a good explanation how the AWS services are used together.

We list here all our AWS related articles written this far:

* [**DevOps Success Factors**](https://medium.com/tieto-developers/devops-success-factors-53beafe63942#). A description of DevOps Success Factors / AWS Best Practices we have used in our previous AWS project.
* [**How to Create and Manage Resources in Amazon Web Services Infrastructure?**](https://medium.com/tieto-developers/how-to-create-and-manage-resources-in-amazon-web-services-infrastructure-f9af85b77c4a#) A description how to create AWS resources. Do not create them using Ad hoc process with AWS Console or AWS CLI — use Code as Infrastructure tools.
* [**How to Create EC2 Images in AWS?**](https://medium.com/tieto-developers/how-to-create-ec2-images-in-aws-a27b1afc97c6#) You can bake EC2 images in many ways — read our preference.
* [**How to Create Lambdas in AWS?**](https://medium.com/@kari.marttila/how-to-create-lambdas-in-aws-8f04ac833b2e#) AWS Lambdas are an excellent way to create serverless functionalities.
Both writers are AWS Certified Solutions Architects Associate, architecting and implementing AWS projects in Tieto CEM Finland. If you are interested about starting a new AWS project in Finland, you can contact us with firstname.lastname at tieto.com.

Kari Marttila & Timo Tapanainen

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
* Timo Tapanainen's Home Page in LinkedIn: <https://www.linkedin.com/in/timo-tapanainen/>
  
