---
layout: post
title: Start Learning Cloud Skills!
category: [cloud]
tags: [cloud, aws, azure, gcp, iac]
date: 2021-06-24
---

![A screenshot of one of my personal cloud projects](/img/2020-04-10-cloud-infrastructure-golden-rules_img_1.png)

*A screenshot of one of my personal cloud projects.*

### Introduction

In my previous corporation an Azure guy once told me: _"It is not possible to be an expert in more than one cloud. If you want to be an expert in Azure you have to focus on Azure."_ A few days passed and an AWS guy told me: _"If you want to be an expert in AWS you have to focus on AWS."_

### Is It True?

Is it really true? If you want to be an expert in one cloud, do you really have to ditch other clouds away and focus solely on one cloud? No, it isn't true. You can happily be an expert in so many clouds you want. I have done IaC in Azure, GCP and AWS projects. There is no law that says you can't do so. I also have done certifications in all of these clouds (see in [www.karimarttila.fi/about](https://www.karimarttila.fi/about/) the status of my current cloud certs).

### The Argument

I guess those guys just were a bit troubled talking to a guy who does multi cloud work. It is very human to try to come up with an argument to rationalize your choices. Their choice was to stick with one cloud - and one of the arguments for that choice was that _"it is not possible to be an expert in more than one cloud."_

### On the Other Hand

On the other hand those guys were right, sort of. If you focus only on one cloud and dedicate all your learning resources to study that cloud - naturally you will be a better expert in that cloud compared to studying two or three clouds.

But this argument does not fly that far. I have two academic degrees, M.A. in psychology and M.Sc. in Software Engineering. It is perfectly possible to be interested in various fields and technologies and study them all. To study those academic degrees took me a lot more efforts than studying three clouds. The thing that those dedicated cloud guys didn't get was that I work as a software architect and programmer. I don't have to remember by heart every little detail of any specific cloud. It's entirely sufficient to understand the basic concepts well and the most important services. Every cloud provides VPCs, subnets, firewalls, VMs, container registries, Kubernetes as a service, databases etc. If you have learned one cloud it is easier to learn the same basic concepts and how to use them in new clouds, just like with programming languages. No one says that you cannot learn more than one programming language and therefore you have to stick with Javascript (if you don't believe me, read my story [Five Languages - Five Stories](https://www.karimarttila.fi/languages/2018/11/19/five-languages-five-stories.html)).

### Why to Study Cloud Technologies?

I'm old enough to remember the era of the managed services divisions in big IT corporations. At the beginning of the project the software architect had to calculate the needed capacity for various servers since when you sent the order to the managed services division and they started to order those physical servers it took weeks to have them at your data center with all middleware and firewalls installed and configured. In my first cloud project some years ago I was just astonished when I realized that I can create a whole virtual infrastructure with VPC, subnets, firewalls, frontend servers, backend servers, databases, queues, alarms etc using infrastructure code, and without any help from some managed services division. And that I could deploy a new copy of that environment in minutes, up and running, everything configured just as defined in the infrastructure code. Ever since that insight I never looked back. I decided that I will work only in cloud projects in which I can create the cloud infrastructure as code and the applications running in that cloud infra.

Public clouds with virtual infrastructures are the new paradigm. If you are an application developer I strongly recommend you to start learning cloud technologies as well.

### Do You Have to Certify?

No, you don't. You can learn everything you need to know about public clouds without ever going to one certification exam. The reason I like to take these certification tests is that the requirements for the basic cloud certifications provide really good list for the most important concepts and services you need to understand anyway for being a successful cloud expert in the cloud projects (and I'm talking here the "basic" developer/engineer/architect certs and not the specialty certs). And once you have learned those requirements why not to take the certification exam as well, put the cert in your LinkedIn profile and start getting calls from headhunters.

### How to Start Learning Cloud Skills?

Everyone has his or her own learning methods. Mostly just follow your common sense and start learning the basics whatever methods suites you. There are a lot of material on the Internet. I have liked a lot about the courses in [Pluralsight](https://www.pluralsight.com/) and in [Coursera](https://www.coursera.org/) and do the [Qwiklabs](https://www.qwiklabs.com/) or other labs related to those courses. I have also attended some traditional courses related to some specifics like Security on AWS.

If you know nothing of public clouds, just choose one of the "Big Three" - i.e. AWS, Azure, or GCP. It is not that important which one you choose - they are all good public clouds. Once you master the basics in that cloud it is easier to learn the basics regarding the other two clouds as well.

Your next step is to get **a cloud account** - you cannot learn cloud skills just by reading books - you absolutely need a cloud account. All Big Three clouds provide "free tier services" or a certain amount of free credits if you order a personal account (at least they used to do that when I learned the basics - nowadays I mostly use my company's cloud accounts for my personal projects as well and let the capitalist pay my learning bills).

I worked in my previous corporation as a Cloud Mentor. One of my duties was to teach cloud skills to our developers. I always emphasized that there are three things you need to learn:

1. **Basic information regarding the cloud and its basic services** (e.g. virtual network, basic storage options, basic computing options).
2. How to use the **cloud provider's portal** efficiently.
3. How to create **infrastructure as code**.

You can learn the basic concepts and services, and how to use the cloud provider's portal following various video courses eg. in [Pluralsight](https://www.pluralsight.com/) or [Coursera](https://www.coursera.org/), and do the labs in dedicated sites (like [Qwiklabs](https://www.qwiklabs.com/)) or using your personal cloud account (or do like I do: use your company's cloud account (with permission, of course) and let the capitalist pay your learning bills). But what those courses don't teach that much is how to create infrastructure as code (IaC) using eg. Terraform or Pulumi or cloud provider's native IaC tool like CloudFormation, ARM, or Deployment Manager. It is utterly important to learn how to create infrastructure as code since in real projects that is how the cloud solution is done professionally (and not by clicking resources with a mouse in the cloud portal).

So, what are you waiting for? Start learning cloud skills! (And once you think you are an expert, read: [Cloud Infrastructure Golden Rules](https://www.karimarttila.fi/iac/2020/04/10/cloud-infrastructure-golden-rules.html) - my personal manifest how to create cloud infrastructures.)


*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a cloud or Clojure project in Finland or you are interested to get cloud or Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
