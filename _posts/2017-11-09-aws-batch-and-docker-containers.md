---
layout: post
title: "AWS Batch and Docker Containers"
category: [aws]
tags: [aws, cloud, iac, docker, container, batch, cloudformation]
date: 2017-11-09
---

![](/img/2017-11-09-aws-batch-and-docker-containers_img_1.png)

*AWS Batch Ecosystem.*

### Introduction

There are two very typical use cases in enterprise software: [API](https://en.wikipedia.org/wiki/Application_programming_interface) and [Batch processing](https://en.wikipedia.org/wiki/Batch_processing). For API work AWS provides a lot of useful services: you can bake your API into an [Amazon Machine Image](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) (AMI), [Docker](https://www.docker.com/) container using [EC2 Container Service](https://aws.amazon.com/ecs/) (ECS), utilize [Auto scaling](https://aws.amazon.com/autoscaling/) for elasticity, possibly use [API Gateway](https://aws.amazon.com/api-gateway/) to publish your API (not necessary even some junior developers assume that you have to publish your APIs using AWS API Gateway) etc.

For batch processing you usually don’t have such strict latency requirements as in API processing, error handling can be more relaxed etc. Typical batch processing tasks are e.g. to do some nightly analysis, do reports and send them to some integrations etc.

### AWS Batch

If you need to do some batch processing AWS provides an excellent framework for it: [AWS Batch](https://aws.amazon.com/batch/). Using AWS Batch you don’t have to reinvent the batch processing framework wheel, you get all necessary entities nicely glued together: Computing environment, job definitions, job queues and batch orchestration. Your batches are elastic: AWS Batch takes care of the needed capacity to run your batch jobs efficiently. Let’s take a closer developer’s point of view at the AWS Batch.

The original AWS Batch archicture and CloudFormation setup described in this blog article was done by my brother in arms in AWS battles, distingueshed AWS Guru [Timo Tapanainen](https://www.linkedin.com/in/timo-tapanainen/), so let’s give him most of the credit regarding the information we are sharing here.

### The Batch Application

You need some business logic to run in your batch processing, of course. Maybe this business process needs to fetch some data from your AWS Big data database, e.g. [Redshift](https://aws.amazon.com/redshift). Then you need to do some processing regarding the data, which may involve more or less complex business rules. Once you are ready you create a report and store it to somewhere (e.g. [S3](https://aws.amazon.com/s3)) or send it via integration to some integration partner.

Ok. You are ready with your application. You have tested application logic with your local tests and with integration tests in AWS. What next? You need to create a deployment unit, that is to bake your application e.g. either into an Amazon Machine Image to be deployed to EC2, or bake the application into a Docker container to be run in AWS ECS. Batch processing typically is stateless activity — you fetch data, create report, store report, end of story (at least for that night). This is a perfect use case for a Docker container: Docker containers can be booted in milliseconds, they do their job and once the job is over you stop or terminate the container. Using AWS Batch with Docker containers is a perfect mach. If you have to create e.g. 10.000 reports with different parameters every night, create one AWS Batch Job definition, and trigger it every night e.g. using Lambda to create 10.000 Batch jobs with different parameters in each. You can configure AWS Batch to run these jobs in parallel — this way you can process a lot of even long lasting batch jobs during a short period of time.

We have already covered using Dockers in AWS context in another blog post ([How to Create Docker Containers in AWS?](https://medium.com/tieto-developers/how-to-create-docker-containers-in-aws-3134daec423c)), but let’s iterate here the process from the AWS Batch point of view.

Creating a docker image is rather staightforward. You create a Dockerfile in which you tell what kind of stuff you need in your application, configure entrypoint to application etc. Let’s skip that part and show how to ask your Continuous server to build the docker image after every successful build:

![](/img/2017-11-09-aws-batch-and-docker-containers_img_2.png)

Docker build.Ok. Now you have your docker image. Next do some testing with the image, start container, run tests with it etc.

Next stop is to deploy the Docker image to [AWS ECR](https://aws.amazon.com/ecr/) to be used by various other AWS Services (like in AWS Batch Computing Environment we are going to see soon):

![](/img/2017-11-09-aws-batch-and-docker-containers_img_3.png)

Docker deploy to AWS ECS.All right! You are ready to rock and roll with your Docker image in AWS!

### The AWS Batch CloudFormation Stack

When we started to use AWS Batch Terraform didn’t provide full support for AWS Batch. Therefore we created the AWS Batch environment using [AWS CloudFormation](https://aws.amazon.com/cloudformation) (nowadays there is full support by Terraform, so you can use either tool). The whole CloudFormation stack description is easily some 200 lines — you have to glue every single bit and piece together, but don’t get intimidated by it — you soon learn to see Terraform / CloudFormation code visually as AWS resources and how they are co-operating. An example of the CloudFormation file which introduces the most important players in the field:

![](/img/2017-11-09-aws-batch-and-docker-containers_img_4.png)

AWS CloudFormation snippet for AWS Batch.So, the main players are:

* **The Docker repository** (AWS ECR). This is the service that hosts your business logic baked as a Docker image. AWS Batch Computing environment needs to be glued to the repository so that it knows where to get the computing resources for the task.
* **Some IAM Roles and Policies**. You need to define for all computing resources the policies so that they are able to utilize other AWS services and your AWS entities hosted by those services to do the actual job (e.g. to store the final reports to S3).
* **Job Queue**. This is the queue you send your jobs to be processed. AWS Batch automatically assigns needed resources for the job once it is delivered to the Job queue.
* **Job Definition**. Here you actually tell AWS Batch Computing environment the Docker image you want to use as a computing resource, what environment variables you want to inject to the Docker container once it is up etc.
Deploy stack. If everything went well you should have a fully operating Batch processing environment in AWS.

### Run It!

Everything is ready for your first AWS Batch test! Open AWS Console, go to AWS Batch view, then Job definitions — you should see your Job definition here. Select your Job definition, click Actions / Submit job. This is a testing stage in which you can manually test your AWS Batch logic. You can define various parameters here, e.g. environment variable values. Then hit Submit job button, go to AWS Batch Dashboard view and see how your batch job goes through all batch lifecycle steps (Submitted, Runnable, Starting, Running, Succeeded). Finally you can open CloudWatch Logs and see that everything went well (if you have test trace).

Once you have done all testing, you can add the triggering mechanism in CloudFormation to start the batch job e.g. nightly. For the triggering mechanism you can use e.g. [AWS Lambda](http://docs.aws.amazon.com/lambda/latest/dg/with-scheduled-events.html).

Also configure your Continuous integration server (using the scripts given previously) automatically to build a new Docker image after every successful build and deploy image automatically to AWS ECR, so after every Git commit your CI pipeline creates a new Docker image in AWS ECR ready to be used in your AWS Batch Job definition (more about that part, see: [Jenkins on AWS](https://medium.com/tieto-developers/jenkins-on-aws-49133e011ac5)).

### Conclusions

AWS provides an excellent batch processing framework which beautifully integrates to all other AWS services — no need to create an in-house solution for your batch processing. This idea nicely fits to the paradigm “[Use AWS Services as Building Blocks to Implement Your Enterprise System](https://medium.com/tieto-developers/use-aws-services-as-building-blocks-to-implement-your-enterprise-system-598676a0ee49)”.

Both writers are AWS Certified Solutions Architects Associate, architecting and implementing AWS projects in Tieto CEM Finland. If you are interested about starting a new AWS project in Finland, you can contact us with firstname.lastname at tieto.com.

Kari Marttila & Timo Tapanainen

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
* Timo Tapanainen’s Home Page in LinkedIn: <https://www.linkedin.com/in/timo-tapanainen/>
  
