---
layout: post
title: "Studying Machine Learning — First Impressions"
category: [ml]
tags: [ml, ai]
date:	2018-01-01
---


![](/img/2018-01-01-studying-machine-learning-first-impressions_img_1.jpeg)

*[Technology image created by Kjpargeter — Freepik.com](https://www.freepik.com/free-photo/robot-with-nuts-and-bolts_958203.htm)*

### Introduction

Studying is one of my favourite hobbies and winters are good time for studying in Finland, since it's dark and cold and you have to stay inside most of the day. Last winter I studied [Clojure](https://clojure.org/), the incredible Lisp implementation on JVM (and used it to implement a [microservice to process Big data on AWS]({% post_url 2017-11-09-aws-batch-and-docker-containers %}), for more information about my Clojure experiences see my previous blog articles: [Clojure First Impressions]({% post_url 2017-08-29-clojure-first-impressions %}) and [Clojure Impressions Round Two]({% post_url 2017-09-14-clojure-impressions-round-two %}). A couple of winters before that I studied [AWS](https://aws.amazon.com/) (and have been using AWS quite a lot ever since, feel free to browse the experiences using AWS [here](https://medium.com/@kari.marttila)). This autumn and winter I decided to study [Machine learning](https://en.wikipedia.org/wiki/Machine_learning).

### What is Machine Learning?

[Artificial intelligence](https://en.wikipedia.org/wiki/Artificial_intelligence) is basically a generic term for any capability that a machine can be programmed/trained to do which we humans consider somehow intelligent. Machine learning is one of the subfields of Artificial intelligence. Wikipedia defines Machine learning as "*a field of computer science that gives computers the ability to learn without being explicitly programmed.*" Machine learning itself has many [subfields](https://en.wikipedia.org/wiki/Outline_of_machine_learning) which take different approaches how to teach machines to learn.

### Why Machine Learning?

I often have coffee discussions with my good friend and colleague regarding what to learn next, what is going to be the next big hit in IT (and therefore in which studies you should invest as a developer to be competent in the job market). We realized that Machine learning is a topic that tends to appear more and more in various software engineering sites. Machine learning was also a big topic in this year's AWS re:Invent conference (more about it [here]({% post_url 2017-11-29-aws-re-invent-2017-conference-reflections-part-1 %}). We decided together to dedicate this autumn and winter for studying Machine learning.

### Our Future Machine Learning Study Path

Machine learning itself is a broad field to study, so where to begin? We decided to make a Machine learning study path for ourselves. The Machine learning study path has the following three category:

#### 1. Learn the Basics

You could start using various higher level machine learning frameworks and platforms without learning how the algorithms actually work, but that might not be such a good idea. Instead, to get a more deeper understanding what Machine learning is and what is happening under the hood we decided also learn the basics. There is a lot of free material in the web to learn the Machine learning terminology, theory and how the basic algorithms work. I have this far been learning the primitive tools, [linear regression](https://en.wikipedia.org/wiki/Linear_regression), [logistic regression](https://en.wikipedia.org/wiki/Logistic_regression) and [artificial neural networks](https://en.wikipedia.org/wiki/Artificial_neural_network) doing various excercises using [Octave](https://www.gnu.org/software/octave/) which is a mathematics oriented programming environment. Most of the stuff related to primitive algorithms mentioned above can be implemented using vectors and matrices. Octave provides excellent tools for manipulating these data structures, it also provides a simple programming language and tools for plotting your data. So, I definitely recommend learning Octave if you need a simple environment for learning and experimenting Machine learning algorithms.

#### 2. Learn the Frameworks and Platforms

We have been using AWS a lot to build enterprise systems and most probably AWS is going to be our platform of choice for future Machine learning projects as well. AWS provides various [frameworks and services](https://aws.amazon.com/machine-learning/) for implementing Machine learning systems. What this actually means is that AWS provides higher level services for implementing e.g. image recognition or language processing, and also lower level frameworks in the form of specialized Amazon Machine Images which have popular Machine learning frameworks pre-installed in the AMIs. This is going to be the next step in our Machine learning path — let's write another blog article later on when we have done some excercises using these services and frameworks.

#### 3. Find the First Project

The third and most critical part of learning any new skill is actually to find a project and apply the theory and practical skills in real-life. You can spend countless hours at home in the evenings and weekends learning new skills but nothing is going to make the learning as fast as implementing a system and learning while you do actual work. Most probably the problem finding a project will solve itself since everyone predicts that the demand for machine learning will be quite big in the future. So, I take my Machine learning studies as an investment — once I have learned to apply it in real-life I raise the probability that I am also going to find a Machine learning project (which is pretty much what has happened with every new skill I have learned, e.g. with AWS — "once you learn it you get to use it").

### Conclusions

Studying Machine learning has been different than e.g. learning a new programming language (like Clojure) or studying some cloud infra (like AWS). Why is that? Learning a new programming language after you have learned a handful programming languages before that is not going to be a big deal — you just learn some new syntax, new programming idioms and possibly a new paradigm. Learning new cloud infra like AWS was a bit more challenging since you have to learn a lot of new concepts, how things relate to each other, and new tools actually to implement cloud infra (e.g. Terraform or CloudFormation). But studying Machine learning has been most challenging this far because a lot of the stuff you need to learn is not that much computer science specific but more mathematical and theoritical in nature (at least the basics). But it has been a lot of fun also. And if Artificial intelligence is going to be the [new electricity](https://medium.com/@Synced/artificial-intelligence-is-the-new-electricity-andrew-ng-cc132ea6264) then studying how to implement Machine learning systems will most probably be a very good investment for a developer.

  
