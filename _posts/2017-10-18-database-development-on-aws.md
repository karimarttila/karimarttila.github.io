---
layout:	post
title:	"Database Development On AWS"
categories: [blog, aws]
tags: [aws]
date:	2017-10-18
---

  ![](/img/2017-10-18-database-development-on-aws_img_1.jpeg)[Designed by Freepik](http://www.freepik.com/free-vector/big-data-cloud-vector-illustration_1322558.htm)### Introduction

Most of enterprise type backend development falls typically into two categories. Either you create some sort of web applications with REST interfaces or you create some backend processing systems. In both cases you typically work with databases. So, for a seasoned backend developer working with databases should be rather fluent. This does not mean just being fluent with SQL, but also how to make database development itself fast.

### Database Development on AWS RDS

I have used [PostgreSQL](https://www.postgresql.org/) and [MySQL](https://www.mysql.com/) running as [RDS](https://aws.amazon.com/rds) services on AWS. It’s pretty nice to work with these services since you can create the same database system in your local workstation. It’s always beneficial if you can use the same database in your development environment since you can install the same configurations, schemas etc in your workstation database as in your target environment — the tests you run in your workstation database should pass the similar manner in your target database.

Nowadays I have started to use [dockerized](https://www.docker.com/) versions of these databases in my own workstation. The reason is that there is no hassle to install the database or some conflicts between database versions (if I’m developing software in two different projects with the same database product but different versions…), and it also keeps my own workstation cleaner. If you go to Docker repository you can find dockerized versions of both [PostgreSQL](https://hub.docker.com/_/postgres/) and [MySQL](https://hub.docker.com/_/mysql/). E.g. starting docker script from my previous project:

docker run -p 6603:3306 --name XXXX-mysql -v /mnt/data/mysqldocker:/var/lib/mysql -e MYSQL\_ROOT\_PASSWORD=YYYYY -e MYSQL\_USER=kari -e MYSQL\_PASSWORD=ZZZZ -e MYSQL\_ROOT\_HOST=172.17.0.1 -d mysql/mysql-server:5.7… so: forwarding MySQL port 3306 in docker to host port 6603, using mount point to host /mnt/data/mysqldocker (I like to keep this kind of stuff in that mount point) etc. It complains giving password in command line but this is a development workstation and this is a test database so that’s not a problem.

So, the development model in this kind of setup is like this. I run local fast tests hitting the database in my own workstation. I keep open one console next to my IDE and basically after almost any changes (even small changes) I just click the Up+Enter keys to run the tests again. This way I immediately get notified if I broke something. Once I’m ready I commit my changes to Git and CI server (nowadays Jenkins) gets triggered and runs the same tests in the target database in AWS RDS, and then possibly some e2e tests (e.g. using [Robot framework](http://robotframework.org/)) and possibly some long lasting performance tests). The idea is to have a fast development cycle in my own workstation but also get every commit to be verified by CI.

### Database Development using AWS Redshift

You cannot install [AWS Redshift](https://aws.amazon.com/redshift) to your own workstation — it runs only on AWS. So, how do you do database development? You have basically two choices. You can either hit AWS Redshift during your development. Or you can use some poor man’s Redshift substitute. Usually Redshift is behind a bastion host and hitting Redshift via SSH tunnel might be rather slow. And if you have hundreds of small tests running your whole test suite might take almost an eternity — not an option for a restless developer.

So, I usually use some poor man’s substitute. AWS Redshift is based on PostgreSQL so PostgreSQL is a valid choice. You should check [the differences in AWS documentation](http://docs.aws.amazon.com/redshift/latest/dg/c_redshift-and-postgres-sql.html). In my recent project I decided to use H2 since the database queries are pretty standard ANSI SQL which run on every DB. [H2](http://www.h2database.com/html/main.html) is an excellent development database. There is basically no install hassle — you just add H2 as a dependency jar into your Java or Clojure project. And it’s very fast. You can run the database in RAM memory when it is blazingly fast. I use the file based version since it is fast enough and you can easily connect to the database even when your app is not running.

The development model in this setup is basically the same as explained in the previous chapter. But in this setup running the tests in the target environment (AWS Redshift) is much more important since the databases in your development environment and in your target environment are very different. If you install your CI to your AWS environment (as I explained in my previous blog post “[Jenkins on AWS](https://medium.com/tieto-developers/jenkins-on-aws-49133e011ac5)”), and your CI machine has a direct connection to your development env Redshift (i.e. development AWS account), you can let CI run the same test suite hitting Redshift and possibly some long lasting e2e / performance tests. If you broke something you’ll get notified by email in a few minutes (or you can keep your Jenkins dashboard glowing in some monitor).

### Clojure and Database Development

Using [Clojure](https://clojure.org/) in database driven backend development has really been an enermous development boost. Clojure has many excellent qualities for data driven development: all data structures are immutable, there are good tools to parallelize your tasks, functional programming model makes passing data from one function to another via filters, maps and reductions a blast, and the programming language is very developer friendly to work with basic data structures (sequences, vectors, maps, sets etc.).

(defn calculate-total-humidity-sum  
 "Calculates total humidity sum of measurement points"  
 [m-points]  
 (reduce (fn [sum item] (+ sum (item :humidity))) 0 m-points))  
...  
and later in automatic test:  
 (is (= (calculate-total-humidity-sum measurements-testset-1) 65.7))Can it get any easier than that? And using the Clojure REPL is a fantastic tool to interact with the system during development. If you have never used a Lisp REPL you just cannot understand what it is like. You can basically point any code segment (a Lisp S-expression) and tell your IDE to send that expression to REPL for evaluation. I use [Cursive](https://cursive-ide.com/) with [IntelliJ IDEA](https://www.jetbrains.com/idea/) and I have never used as productive developer environment before. More about my Clojure impressions in my previous blog articles [Clojure First Impressions](https://medium.com/tieto-developers/clojure-first-impressions-2c6232f4b514) and [Clojure Impressions Round Two](https://medium.com/tieto-developers/clojure-impressions-round-two-f989c0945f4b). I really can’t emphasize enough how much faster it is to work with data driven application development using Clojure than any other language I have used before (I’ve been in IT industry more than 20 years — I have used quite a few languages in my career: C, C++, Java, Perl, Python, Javascript and Clojure). As Rich Hickey, the creator of Clojure, [said](https://gist.github.com/yogthos/974ef18230ced6cd10dd9e2ec9cc5d82): “I discovered Lisp after ten years of C++ and said to myself, ‘What have I been doing with my life?’ I realized it was just much more productive for me. I fell in love with it right away.” If you are mostly doing data driven development, please give Clojure a chance. Clojure being a Lisp might at fist glance look a bit difficult but once you learn to appreciate the enormous productivity boost with data driven development using Clojure, you are not going to look back to your old language.

The writer is AWS Certified Solutions Architect Associate, architecting and implementing AWS projects in Tieto CEM Finland using Python, Java and Clojure. If you are interested about starting a new AWS project in Finland, you can contact me with firstname.lastname at tieto.com.

Kari Marttila

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
  