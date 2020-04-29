---
layout: post
title: "Using Clojure to Implement a Web Service Server"
category: [clojure]
tags: [clojure, web]
date:	2018-03-25
---

### Introduction

I was asked to create a proof-of-concept for storing and analyzing clickstream events. The events would be stored in a data lake using AWS S3 and AWS Glue, and for testing the data lake I needed a clickstream generator that simulates customers browsing product groups and products in a hypothetical web store. The picture below shows the AWS infrastructure of the POC.

![](/img/2018-03-25-using-clojure-to-implement-a-web-service-server_img_1.jpeg)

*Proof-of-concept of a S3 based Data lake for Clickstream events.*

So, this would be a good reason to refresh my [Clojure](https://clojure.org/) programming skills (see my earlier Clojure related blog post: [Clojure Impressions Round Two](https://medium.com/tieto-developers/clojure-impressions-round-two-f989c0945f4b)).

The implementation of the Clickstream Web service server is in my github account: <https://github.com/karimarttila/clojure/tree/master/clickstream-generator> . The link shows the README.md which explains most of the technicalities, so I’m not going to explain the same stuff here. Instead, I’ll write here some experiences implementing a Web service server using Clojure, how to develop a Clojure application and how to integrate the Clojure application with AWS.

### Clojure REPL Rocks

I have been using Tensorflow lately in [my Machine learning exercises](https://medium.com/@kari.marttila/writing-machine-learning-solutions-first-impressions-93b5e4385970), and therefore used quite a bit the Python REPL (both with [PyCharm](https://www.jetbrains.com/pycharm/) and command line). Python REPL is good (if you have used languages like Java which provide no REPL at all — except Java 9 introduced some REPL which I haven’t tried yet). But every time I use the Python REPL I just miss the power and easiness of the Clojure REPL, especially when using it with [Cursive](https://cursive-ide.com/userguide/repl.html). And using Cursive is even more enjoyable if you assign good hot keys for it. Some of my own favorite Cursive hot keys are:

* <shift><ctrl><N> (In editor)=> Switch REPL namespace to current file (which is in editor when the command is given).
* <shift><ctrl><M> (In editor)=> Load current namespace to REPL.
* <shift><ctrl><Å> (In editor)=> Send S-expression to REPL input.
* <esc><esc> => Focus to Editor last edit (this is a standard functionality in Cursive).
* <caps><alt><I> => Focus to REPL output.
* <caps><alt><O> => Focus to REPL input.
* <caps><ctrl><I/K> (In REPL) => Browse REPL history.
* <ctrl><enter> (In REPL) => Send REPL input to REPL to be evaluated.
* <caps><alt><J/L> (In editor) => Switch tabs in editor.
* <ctrl><X>-<ctrl><O> (In editor) => Switch to next editor window (as in Emacs).
* <caps><alt><7> => (In editor) Reformat current file.
* <caps><alt><8> => (In editor) Run the test case which the cursor points currently.
* <caps><alt><9> => (In editor) Run all test cases in the file.
The following picture shows an example how to use the IntelliJ IDEA / Cursive REPL.

![](/img/2018-03-25-using-clojure-to-implement-a-web-service-server_img_2.png)

*IntelliJ IDEA with Cursive.*

So, in the editor window I have entered the hot key <shift><ctrl><N> which switches the REPL namespace to the current file, then entered <shift><ctrl><M> which loads the current namespace to REPL (I assigned keys N and M for these functions since they are often used together in that order). Then I have entered <caps><alt><O> which puts the IDE focus to REPL input window. Then I’m ready to explore and try any function in that Clojure namespace in the REPL. This is really a breeze since I can do all of this without ever leaving my fingers from the keyboard (I type using all 10 fingers and I have configured my Linux keymap for other hot keys, e.g. Caps Lock is a special key, and with Caps Lock and other keys I can move the cursor etc). This makes development really fluent since you can navigate between editor window and REPL really fast, load various namespaces to REPL and quickly try the functions in isolation in the REPL.

### Clojure Testing

The example above was created for an internal throw-away POC so I didn’t implement that many test cases for it. There is one test, however. The test verifies that the product group probabilities work as expected within a given delta. Test cases are pretty easy to create using Clojure. Clojure code is tight and pretty declarative so it is usually very easy to see the purpose of the test code just by looking at it. For tests I have two favorite hot keys: <caps><alt><8> which runs the test case which is currently in cursor in editor, and <caps><alt><9> which runs all test cases in the file.

### Web Service Implemented Using Clojure

Starting/stopping the clickstream server is exposed as a web service. I have used two widely used Clojure libraries for the web service functionality:

* [Ring](http://ring-clojure.github.io/ring/) which is a Clojure web applications library.
* [Compojure](https://github.com/weavejester/compojure) which is an excellent routing library for Ring
Take a look of the [server.clj](https://github.com/karimarttila/clojure/blob/master/clickstream-generator/src/csgen/webserver/server.clj) file which provides the web service server functionality. The web server initializes itself and then goes to stopped state (not sending clickstream events to Kinesis stream). When you want to start the demonstration you send a http command to the web server:

curl <http://localhost:3000/start?token=><your-token-in-plaintext>The example above sends the command to the web server that is running locally and listening in port 3000. The token is just a simple security measure so that not anyone can send commands to the server.

The server provides simple routing of these urls:

```clojure
(co-core/defroutes app-routes  
 (co-core/GET "/info" [] (-get-info))  
 (co-core/GET "/status" [token] (-get-status token))  
 (co-core/GET "/start" [token] (-start-click-generator token))  
 (co-core/GET "/stop" [token] (-stop-click-generator token))  
 (co-route/resources "/")  
 (co-route/not-found "Not Found. Use /info for information."))
```

I use the Clojure [async/go](https://clojuredocs.org/clojure.core.async/go) functionality to generate the clicks while the web server itself continues to listen commands in the main thread:

```clojure
(defn -generate-clicks  
 []  
 (log/trace "ENTER -generate-clicks")  
 (async/go (while (= [@server](http://twitter.com/server "Twitter profile for @server")-state :running)  
```


### Clojure and Data Handling

Using Clojure to implement data oriented applications is easy since Clojure is very much data oriented language itself. I don’t miss that much of the Java boilerplate (classes, setters/getters etc.) to implement simple stuff. With Clojure you don’t need any of that boilerplate but you can focus on the application data and logic itself.

### Clojure and AWS Integration

There is an excellent AWS API library for Clojure: [Amazonica](https://github.com/mcohen01/amazonica). There is an example in the [streamer.clj](https://github.com/karimarttila/clojure/blob/master/clickstream-generator/src/csgen/stream/streamer.clj) file how to use it:

```clojure
(defn -put-record-to-kinesis  
 "Puts a url event to kinesis stream."  
 [url]  
 (let [kinesis-stream-name (cg-prop/get-str-value "url-kinesis-stream-name")  
 data {:url url}]  
 (amazonica-kinesis/put-record kinesis-stream-name  
 data  
 (str (java.util.UUID/randomUUID)))))
```

If you are interested to use Clojure with AWS you should explore Amazonica — there are API functions for all major AWS services like Kinesis, EC2, S3, Lambda, SNS, SQS… And if you don’t find a Clojure API for some specific AWS service or functionality you can always use the [Java AWS SDK](https://aws.amazon.com/sdk-for-java/) since Clojure is a JVM hosted language and seamlessly integrates with Java.

### Deployment

There are two basic ways to create a deployment unit for this kind of web server in an AWS environment: AWS AMI and Docker container. AWS Lambda is not an option since the web server is a stateful entity (knows whether it is in a stopped or running state). For the POC I wanted minimal hassle so I used Packer and Ansible to create a simple AMI which could be then deployed as part of the AWS POC infra. I have already covered how to create AWS AMIs and Docker containers in other blog posts, e.g. How to [Create EC2 Images in AWS?](https://medium.com/tieto-developers/how-to-create-ec2-images-in-aws-a27b1afc97c6), [How to Create Docker Containers in AWS?](https://medium.com/tieto-developers/how-to-create-docker-containers-in) and [AWS Batch and Docker Containers](https://medium.com/@kari.marttila/aws-batch-and-docker-containers-41c92784bd96).

### Conclusions

Using Clojure to implement all kind of software components is really a joy. Using Clojure REPL makes the software development an exciting exploratory journey — no more long compile-build-deploy-test cycles but you can test small Clojure functions in isolation using the REPL. You can then package your Clojure applications as an AWS AMI or a Docker image to be used in an AWS environment.

  
