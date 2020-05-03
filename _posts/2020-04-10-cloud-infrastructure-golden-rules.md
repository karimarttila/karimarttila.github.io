---
layout: post
title: Cloud Infrastructure Golden Rules
category: [iac]
tags: [iac, aws, azure, gcp, cloud, productivity, quality, featured]
date:	2020-04-10
---

![A screenshot of one of my personal cloud projects](/img/2020-04-10-cloud-infrastructure-golden-rules_img_1.png)

*A screenshot of one of my personal cloud projects.*

### Introduction

I have been working in various cloud infrastructure projects using AWS, Azure, and GCP implementing infrastructure with native tools and with Terraform and Pulumi. I have seen that there are a few success factors how to make infrastructure in software development projects. I have also seen some tar pits to avoid. In this short blog post, I write about my experiences regarding how to do infrastructure as code — and how not to do it.

I am a software developer. I have a software developer's point of view when I create cloud infrastructure. I'm talking here about software development projects — we create both the applications and the cloud infrastructure in which we run the applications. (I.e. in this blog post I'm not talking about how to create cloud infrastructure for migration or lift-and-shift projects.)

### Two Golden Rules Above All

If you don't remember anything else after reading this blog post, remember these two golden rules:

1. All infrastructure must be created fully automatically by the infrastructure code (whether it's Terraform, Pulumi or some cloud native code like CloudFormation).
2. The infrastructure solution must be so simple that any developer in the software project is able to do both applications and infrastructure.
The rest of this blog post is more or less elaborating what these two infrastructure golden rules mean.

### Golden Rule 1: All Infrastructure Must Be Created Fully Automatically by the Infrastructure Code

It is so common how often this rule is violated and practically always violating this rule ends up with a fragile infrastructure.

Once a customer asked my team to audit a cloud project that had some issues to deliver results at a faster pace. The story was, the client told us, that in the beginning the project seemed to be progressing nicely but had halted later on. My team took a look at the project — everything was implemented using the cloud provider's portal. No wonder the progress was fast in the beginning — it is fast to create the infrastructure using the portal. But the ugly side of the coin is that you can't automatically create identical copies of that infrastructure — since every single detail in the infrastructure was added using the Portal. And you are going to need identical copies — for developer experimentations, for CI/CD, test team, performance testing, customer acceptance testing, production.

Another violation of this rule is when at the beginning of the project the infra developers have good intentions to use an IaC tool. But in a hurry every now and then they cut corners — let's add this little detail using the Portal since I don't have time now to figure out how to do it using my IaC tool. And then, later on, you forget those little things and another developer needs to create a copy of the infrastructure e.g. for performance testing — and the infrastructure doesn't work. "But it works in my machine!" — sorry, you can't use that phrase anymore. What you have now is one single working but brittle infrastructure environment which is not defined in the infrastructure code — and no way to reproduce it. So: you must not add to the infrastructure any resources bypassing your infrastructure code.

You should keep your infrastructure code in a version control system like Git, of course. And infrastructure code is code, and like with any code you don't commit broken code into a Git repository. When you pull latest from a Git repository the first thing you should do without making any modifications is to run the apply command ("terraform apply", "pulumi up"…) with the code. There should be no changes to the infrastructure at this point, of course. If the IaC tool says it needs to add, delete or modify some resources either someone did commit to a Git repository infra code without applying it to your common environment or your infrastructure code is broken. A professional programmer in any language won't create new code if some tests are broken — first, you fix the tests and then you continue to add new code. The same way an infra programmer should stop adding new infra code if the infra deployment is broken — i.e. if you can't run your infra deployment with one IaC command fully automatically without errors. Don't continue adding tricks and hacks to the infra code if your infra code is broken — stop adding new features and fix the root cause first. If you let one hack inside you soon have twenty of them.

There is a very simple way to validate the infrastructure solution every now and then. Ask one junior developer with not that much experience doing infrastructure to create the infrastructure from scratch — a very simple test and non-intrusive since he or she can create a personal copy of the infrastructure. If he or she can't do it — your infrastructure code is either not automatic and/or it isn't simple.

### Golden Rule 2: The Infrastructure Solution Must Be Simple

The infrastructure solution must be so simple that any developer in the software project is able to do both applications and infrastructure.

The most successful software projects in which we built both applications and cloud infrastructure followed this rule by the book — every team member could work both with apps and infra.

I have seen ambitious cloud infrastructures that violated this rule. The cloud infrastructure ended up as a complicated beast and comprised various hacks and tricks you had to remember to do when running the IaC tool. Wrong! Your cloud infrastructure should be simple and automatic so that you can create the whole infrastructure from scratch by just cloning the infra Git repo and running your IaC tool against the infra code, and using the IaC tool just once (e.g. "terraform apply" or "pulumi up"), with no errors.

Another consequence with complex cloud infra is that you end up with two clans: application developers and infra magicians. Only the infra magicians are able to modify the infrastructure. Gosh, no! If I need a new cloud resource for my application, I don't want to wait when some infra magician has time to fulfill my humble request — no, I want to create a personal copy of the infrastructure right now, add this new cloud resource to my infrastructure and experiment with it. I consider "two clans" as a major anti-pattern in software projects.

### How Well Does Your Infrastructure Score?

If I audit some cloud infrastructure I could be using e.g. these simple scores from zero to five.

**0 points**. No infrastructure code. The infrastructure was implemented ad hoc using the cloud provider's portal. Shame on you!

**1 points.** No real infrastructure code but you didn't use the portal either. The infrastructure is created by cryptic bash shell scripts calling cloud provider's cli, parsing the returned json using jq and then passing the results to another cli command. The hideous solution comprises documentation emphasizing that in step 27 you have to create a text file for certain environment variables (but only if you create infrastructure for testing environment since in testing environment you need to…). Shame on you!

**2 points**. You did use an IaC tool. But your solution violates both infrastructure code golden rules. You added little pieces here and there bypassing the IaC tool. Your infrastructure has documentation which says that in the first run you need to add flags A, B and C, and in the first run the IaC tool fails "but it's ok". Then turn on flag A and B, copy-paste the security group id that was created in the first run into file X and do the second run. Your project is divided between the application team and the infra team (the magicians who are the only persons who know how to maintain the fragile infrastructure). Shame on you, you could do better than that!

**3 points**. Your infrastructure is simple. There are no flags, hacks or tricks, manual steps or long documentation — the infra code is the documentation. You are able to create a new environment just by running the IaC tool (once and fully automatic without errors). Any developer can create a personal copy of the infrastructure for his or her purposes to experiment with some ideas. And once the developer is happy with the experiment he or she can destroy the environment with one IaC command (fully automatic and without errors, of course) — and all infra resources are gone, no orphans left behind silently to accumulate costs). Well done! But — you can do better.

**4 points**. All aspects of the 3 points category are implemented. The solution needs to be fully automatic but also elegant and maintainable — all cloud resources need to have a consistent name and tag scheme, cloud resources need to be defined in reusable modules etc. And additionally, your IaC also creates all monitoring, alarms, and dashboard fully automatically when you create a new cloud environment. Wow! You can simulate production in every aspect. Great! But there is one more step if you want to be perfect.

![Default tags that make it easy to look for resources by tags in AWS](/img/2020-04-10-cloud-infrastructure-golden-rules_img_2.png)

*Default tags that make it easy to look for resources by tags in AWS.*

**5 points**. This is the highest level of cloud infrastructure perfection. In this category, you have a simple IaC solution that you are able to create with one IaC command and destroy with one IaC command, fully automatically and without any errors. Your infrastructure also creates all monitoring, alarms, notifications, and dashboard for your infrastructure. But additionally, your IaC mechanism after creating the cloud infra also fully automatically creates all databases, test databases with application users and rights, installs all applications to cloud infrastructure compute entities (whether they are virtual machines or containers) and runs all e2e tests using those applications and databases hitting all API layers, monitoring results in your buckets, databases, and queues. And with one command your CI machine can automatically create your infrastructure every night, install latest versions of applications, run performance tests, destroy the infrastructure and warn you by email when you come to the office in the morning if the previous day something was added to the applications or infrastructure which resulted in performance to degrade.

### You Are Allowed To Use the Portal, But…

So, does this mean that I'm not allowed to use the portal or cloud cli tools at all and I only should use infrastructure code? Of course not. I use the portal and cli tools all the time. But: I don't create production systems using the portal or the cli tool. This is the way you should use them.

If you have a difficult problem you just can't figure out how to do using infrastructure code, first create the solution using the portal and/or cli tool. In some cases, the portal and cli tool create all kinds of magic behind the curtains that is hard to find out just by reading the documentation (e.g. roles and rights, annotations in resources, etc.). Then experiment with the solution you created with portal or cli tool: check that it works, and then carefully examine all resources that got created. Then re-implement the same resources using infrastructure code. If your infrastructure code solution doesn't work the same way you have a working reference solution (the one you created using the Portal) and examine carefully all resources side by side. Finally, you figure out how to do it using infrastructure code only. Migrate this infra code solution as part of your IaC solution and verify that nothing broke by creating the whole infra from scratch, at least every now and then.

### Feeling Is Important

Infrastructure should not be something intimidating, a magical black box that you should be afraid of, only allowed to be touched by those magicians in the infra team. Every developer in the software project should understand the infrastructure and feel a sense of ownership regarding the infrastructure. You should also feel a sense of proudness regarding your work — no-one should be proud of infrastructure that barely holds together and re-creating a new environment is a hideous and fragile endeavor applying various tricks and following cryptic instructions. The more points in our infrastructure scale from zero to five you have the more resilient and maintainable the infrastructure also tends to be.

### Conclusions

Cloud infrastructure programming is hard. But it is not rocket science. I have found out that the hardest part is not the technology itself but how to be stringent with some basic rules — no infrastructure creation bypassing the IaC tool and infrastructure solution must be simple. If you follow those two rules you already are on a good path in your software project.

*The writer is working at [Metosin](https://www.metosin.fi/) using [Clojure](https://clojure.org/) in building applications and using e.g. Terraform and Pulumi in building cloud infrastructures. If you are interested to start a cloud project in Finland you can contact me by sending an email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
