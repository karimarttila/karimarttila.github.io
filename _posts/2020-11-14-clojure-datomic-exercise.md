---
layout: post
title: Clojure Datomic Exercise
category: [clojure]
tags: [clojure, datomic, repl, database, datalog]
date: 2020-11-14
---

![IntelliJ IDEA and Cursive](/img/2020-11-14-clojure-datomic-exercise_img_1.png)

*Clojure Datomic Exercise in IntelliJ IDEA / Cursive IDE.*

### Introduction

In my previous [Integrant exercise]({% post_url 2020-09-07-clojure-integrant-exercise %}), I had converted my earlier SimpleServer exercise to use the [Integrant](https://github.com/weavejester/integrant) state management library. In that Integrant exercise, there were three datastores: in-memory datastore that read the initial data from CSV files, [AWS DynamoDB](https://aws.amazon.com/dynamodb/) datastore, and [PostgreSQL](https://www.postgresql.org/) datastore. I implemented the domain layer using [Clojure Protocols](https://clojure.org/reference/protocols) so that in the application it was easy to switch the datastore by changing the value in one Integrant configuration (`:backend/active-db `) - and reset the application state by using [Integrant reset](https://github.com/weavejester/integrant-repl).

My next exercise was to implement a frontend to that application using [re-frame](https://github.com/day8/re-frame), read more about that exercise in my blog article [Clojure Re-Frame Exercise]({% post_url 2020-10-15-clojure-re-frame-exercise %}). Since this new Datomic exercise is basically the same exercise as the previous re-frame exercise except the new Datomic datastore added, I created this exercise into the same re-frame application directory.

So, this Datomic exercise can be found in my [Clojure](https://github.com/karimarttila/clojure) repo in directory [re-frame](https://github.com/karimarttila/clojure/tree/master/webstore-demo/re-frame-demo).


### What is Datomic?

[Datomic](https://www.datomic.com/) is an immutable transactional hosted database that uses [Datalog](https://en.wikipedia.org/wiki/Datalog) language for queries. Being hosted means that Datomic uses another data store service, e.g. DynamoDB, for persistence.

If you are interested to learn more about Datomic I list below some Datomic resources I used myself while implementing this exercise:

- [Datomic Home Page](https://www.datomic.com/)
- [Datomic Documentation](https://docs.datomic.com/cloud/index.html)
- [Datomic Dev-Local](https://docs.datomic.com/cloud/dev-local.html)
- [Cognitect dev-tools download](https://cognitect.com/dev-tools/index.html)
- [Day of Datomic 2016 Video Series](https://docs.datomic.com/on-prem/day-of-datomic.html)
- [Day of Datomic 2016 Github Repo](https://github.com/Datomic/day-of-datomic)
- [Learn Datalog Today](http://www.learndatalogtoday.org/)
- [Migration from Postgres to Datomic](https://grishaev.me/en/pg-to-datomic/)
- [Local Dev Setup](https://docs.datomic.com/on-prem/dev-setup.html)

I strongly recommend to read [Learn Datalog Today](http://www.learndatalogtoday.org/) and do all exercises - a great resource to learn the [Datalog](https://docs.datomic.com/on-prem/query.html) language. If you are interested in watching the Day of Datomic videos there are newer videos from 2018 in Youtube. Stuart Halloway is an excellent presenter and it was a joy to watch those video presentations.

### Choosing Datomic Version

Choosing the Datomic version for development was a bit confusing: should I choose Datomic dev-local, Datomic Free, or Datomic Starter? Finally, I decided to use [Datomic Starter](https://www.datomic.com/get-datomic.html).

I used the following instructions to set up the local Datomic Starter instance:

- [Local Dev Setup](https://docs.datomic.com/on-prem/dev-setup.html)
- [Get Datomic](https://docs.datomic.com/on-prem/get-datomic.html)

I must say that choosing the right Datomic version was a bit confusing - not to talk about the documentation. There really should be one dedicated "start here" page which could provide easy to follow instructions for a complete Datomic newcomer.

Once you have unzipped the Datomic server installation package, configure the `dev-transactor-template.properties` as instructed in the Datomic documentation and then run the `bin/maven-install` to install the library jar. Add the dependency to your project's `deps.edn` file.


### Datomic Tooling

I created some [Just recipes](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/Justfile) for Datomic development:

```bash
# Start local Datomic database.
@datomic:
    cd datomic && ./start-datomic.sh

# Reset Datomic SimpleServer databases.
@datomic-ss-reset:
    cd datomic && ./reset-simpleserver-db.sh

# Start Datomic Peer Server. NOTE: NOT NEEDED IN THIS EXERCISE!
@datomic-peer-server:
    cd datomic && ./start-peer-server.sh

# Start Datomic Console.
@datomic-console:
    cd datomic && ./start-console.sh
```

Short descriptions of the Just recipes:

- datomic: Starts the Datomic database (transactor etc.).
- datomic-ss-reset: Deletes and creates simpleserver and simpleserver_test databases (for clean development start).
- datomic-peer-server: Starts the Datomic Peer Server. Not needed in this exercise, I just tested that it works.
- datomic-console: Starts the Datomic Console server, open Chrome: `http://localhost:8080/browse`

The `datomic-ss-reset` is a bit of a hack, calls `reset-simpleserver-db.sh`:

```bash
#!/bin/bash
THIS_DIR=$(pwd)
TRANSACTOR_DIR=/mnt/ssd2/local/datomic-pro-1.0.6202
pushd $THIS_DIR
cd $TRANSACTOR_DIR
pwd
sleep 2
bin/repl < $THIS_DIR/reset-simpleserver-db.clj
popd
```

So, it starts the `repl` which is provided by the Datomic server. For some reason, there is no `deps.edn` for this repl but a cryptic shell script which populates the jar files, the reason I start it like this. I pipe the required function calls to this repl, see `reset-simpleserver-db.clj`:

```clojure
(require '[datomic.api :as d])
(println "**********************************")
(println "Starting to reset Simpleserver databases...")
(def db-uri "datomic:dev://localhost:4334/simpleserver")
(def test-db-uri "datomic:dev://localhost:4334/simpleserver_test")
(println "Listing databases before reset...")
(d/get-database-names "datomic:dev://localhost:4334/*")
(println "Deleting databases...")
(d/delete-database db-uri)
(d/delete-database test-db-uri)
(println "Listing databases after deletes...")
(d/get-database-names "datomic:dev://localhost:4334/*")
(println "Creating databases...")
(d/create-database db-uri)
(d/create-database test-db-uri)
(println "Listing databases after creations...")
(d/get-database-names "datomic:dev://localhost:4334/*")
(def exercise-dir "/a/prs/github/clojure/webstore-demo/re-frame-demo")
(def ss-schema (read-string (slurp (str exercise-dir "/datomic/simpleserver-schema.edn"))))
(def ss-conn (d/connect db-uri))
(def test-ss-conn (d/connect test-db-uri))
@(d/transact ss-conn ss-schema)
@(d/transact test-ss-conn ss-schema)
...
(def product-groups (load-product-groups))
(def product-group-datoms (get-product-group-datoms product-groups))

@(d/transact ss-conn product-group-datoms)
...
```

An easy way to create the development and test databases and transact the schema to the databases, and then transact the development data to the database. 

Now that we have some real data in the Datomic database let's look at the data using the Datomic Console we started earlier with the Just recipe (open Chrome and navigate to http://localhost:8080/browse):

![Datomic Console](/img/2020-11-14-clojure-datomic-exercise_img_2.png)

*Datomic Console.*

I ran the same query I can also run quite easily with the Clojure REPL:

```clojure
  (d/q '[:find ?title ?a_or_d ?year
         :in $ ?pg-id ?y
         :where
         [?e :domain.product/title ?title]
         [?e :domain.product/year ?year]
         [?e :domain.product/a_or_d ?a_or_d]
         [?e :domain.product/pg-id ?pg-id]
          [(>= ?year ?y)]]
       (d/db ss-conn) 2 2000)
```

I.e. Find all products that belong to the product group `2` and are created after year `2000` (including): print the title, author/director and year. In REPL you get:

```clojure
#{["A History of Violence" "Cronenberg, David" 2005]
  ["Mulholland Dr." "Lynch, David" 2001]
  ["Paha maa" "Aku Louhimies" 2005]
  ["Avatar" "Cameron, James" 2009]
  ["Hyvä poika" "Zaida Bergroth" 2011]
...
```

I didn't use the Datomic Console at all during the exercise. It was a lot easier to develop and test the queries interactively using the Clojure REPL.

### Experimenting with the Datomic Sample

There are quite a lot of samples with the Datomic Starter installation package. Unfortunately, there was no standard `deps.edn` file with the samples. Therefore I added [nrepl](https://github.com/nrepl/nrepl) to the run script to be able to start a nrepl server and then connect to that server using IntelliJ IDEA / Cursive to make it easier to experiment e.g. with the Seattle sample that was provided with the installation package.

I also threw the Seattle sample data to the Datomic dev server instead of the in-memory server.

### Setting Up the Connection to the Datomic Database

The connection to the Datomic database is created in the Integrant state management of the application:

Integrant configuration in [config.edn](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/resources/config.edn):

```clojure
 :backend/datomic {:active-db #ig/ref :backend/active-db
                   :uri "datomic:dev://localhost:4334/simpleserver"}
```

And the state set up in [core.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/src/clj/simpleserver/core.clj):

```clojure
(ns simpleserver.core
  (:require
...
    [datomic.api :as d]
...

(defmethod ig/init-key :backend/datomic [_ {:keys [active-db uri]}]
  (log/debug "ENTER ig/init-key :backend/datomic")
  (if (= active-db :datomic)
    {:conn (d/connect uri)}))
```

### Using the Datomic Api

Using the Datomic api was really easy and experimenting with the api using the Clojure REPL was a real joy. Some examples.

**Transact the schema** to the Datomic database ([reset-simpleserver-db.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/datomic/reset-simpleserver-db.clj)):

```clojure
(def db-uri "datomic:dev://localhost:4334/simpleserver")
(d/create-database db-uri)
(def exercise-dir "/a/prs/github/clojure/webstore-demo/re-frame-demo")
(def ss-schema (read-string (slurp (str exercise-dir "/datomic/simpleserver-schema.edn"))))
(def ss-conn (d/connect db-uri))
@(d/transact ss-conn ss-schema)
```

The schema consists of the same datoms that you use also in ordinary queries ([simpleserver-schema.edn](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/datomic/simpleserver-schema.edn)):

```clojure
[
 ; DOMAIN
 ; Product group
 {:db/ident :domain.product-group/id
  :db/valueType :db.type/long
  :db/cardinality :db.cardinality/one
  :db/unique :db.unique/identity
  :db/doc "The id of the product group"}

 {:db/ident :domain.product-group/name
  :db/valueType :db.type/string
  :db/cardinality :db.cardinality/one
  :db/doc "The name of the product group"}

 ; Product
 {:db/ident :domain.product/id
  :db/valueType :db.type/long
  :db/cardinality :db.cardinality/one
  :db/unique :db.unique/identity
  :db/doc "The id of the product"}
...
```

**Transact data** to the Datomic database ([simpleserver-schema.edn](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/datomic/simpleserver-schema.edn)):

```clojure
...
(defn get-product-group-datoms [product-groups]
    (mapv (fn [product-group]
            (let [id (:id product-group)
                  name (:name product-group)]
              {:domain.product-group/id id
               :domain.product-group/name name}))
          product-groups))

(def product-groups (load-product-groups))
(def product-group-datoms (get-product-group-datoms product-groups))

@(d/transact ss-conn product-group-datoms)
...
```

**Query datoms** from the Datomic database ([user-datomic.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/src/clj/simpleserver/service/user/user_datomic.clj)):

```clojure
(credentials-ok?
    [_ _ email password]
    (log/debug (str "ENTER credentials-ok?"))
    (let [found (d/q '[:find ?email ?hashed-password
                       :in $ ?email ?hashed-password
                       :where
                       [?e :user.user/email ?email]
                       [?e :user.user/hashed-password ?hashed-password]]
                     (d/db conn) email (str (hash password)))]
      (if found
        (= found #{[email (str (hash password))]})
        false)))
```


### Development Flow

The development flow using the Clojure REPL and Integrant was really nice. Usually, I experimented the queries in my scratch file (see more about this technique in my blog post [Clojure Power Tools Part 1]({% post_url 2020-10-26-clojure-power-tools-part-1 %})). Once I was confident the query worked I moved the code snippet from the scratch file to the production source file and reset Integrant state (with IntelliJ IDEA hotkey, of course), and ran the unit tests to see that the code worked also as part of the application.

### Conclusions

It was really fun to do this exercise. The Day of Datomic video presentations were really good, also other material regarding Datomic and the Datalog query language. The exercise itself was quite effortless to implement using Datomic.

I would definitely use Datomic with Clojure in real projects, too. The idea of an immutable database is a real nice fit for many use cases.

*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>

