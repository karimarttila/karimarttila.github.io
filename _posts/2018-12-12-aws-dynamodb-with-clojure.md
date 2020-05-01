---
layout: post
title: "AWS DynamoDB with Clojure"
category: [aws]
tags: [aws, iac, cloud, clojure, dynamodb]
date: 2018-12-12
---

![](/img/2018-12-12-aws-dynamodb-with-clojure_img_1.png)

*Running Clojure unit tests in IntelliJ IDEA / Cursive.*

### Introduction

After my [Five Languages](https://medium.com/@kari.marttila/five-languages-five-stories-1afd7b0b583f) project I was searching something new to learn. I decided to refresh my AWS skills a bit and and compare AWS container services [EKS](https://aws.amazon.com/eks/) and [Fargate](https://aws.amazon.com/fargate/) and compare their deployment models for Docker containers. I decided to use my [Clojure Simple Server](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server) exercise as a demo application. There was one problem, though. The Simple Server was just a demo application and it simulated database by reading csv files to internal data structures — therefore making the server stateful. First I needed to change the implementation to make the server fully stateless — storing all state outside the server in a real external database. I decided to use AWS [DynamoDB](https://aws.amazon.com/dynamodb/) nosql database. In this blog post I write about my experiences to manipulate DynamoDB using Clojure and how to use local DynamoDB Docker container instance as a development database. Next I’ll be using this DynamoDB version of Simple Server to deploy the server to EKS and Fargate and write new blog posts regarding those experiences, so stay tuned.

You can find the project in my [Github account](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server).

### Local DynamoDB Docker Container Based Development Database

For a small exercise with minimal demo data set the DynamoDB price is almost neglectable, but there there is always the round-trip latency to AWS data center. Therefore I decided to use a local [DynamoDB Docker container](https://hub.docker.com/r/amazon/dynamodb-local/) based development database — and have a nice excuse to learn to use it. The local DynamoDB was a positive surprise — all DynamoDB APIs I used (cli, Python/boto3, Clojure/amazonica) worked nicely out of the box. I found only one item that worked differently in local DynamoDB compared to real AWS DynamoDB service (more about that later).

You can start the local DynamoDB Docker container using command:

docker run -p 8000:8000 amazon/dynamodb-local:latestThat’s it! You have a running DynamoDB in your workstation and you can use [AWS command line interface](https://aws.amazon.com/cli/) (cli), or any DynamoDB API SDK to access this DynamoDB instance.

### Let’s Create Tables

I created a sub-directory [[dynamodb](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server/dynamodb)] for DynamoDB related utilities I used in this project. You can find in that directory [create-tables.sh](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/dynamodb/create-tables.sh) script which creates the four tables needed in the project: session table, users table, product groups table and products table. It is pretty straightforward to use aws cli to create the tables, and the script works the same way with local DynamoDB Docker container instance and with real AWS DynamoDB service. Example of one aws cli call to create the product table:

AWS_PROFILE=$MY_AWS_PROFILE aws dynamodb create-table $MY_ENDPOINT — table-name $MY_PRODUCT_TABLE — attribute-definitions AttributeName=pgid,AttributeType=S AttributeName=pid,AttributeType=S — key-schema AttributeName=pid,KeyType=HASH AttributeName=pgid,KeyType=RANGE — provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 — global-secondary-indexes IndexName=PGIndex,KeySchema=[“{AttributeName=pgid,KeyType=HASH}”,”{AttributeName=pid,KeyType=RANGE}”],Projection=”{ProjectionType=INCLUDE ,NonKeyAttributes=[“title”,”price”]}”,ProvisionedThroughput=”{ReadCapacityUnits=5,WriteCapacityUnits=5}”The endpoint is used only for local DynamoDB Docker container instance and it points to the port you deployed the container earlier:

MY_ENDPOINT=” — endpoint-url http://localhost:8000"### Let’s Import Data

The next task is to import the test data. I used this as an excuse to refresh my Python/[boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) skills. I could have used Clojure for this as well but since this is my personal project I can do it any way I like. I thought that it might be interesting to do the data import using Python.

First we need to create the virtual environment for python3, activate the virtual environment and then install boto3 to the virtual environment (well, you don’t have to use virtual environment but I personally almost always create a virtual environment to keep my workstation Python installation cleaner):

```bash
# Create the virtual environment for Python.  
./create-virtual-env.sh  
# Activate the virtual env.  
source venv3/bin/activate  
# Install aws library boto3.  
pip install boto3Then create the local-dynamodb profile and dummy AWS credentials in your ~/.aws/credentials file:
```

```bash
[local-dynamodb]  
aws_access_key_id = XXXXXXXXXXXXXX___NOT  
aws_secret_access_key = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX___NOT
```

NOTE: When I first experimented with the local DynamoDB Docker container instance I just couldn’t make it work. Then I tried a real access key and secret key generated by AWS — and they magically worked. So, I guess the local DynamoDB Docker container instance expects the access key and secret key to be with valid length even though the content is irrelevant.

Then you are ready to import the data using the script:

./run-local-dynamodb.sh # Start the DynamoDB Docker container in another terminal.  
./import-all-tables.sh local-dynamodb dev # Import all tables to that instance. Calls table_importer.py for each 4 tables.The [table_importer.py](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/dynamodb/table_importer.py) is pretty simple. First we get the dynamodb client:

```python
if my_aws_profile == ‘local-dynamodb’:   
 dynamodb = session.resource(service_name=’dynamodb’, endpoint_url=’http://localhost:8000')   
else:   
 dynamodb = session.resource(service_name=’dynamodb’)And then we just dump everything from the csv file to DynamoDB:

with open(my_csv_file, ‘r’) as csvfile:   
 reader = csv.reader(csvfile,delimiter=’\t’)   
 with table.batch_writer() as batch:   
 for user_id, email, first_name, last_name, hashed_password in reader:   
 batch.put_item(Item={“userid”: user_id, “email”: email, “firstname”: first_name, “lastname”: last_name, “hpwd”: hashed_password})
```

That’s it! With two commands we can create the development tables and import the test data to the development tables.

### Clojure Implementation

Programming [Clojure](https://clojure.org/) was once again a real joy. Clojure REPL and especially the [Cursive](https://cursive-ide.com/) REPL is the best productivity tool I have ever used in any programming language. Once you have configured all your favorite hot keys e.g. for jumping between code editor and REPL editor using REPL to explore new APIs, try various ways to create functions etc makes programming an interesting interaction between the programmer and the code — and code evolves in an organic manner during this interaction.

I used [amazonica](https://github.com/mcohen01/amazonica) library to manipulate DynamoDB. There is going to be another interesting choice with the next Clojure 1.10 version by Cognitect: [aws-api](https://github.com/cognitect-labs/aws-api). I definitely try to use it once Clojure 1.10 is rolled out. But at least this time I was still using amazonica.

I refactored the original stateful Clojure implementation under new profile “single-node” denoting that this profile runs the server in a stateful single node. Then I created two new profiles: “local-dynamodb” and “dynamodb-dev” for local DynamoDB Docker container instance and for real AWS DynamoDB development environment. Check the details in [profiles.clj](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/profiles.clj) file.

Clojure provides nice polymorhism by [multimethods](https://clojure.org/reference/multimethods) and [protocols](https://clojure.org/reference/protocols). I used these mechanisms to abstract the database manipulation — i.e. the unit tests and the server layer has no idea whether the database queries are hitting the internal dummy database, local DynamoDB Docker container instance or the real AWS DynamoDB service. When running the unit tests you just select the right profile and the server uses the database configured for that profile (see the shell scripts I provided for all three environments).

Using amazonica is really simple. You just provide the credentials for that DynamoDB instance, table name and query parameters, example:

```clojure
(dynamodb/query (ss-aws-utils/get-dynamodb-config)  
 :table-name my-table  
 :select “ALL_ATTRIBUTES”  
 :key-conditions {:pgid {:attribute-value-list [(str pg-id)]  
 :comparison-operator “EQ”}  
 :pid {:attribute-value-list [(str p-id)]  
 :comparison-operator “EQ”}})
```

Once I had implemented all DynamoDB manipulations and had run the unit tests that everything is working properly using the local DynamoDB Docker container instance I tried the unit tests with real AWS DynamoDB profile. There was one discrepancy that I found out. I had created a secondary index for one table and local DynamoDB Docker container instance happily accepted ***:select “ALL_ATTRIBUTES”*** but real AWS DynamoDB service complained about this mistake. When changing the select clause to ***:select “ALL_PROJECTED_ATTRIBUTES”*** everything worked fine also with the real AWS DynamoDB service.

### Conclusions

Using local DynamoDB Docker container instance makes DynamoDB development really fast. Clojure is really superb for data manipulation, and using Clojure with amazonica library it is pretty effortless to work with DynamoDB.

  
