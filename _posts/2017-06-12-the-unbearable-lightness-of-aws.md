---
layout: post
title: "The Unbearable Lightness of AWS"
category: [aws]
tags: [aws, cloud, iac, devops, productivity]
date: 2017-06-12
---

### Introduction

I've been using AWS for some time now, done one [successful project](https://medium.com/tieto-developers/devops-success-factors-53beafe63942) with production deployments with AWS and currently doing a big data project with AWS. And I just love AWS. Why? Let's list here some reasons why architect feels his life is full of unbearable lightness of AWS.

### No Dependencies

I occasionally mentor junior architects in our corporation. Young lions are always so eager to get recommendations of the coolest bleeding edge technologies. But my first lesson to them is this: "Beware of all dependencies, of any kind". If you have a dependency in your project or system you will always be — dependent of it. If you want to use some framework, library or component — always assess that that it will save time or money, or bring some essential stability or performance to the system that will compensate the dependency that you are locking yourself into. The same applies to infrastructure. I remember the old days when starting a new project we had to guess beforehand (dependency) what kind of servers we think we are about to need for development, testing and running production, send orders for them (dependency), wait for some guy to install servers to some data center (dependency), wait for another guy to open firewalls (dependency) etc. Cloud infrastructure has liberated us from all that. I as an architect am no more dependent to some ordering process, or some tech guys to do some installations. I don't have to guess beforehand what we are going to need. I start small to save customer's money and pay as I go. I add services to my infrastructure solution in an evolutionary manner without needing to wait anyone else. Everything happens a lot faster in the cloud era. And time is money, and saved time is saved money.

### No Low Level Stuff

In the old days some enterprise architect might have chosen some application server, some db server and some integration platform, and then we as application architects and developers just had to live with the realities and glue everything together and possibly do quite a lot of plumbing (low level stuff) to make everything work together. Not anymore. I really do love the AWS ecosystem. Amazon Web Services has a great portfolio of anything an architect would need for building an enterprise system: networks, virtual servers, databases as services, queues and notifications as services etc. And the best part is that all services really are high level services that integrate to each other beautifully — you really can [use AWS services as building blocks to implement your enterprise system](https://medium.com/tieto-developers/use-aws-services-as-building-blocks-to-implement-your-enterprise-system-598676a0ee49).

### Explode Your Comfort Zone

In the old world most projects were somehow complex and slow — all kinds of servers and components that really didn't integrate well together, and you had to specialize to some area, e.g. being an application architect. No more. Because AWS services integrate well together and you have less dependencies you need a lot smaller team and you can move on a lot faster pace than in the old days. You have more time and you have a chance to do a lot more yourself (no dependencies). You realize that instead of being just an application architect you get to be also infrastructure architect, enterprise architect, security specialist, database admin, performance specialist, tester, developer etc. — and it's a lot of fun!

### Point of No Return

Is there going back to the old world software development with all kinds of ugly and slow dependencies, uninteresting low level stuff that really didn't bring any value to the customer or locking yourself into a pitiful small role in a mega project? No, I don't think so. For customer using cloud is going to be a lot more cost effective way to build software (because it is faster and you need a smaller team) and also running production (economy of scale). And this is good news for architects and developers — you can explode your old comfort zone and start learning new skills, and you realize how much more fun building software is going to be now.

  
