---
layout: post
title: "Jenkins on AWS"
category: [devops]
tags: [devops, ci, aws, jenkins]
date: 2017-10-10
---

![](/img/2017-10-10-jenkins-on-aws_img_1.png)

*Jenkins.*

### Introduction

I started a new project and there were two options for [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration): 1. Running CI as a hosted service using [Travis](https://travis-ci.org/). 2. Using my old friend [Jenkins](https://jenkins.io/). There were some pros and cons with both options:

- **Travis**. Pros: Hosted as a service — no installation / configuration hassle. Just start using it. Cons: Possibly some issues / hassle with integration tests to hit the AWS services on our AWS development account.
- **Jenkins**. Pros: Total ownership of the server — you can tweak it any way you like. Also Jenkins being on the same AWS account as the AWS resources under testing is a big plus. Cons: Installation / configuration hassle.

I have been using Jenkins/Hudson for several years and it has been a practice to install and configure Jenkins myself for my new projects — therefore I wasn't that afraid of the cons of option #2. I was more afraid that using Travis 95% I want from CI is very easy (using it as a service) but the rest 5% (hitting AWS resources etc) might be painful (since most AWS resources are on AWS account in private subnets behind bastion hosts etc.). The decision was to use Jenkins.

### Installation

I actually once installed Jenkins on a docker container and deployed it to AWS [EC2 Container Service](https://aws.amazon.com/ecs/) (ECS). I remembered that dockerizing Jenkins and needed plugins was pretty simple but deploying it to ECS was a bit of a hassle (maybe I'll have a look of the old Docker/Terraform scripts I created and write another blog article about that later). This time I didn't want to spend too much time with Jenkins installation, so I dediced to use plain [EC2](https://aws.amazon.com/ec2/) and avoid extra automation since this is basically a one-time deployment task (if you deploy something once a day or once a week — fully automate it; but if you do it only once and automation takes a lot of time — just do it manually).

I did some automation, however. The kind of automation that is easy and serves as a documentation later. Example creating a simple [Packer](https://www.packer.io/) script to create the basic [AMI](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) you are going to tweak later on:

```json
"builders": [  
 {  
 "associate_public_ip_address": "true",  
 "type": "amazon-ebs",  
 "region": "XXXX",  
 "source_ami": "ami-785db401",  
 "instance_type": "t2.small",  
 "ssh_username": "XXX",  
 "subnet_id": "subnet-XXXXX",  
 "vpc_id": "vpc-XXXXX",  
 "ami_name": "XXX-jenkins-ami-{{timestamp}}"  
 }
 ```
 
 I also added Packer to install some basic tools I was going to need like Git, Emacs etc. I could have added the installation of Jenkins itself in the packer script but I decided that I do it manually when the AMI is ready on AWS. When the new AMI was built by Packer, I launched it using AWS Console and once the server was up and running I logged on to check everything was good. Then I terminated the EC2.

The next step was to create the Terraform scripts to create the AWS infra around Jenkins — that is definitely something you want to automate also in a one-time deployment like this. One reason is that you get an excellent documentation regarding your AWS infrastructure in the form of Terraform code, and another reason is that using Terraform it is kind of easier to see your infrastructure in one glance in a couple of configuration files where all components are glued nicely together than to navigate in various AWS console views to figure out where everything is. So, I created a dedicated small (t2.nano EC2 server) bastion host for my Jenkins server (so that all connections to my Jenkins server go through bastion host, an extra onion level of security). I configured a security group for the bastion host to accept SSH connections only from my IP address (and later developers can add their IP addresses). I also configured a SSH key for this bastion host. Then I created a bigger EC2 server type with bigger disk for my Jenkins server with another security group accepting connections only from my bastion host to ports 22 (SSH) and 8080 (Jenkins http port), and another SSH key. I also decided to create a poor man's backup service for my Jenkins server. So I created a S3 bucket for backups and an AWS IAM policy for the EC2 server to allow storing backup files to that S3 bucket. And of course I added the AMI identifier I previously got to be used with the Jenkins server EC2. Below some examples regarding Terraform automation.

```terraform
resource "aws_instance" "ci_jenkins_server" {  
 ami = "ami-XXXXX"  
 instance_type = "t2.medium"  
 subnet_id = "${var.subnet_id}"  
 key_name = "${var.instance_ssh_key_name}"  
 vpc_security_group_ids = ["${aws_security_group.ci_jenkins_server_sg.id}"]  
 iam_instance_profile = "${aws_iam_instance_profile.ci_jenkins_instance_profile.name}"  
```

...The Jenkins installation itself I did pretty much manually. You could automate this part e.g. using [Ansible](https://www.ansible.com/) but since this was pretty much a one-time task I didn't want to automate these parts but did them mostly manually. I knew that once all configurations are ready I was about to create a new AMI from this manually baked EC2 instance — and that would be my final AMI to be used in Terraform configuration.

So, earlier I had chosen Ubuntu as the basic AMI so installing Jenkins is just calling "sudo apt-get install jenkins". Then get the initial admin password and logon to Jenkins using the web console (before that you create an SSH tunnel via your bastion host from localhost to Jenkins EC2 port 8080, of course).

Then logon also to EC2 server (i.e. using ssh, via bastion host, of course) and get the Jenkins Command Line Interface:

```bash
wget <http://localhost:8080/jnlpJars/jenkins-cli.jar>
```

I had a lengthy list of Jenkins plugins I needed to install so I used a simple script to loop the list of my plugins and call the jenkins-cli to install the plugins one after another. If you have just a couple of plugins, just install them manually.

Then I created a dedicated Jenkins user to my team's Github team and created a key pair for Jenkins server to be used as an authentication method to our team's Github projects. Another solution would be to configure [OAuth](https://developer.github.com/apps/building-integrations/setting-up-and-registering-oauth-apps/) which would be a bit more transparent but let's do that another time.

We are almost there. Next step is to configure email so that developers can subscribe emails of failed builds. I used AWS [Simple Email Service](https://aws.amazon.com/ses/) (SES) for sending emails. I created IAM user with policy "ses:SendRawEmail", i.e. the IAM user can just send emails. Then I added this user's AWS credentials to Jenkins so that Jenkins can use this IAM user's credentials to call SES service. Remember to register your email address to SES service and verify the address or SES won't send the emails to you. Also remember to add plugins [email-ext](https://wiki.jenkins.io/display/JENKINS/Email-ext+plugin) and [emailext-template](https://wiki.jenkins.io/display/JENKINS/Email-ext+Template+Plugin) to your plugins.txt or install them manually. Also verify that your company's email service is not accidentally migrated the very day you are doing the configurations. (As happened to me, can you believe the coincidence?) After testing with:

```bash
cat ses-input.txt | openssl s_client -crlf -quiet -connect <aws-email-server>:465
```

... and figuring out that it sends emails to my personal address but not company address I remembered reading in the intranet about the email migration, damn.

Now we are almost there. The final part. Adding jobs and testing that everything works. Configure Jenkinsfile in your Git project, e.g.

```groovy
node {
 try {
 stage "Checkout"
 git credentialsId: 'XXXXXX',
 url:'[git@github.com](mailto:git@github.com):XXXXX/YYYYY.git'
 stage "Local-H2-Unit-Tests"
 sh "lein with-profile +XXX-h2,+log-dev test"
 stage "Integration-Redshift-Tests"
 sh "lein with-profile +XXX-redshift,+log-dev test"
 } catch (err) {
 currentBuild.result = "FAILED"
 notifyFailed()
 throw err
```

...Enlightened readers notify that we are building using [Leiningen](https://leiningen.org/) (i.e. a [Clojure](https://clojure.org/) project, see my previous blog articles about it: [Clojure First Impressions]({% post_url 2017-08-29-clojure-first-impressions %}) and [Clojure Impressions Round Two]({% post_url 2017-09-14-clojure-impressions-round-two %}), so I naturally had to install Leiningen first. Once your Jenkinsfile is ready you configure a Jenkins job and add your Jenkinsfile to script path. Do some testing: commit changes and check that your build gets triggered and all pipeline stages pass, commit some tests to fail and see that the right pipeline stage fails and you get the email regarding a failed build etc.

As a paranoid person I also created a poor man's backup: A simple bash script that zips Jenkins configurations and workspace directory, stamps timestamp to zip filename and pushes the zip file to S3 Jenkins backups bucket, just in case. And added a cron task to call this script every night.

Now, we are finally there. For the last time: Stop Jenkins service, stop EC2 server, create a new AMI image of this EC2 instance, add the AMI identifier to your CI Terraform infra, terraform plan, apply and voila — your CI infra is up and running. Now you have your Jenkins server AMI nicely backed up, and all your CI infra in Terraform code. You can destroy and create the whole CI infrastructure in a few minutes fully automated. And for the next project you don't have to do this again — you just reuse the AMI and Terraform code.

### Conclusions

For an experienced AWS / Jenkins developer the whole process should not take more than a few hours. And you have a CI infra that sits right where your AWS components are that are under testing in your Continuous integration process, and you have full power to tweak your CI server any way you need. And you also can reuse the infra for your next project. In AWS development this is pretty powerful way of working since e.g. hitting Redshift in your tests using ssh tunnel, and tearing down previous test set and setting up next test set before every test takes a bit of time. So, locally in my personal Ubuntu laptop I run the tests using [H2 database](http://www.h2database.com/html/main.html) (a poor man's Redshift simulation) and let Jenkins to verify that the same tests run also in AWS [Redshift](https://aws.amazon.com/documentation/redshift/) which is the actual datastore the application under testing is using in production.

  
