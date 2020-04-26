---
layout:	post
title:	"Azure Table Storage with Clojure"
date:	2019-01-01
---

  ![](/img/1*aQwc43xILvoetPp8SzHz_g.png)### Introduction

In my previous project “[AWS DynamoDB with Clojure](https://medium.com/@kari.marttila/aws-dynamodb-with-clojure-b4402bf8e8e)” I implemented all database handling using AWS DynamoDB for my Simple Server which I have used recently for my personal study projects. My original idea was to use this AWS Simple Server version to create AWS EKS and Fargate Kubernetes deployments for the Simple Server to refresh my AWS and Terraform skills. But then I decided to make some changes to my plans. I did my first Azure certification ([Architecting Microsoft Azure Solutions](https://www.youracclaim.com/badges/62494509-bf1c-4cd0-ad5a-0b82d1dacfac/public_url)) on December and heard from my boss that I’m going to be working in an Azure project in the beginning of year 2019. For this reason I thought that it might be a better idea to refresh my Azure skills instead and gather some experience how to create Azure infra using Terraform (I have used only ARM in the Azure side).

So, the first task in this new project was to implement the Simple Server to use some Azure database service. Since I used DynamoDB (nosql) in the AWS side the natural candidates in the Azure side were [CosmosDB](https://docs.microsoft.com/en-us/azure/cosmos-db/) and [Table Storage Service](https://azure.microsoft.com/en-us/services/storage/tables/). Since this was just an exercise I chose Table Storage Service. In my next project I’m going to use this Azure version of Simple Server to create a Terraform configuration for using Azure AKS and deploy the Simple Server to an AKS Kubernetes cluster. But let’s talk about that in more detail in my blog and in this blog post let’s focus on my experiences using Azure Table Storage Service in Clojure.

You can find the project in my [Github account](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server).

### Local Azure Table Storage Database — Azurite

As in AWS side I first googled a bit to find a local Azure Table Storage Database. And there was one — [Azurite](https://github.com/arafato/azurite). I started to use it in development until the point I just couldn’t get one query to work. I rechecked the same query using the real Azure Table Storage Service and found out that the query was working just fine but. I then stopped using Azurite and used the real Azure Table Storage Service as my development bench for the rest of the project. Hopefully Azurite will be finalized in the future — it would be a perfect tool to be used with Azure Storage development.

### Azure Table Storage Tables

For making the development smooth I implemented some scripts to create the Azure Table Storage tables, import my test data to the tables, scan tables etc. You can browse these scripts in [azure-table-storage](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server/azure-table-storage) directory.

I shamelessly re-used my old [table-importer.py](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/dynamodb/table_importer.py) script from the AWS side and converted it to use Azure Table Storage Service: [table-importer.py](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/azure-table-storage/table_importer.py). The scripts are pretty much the same using just different libraries ([boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) for AWS and [azure-cosmosdb-table](https://pypi.org/project/azure-cosmosdb-table/) for Azure). There is a [README.md](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/azure-table-storage/README.md) file in that directory which tells in more detail how to setup a development environment for creating the needed tables, how to import data etc.

So, after I had created the development bench (database) I could start the actual Clojure / Azure implementation.

### Clojure Implementation

Clojure implementation was once again really nice. Clojure and especially the Clojure REPL is the most productive software production environment I have ever used.

#### Azure Table Storage API

My first task at this point was to find an appropriate Azure Table Storage API. I googled for a while to find a native Clojure API but couldn’t find one. No problem! Since Clojure is a JVM hosted language and there is an excellent interoperability between Clojure and it’s host language Java you can use any Java API from Clojure. This project was a perfect chance to learn how to use Clojure/Java interoperability. I used [azure-storage-java](https://github.com/Azure/azure-storage-java) v. 8.0.0 as my API when manipulating Azure Table Storage. I couldn’t actually find the Table Storage part in the newest version 10 in that repo (there was an API for Blob Storage, though), but a short googling told me that there is an API for Table Storage in the previous 8.0.0 version. So I git cloned the repo, checked out the 8.0.0 version and read the samples how to use that version.

#### Clojure / Java Interop

Clojure / Java interop was mostly very smooth. I basically just read in the Java API sample how the sample used the API and aped the usage in Clojure. Example. The API sample shows how to insert an entity to the Azure Table Storage:

tableClient = TableClientProvider.getTableClientReference();  
...  
CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");  
customer1.setEmail("[walter@contoso.com](mailto:walter@contoso.com)");  
customer1.setHomePhoneNumber("425-555-0101");  
table1.execute(TableOperation.insert(customer1));In my Clojure code I create a new user using the same API as:

(let [my-env (environ/env :my-env)  
 users-table (. table-client getTableReference (str "sseks" my-env "users"))  
 new-user (new simpleserver.util.azuregenclass.users)  
 \_ (.setPartitionKey new-user email)  
 \_ (.setRowKey new-user (ss-users-common/uuid))  
 \_ (.setFirstname new-user first-name)  
 \_ (.setLastname new-user last-name)  
 \_ (.setHpwd new-user (str (hash password)))   
 table-insert (TableOperation/insert new-user)  
 ; In real production code we should check the result value, of course.  
 result (. users-table execute table-insert)]  
 {:email email, :ret :ok}))))If you can read Clojure you realize that the API calls are exactly the same.

The [simpleserver.util.azuregenclass.users](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/src/simpleserver/util/azuregenclass/users.clj) class is a so-called Clojure [gen-class](https://clojuredocs.org/clojure.core/gen-class) — a Java class defined in Clojure. You typically create these when you need to use some Java API from Clojure and the API needs some specific extended class (extended from some API base class) — the exact reason I needed to create the users gen-class:

(ns simpleserver.util.azuregenclass.users  
 (:import (com.microsoft.azure.storage.table TableServiceEntity))  
 (:gen-class  
 :extends com.microsoft.azure.storage.table.TableServiceEntity  
 :constructors {[] []}  
 :state state  
 :init init  
 :prefix "bean-"  
 :main false  
 :methods [[getLastname [] String]  
 [setLastname [String] void]  
 [getFirstname [] String]  
 [setFirstname [String] void]  
 [getHpwd [] String]  
 [setHpwd [String] void]]  
 ))(defn bean-init []  
 [[] (atom {:last-name nil, :first-name nil, :hpwd nil})])  
...#### Clojure REPL Issues with Gen-Classes

There was only one issue with my Clojure/Java interop journey — refreshing Clojure REPL with gen-classes. Since gen-classes are Clojure code and you need to compile them to Java classes there was some hassle using them with REPL. If I make changes to many Clojure namespaces it is sometimes easier to refresh the REPL (to load all namespaces) instead of loading the changed namespaces manually one by one:

(do (require ‘[clojure.tools.namespace.repl :refer [refresh]]) (refresh))The problem was that the REPL complained that it cannot find the gen-class after loading it. I understood that there is some patch to fix this issue and I up-voted the Clojure JIRA ticket — hopefully this issue will be solved in the near future. (This was not a major hindrance to my development since usually you are working with 1–2 namespaces at a time and it’s not a big deal to load those namespaces manually to REPL — and even after refreshing you can just manually import the gen-classes.)

#### Some Refactorings

While working with the new Azure version I did some refactorings. I realized that there is some code which is pretty similar in all versions — singe-node, AWS and Azure versions, only some table manipulation parts differed. So, I created those small parts as independent functions in all three versions and then moved the main processing to a common function. The three versions call the common function and inject the table functions as parameters (functions are first-class citizens in Clojure). Examples:

**get-token in Azure version:**

(defn** **get-raw-session  
 [token]  
 (let [my-env (environ/env** ***:my-env*)  
 table-filter (TableQuery/generateFilterCondition** **"PartitionKey"** **TableQuery$QueryComparisons/EQUAL token)  
 table-query (TableQuery/from** **simpleserver.util.azuregenclass.session)  
 table-query (**. **table-query where table-filter)  
 session-table (**. **table-client getTableReference (str "sseks" my-env "session"))  
 raw-sessions (**. **session-table execute table-query)]  
 (first** **raw-sessions)  
 )  
 )  
(defn **get-token**  
 [token]  
 (log/debug (str "ENTER get-token: " token))  
 (let [raw-session (get-raw-session token)]  
 (if (nil? raw-session)  
 nil  
 (. raw-session getPartitionKey))))**get-token in AWS version:**

(defn **get-token**  
 [token]  
 (let [my-env (environ/env :my-env)  
 my-table (str "sseks-" my-env "-session")  
 ret (dynamodb/query (ss-aws-utils/get-dynamodb-config)  
 :table-name my-table  
 :select "ALL\_ATTRIBUTES"  
 :key-conditions {:token {:attribute-value-list [token]  
 :comparison-operator "EQ"}})  
 items (ret :items)  
 found-token (first items)]  
 found-token))… and then in both versions I can just delegate to the common validate-token function and inject the needed functions as parameters:

(validate-token  
 [env token]  
 (log/debug (str "ENTER validate-token, token: " token))  
 (ss-session-common/validate-token token **get-token** remove-token))### Using Clojure REPL to Experiment with a Java API

This was an astonishing moment to realize: If you need to experiment with a new Java API it is actually easier to experiment with the Java API using Clojure REPL than using Java! Clojure REPL example (screenshot from my IntelliJ/Cursive REPL):

![](/img/1*3pN27eYSdrFgZd-OiRpYmA.png)So, I have loaded the simpleserver.userdb.users-table-storage namespace in my Clojure REPL and switched my REPL to use that namespace (easier to call methods without namespace prefix…).

In the first “(into [] (let…” function call I experiment what happens when I ape the functionality I found in the Java API sample in Clojure => I get a list of users classes. Ok. In the next function call I ask the first of these items and then call the getPartitionKey Java method of that Java class => I get the email which was stored as PartitionKey in Azure Table Storage field. Pretty nice to be able to dynamically experiment with a library that was written in a static language.

### Conclusions

Using Clojure/Java interop you can use any Java library if you can’t find a native Clojure library for your purposes. Using Clojure REPL it is actually even easier to experiment with a Java API that using Java itself.

  