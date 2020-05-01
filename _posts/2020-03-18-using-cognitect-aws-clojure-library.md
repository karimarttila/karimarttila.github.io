---
layout: post
title: "Using Cognitect AWS Clojure Library"
category: [clojure]
tags: [clojure, aws, dynamodb, cloud, iac, docker]
date: 2020-03-18
---

### Introduction

![](/img/2020-03-18-using-cognitect-aws-clojure-library_img_1.png)

*A Clojure REPL session using IntelliJ IDEA / Cursive.*

I joined [Metosin](https://www.metosin.fi/en/), a Finnish Clojure shop, at the beginning of this year. For refreshing my Clojure skills I decided to re-implement my old Clojure Simple-Server exercise using Metosin libraries before I joined the company. In the original version of the Simple-Server that I implemented some three years ago, I had used [defprotocol](https://clojuredocs.org/clojure.core/defprotocol) mechanism to provide a single-node version (using CSV files to simulate database) and an AWS version using DynamoDB as a data store. In that [previous Simple-Server version](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server) I had used [amazonica AWS Clojure library](https://github.com/mcohen01/amazonica). In this new version of the Simple-Server exercise, I decided to try [Cognitect AWS Clojure library](https://github.com/cognitect-labs/aws-api). In this blog post, I shortly list some experiences using the Cognitect AWS library with Clojure. If you are interested in Clojure development, in general, you might be interested to read also my previous related blog article [Clojure Impressions Round Three]({% post_url 2020-01-06-clojure-impressions-round-three %}).

You can find the project in [Github](https://github.com/karimarttila/clojure/tree/master/webstore-demo/simple-server).

### Local DynamoDB Docker Emulator

I used [local DynamoDB Docker emulator](https://hub.docker.com/r/amazon/dynamodb-local/) while developing the DynamoDB implementation for the interfaces. I used the DynamoDB scripts I already implemented in the first version of this Clojure Simple-Server exercise, see the [dynamodb](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server/dynamodb) directory for more information.

When I had implemented all functions in all interfaces I deployed the tables to the real AWS DynamoDB and ran the tests hitting the real AWS DynamoDB. You can create the tables, import data, etc. using those scripts both for local DynamoDB Docker emulator and for the real AWS DynamoDB.

Why should you use the DynamoDB Docker emulator? Well, for two reasons. Firstly, you don't generate any costs while doing the development using the local DynamoDB Docker emulator (though, I must say that developing this application required so little resources that in my AWS account everything could be done just using the free tier). Secondly, using DynamoDB locally is faster than sending and fetching data over the wire to your nearest AWS region. E.g. I am in Finland and I used Ireland region, comparisons:

![](/img/2020-03-18-using-cognitect-aws-clojure-library_img_2.png)

Timing the tests... and from Finland hitting the real AWS DynamoDB located in Ireland: **0m42.796s** => takes about 4 times longer to run all tests.

### Using Cognitect AWS Clojure Library

Using the [Cognitect AWS Clojure Library](https://github.com/cognitect-labs/aws-api) was really a joy. The library is very intuitive and provides excellent tools for exploring the API itself. Example listing the functions available for interacting with DynamoDB and querying more specific API description of one API function (PutItem):

![](/img/2020-03-18-using-cognitect-aws-clojure-library_img_3.png)

Exploring the Cognitect AWS API using Clojure REPL.Since you can use Clojure REPL and interact with the DynamoDB with REPL it is pretty developer-friendly to program with the Cognitect AWS Clojure library. I used my Clojure scratch namespace for experimenting with DynamoDB and when everything worked fine I moved the Clojure S-expression to the production source tree. Example of fetching products from a certain product group:

![](/img/2020-03-18-using-cognitect-aws-clojure-library_img_4.png)

So first, you use the above-mentioned **aws/doc** function to find out what Query operation requires as parameters (e.g. DynamoDB table name (**:TableName**)…) and then you can experiment with the API in the REPL and check what the operation returns — and further experiment with the Clojure REPL how to convert the result set into a format you need in your solution.

### Conclusion

The Cognitect AWS Clojure library provides an excellent tool to query the API itself. The library is like created for using a Clojure REPL — probably for the very reason that it was created at Cognitect, the home of Clojure.

*The writer is working at [Metosin](https://www.metosin.fi/) using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
