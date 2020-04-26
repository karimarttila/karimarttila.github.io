---
layout:	post
title:	"How to Create Lambdas in AWS?"
categories: [blog, aws]
tags: [aws]
date:	2017-03-17
---

  We continue our AWS related blog series that we started in our first article “[Use AWS Services as Building Blocks to Implement Your Enterprise System](https://medium.com/tieto-developers/use-aws-services-as-building-blocks-to-implement-your-enterprise-system-598676a0ee49#)”. In this new article we talk about our experiences regarding how to create Lambdas in AWS.

![](/img/1*klfgoXd3AqjEj-c6-cTPug.jpeg)Concept AWS Architecture using AWS Lambdas.### Introduction

In our previous blog post “[How to Create EC2 Images in AWS?](https://medium.com/tieto-developers/how-to-create-ec2-images-in-aws-a27b1afc97c6#)” we already explained that there are several ways to create services to AWS infrastructure: EC2 instances (servers), Docker containers and Lambdas.

This blog post is a short introduction how to create Lambdas for Amazon Web Services.

### **What Are AWS Lambdas?**

[AWS Lambdas](https://aws.amazon.com/lambda/) are serverless functions that AWS hosts without the need to create dedicated servers (EC2) for them. This is a new cloud paradigm that is most probably going to be big in the future. You don’t need to create and pay for EC2 instances if you know that you are going to run your functionality only in certain occasions — you only pay for the milliseconds you run the functions.

### **How to Create AWS Lambdas?**

You can create AWS Lambdas using the AWS Console or CLI — but you should do that only when experimenting with AWS resources — never use that kind of ad hoc process to create / maintain your production infrastructure (for a more detailed reason read our previous blog article “[How to Create and Manage Resources in Amazon Web Services Infrastructure?](https://medium.com/tieto-developers/how-to-create-and-manage-resources-in-amazon-web-services-infrastructure-f9af85b77c4a#)”).

We have created Lambdas using our favorite Infrastructure as Code tool — [Terraform](https://www.terraform.io/).

### **For What Purposes Can You Use AWS Lambdas?**

As always with AWS Services — your imagination is the limit for what purposes you can use AWS services and how to combine them to other AWS services. All AWS services integrate to each other extremely well and AWS Lambdas are no exception. In the picture of the beginning of the article we have depicted some example Lambda use cases:

* **Validation**. Do XML/JSON/Domain/Whatever validation regarding the REST integrations you provide for your partners / on-premise data centers.
* **S3 Triggering**. Trigger some file manipulation in S3. This is pretty idiomatic way to combine AWS Lambdas with S3 based file integrations. You create a S3 bucket and publish it to your integration partner. Your integration partner can upload files to this S3 bucket in its own pace. When a new file is uploaded to the bucket S3 will automatically trigger a Lambda function that is associated to this bucket. File processing can be anything from parsing XML, unzipping file content, moving file to somewhere, pushing it to Kinesis stream etc.
* **Kinesis**. Connect Lambda to Kinesis streams. Kinesis streams can stream terabytes of data. You publish your kinesis stream to your partners who are streaming data to the stream. You associate a Lambda function to read the Kinesis stream — Lambda gets triggered for all new events in the stream. The Terraform code in the picture below depicts this kind of Terraform configuration.
* **Store to database**. You can use Lambdas to read stream and store the data to some AWS data store like DynamoDB.
* **REST API**. You can combine API Gateway with Lambda to provide REST API for your clients.
![](/img/1*yHL78RdzL0_vO7JnO-nF6Q.png)Example Terraform configuration to create Lambda and connect it to Kinesis stream.The example Terraform code shows how to create generic AWS Lambda module and inject all environment related parameters to it — you can use it to create N number of similar Lambda instances to all of your AWS environments (dev, qa, perf, prod…). The latter part of the code shows how to connect the Lambda to be triggered by new Kinesis events.

### **How to Implement AWS Lambdas?**

You can implement Lambdas using your favorite programming language. Currently supported languages are given in [AWS Lambda Programming model Documentation](http://docs.aws.amazon.com/lambda/latest/dg/programming-model-v2.html):

* Java
* C#
* Python
* Node.js
### Conclusion

There are many ways to create application services to AWS infrastructure. When you have short-lived functionalities for one focused task you should use AWS Lambdas.

Both writers are AWS Certified Solution Architects Associate, architecting and implementing AWS projects in Tieto CEM Finland. If you are interested about starting a new AWS project in Finland, you can contact us with firstname.lastname at tieto.com.

Kari Marttila & Timo Tapanainen

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
* Timo Tapanainen’s Home Page in LinkedIn: <https://www.linkedin.com/in/timo-tapanainen/>
  