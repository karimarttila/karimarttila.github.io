---
layout: post
title: Using Clojure in Command Line with Babashka
category: [clojure]
tags: [clojure, languages, cursive, intellij, repl]
date: 2020-09-01
---

![Ordinary Clojure code](/img/2020-09-01-using-clojure-in-command-line-with-babashka_img_1.png)

*Ordinary Clojure code - can be run in REPL or in command line with Babashka (the whole file can be found [here](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/postgres/bb_postgres.clj) )*


### Introduction

I never bothered to learn [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) so that I could be really fluent with it. If I needed anything beyond basic Bash stuff I immediately used [Python](https://www.python.org/) in command-line scripting.

I'm currently implementing my Clojure simple server again, this time using the [Integrant](https://github.com/weavejester/integrant) library. In this new version, I implemented three data stores: CSV, AWS DynamoDB, and Postgres. I had already implemented importing development data into DynamoDB (using Python), this time I used [Babashka](https://github.com/borkdude/Babashka) to import development data into Postgres - mainly just to have an excuse to try if I could replace Python with Clojure when scripting something with Bash.

The scripts can be found in my [Clojure git](https://github.com/karimarttila/clojure) repo in directory [postgres](https://github.com/karimarttila/clojure/tree/master/webstore-demo/integrant-simple-server/postgres).


### Developing with Babashka

The really neat thing with Babashka is that you can develop your Babashka scripts as part of your Clojure project, or independently but using your favorite Clojure IDE. The picture above shows Clojure code in my favorite Clojure IDE, [Cursive](https://cursive-ide.com/). I have the Clojure code that imports data into the Postgres database in directory [postgres](https://github.com/karimarttila/clojure/tree/master/webstore-demo/integrant-simple-server/postgres), so I have the following extra path in my [deps.edn](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/deps.edn):

```clojure
 :postgres {:extra-paths ["postgres"]}
```

Then the nice thing is that I can develop the Clojure code as part of project's other Clojure code. Let's first create a short bash script that tells Babashka to run your Clojure code with a flag so that we know in the Clojure code when we are running the code using Babashka or using Clojure IDE REPL (file [run-bb-load-data.sh](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/postgres/run-bb-load-data.sh)):

```bash
#/bin/bash

export POSTGRES_PASSWORD=simpleserver
export RUNNING_BB=TRUE
bb bb_postgres.clj
```

The flag is the `RUNNING_BB` export above.

Then in the Clojure code we have a top level form `(run-me)` in the namespace (file [bb_postgres.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/postgres/bb_postgres.clj)):

```clojure
(defn run-me []
  "Loads data only if running from Babashka script which sets the environment variable.
  We don't want the repl to load the data when reloading the namespace.
  In repl experimentation use the rich comment below."
  (let [running-bb? (System/getenv "RUNNING_BB")]
    (if (= running-bb? "TRUE")
      (import-data))))

(run-me)
```

I.e. when reloading the namespace REPL runs the code but if the flag is not set it doesn't actually do anything - it imports the code only if we are running the code using Babashka. The reason for this is that when I reload the namespace as part of my Clojure workflow I don't want the data import to happen. For development purposes to test importing data, or any other function, I have a `rich comment` at the end of the file:

```clojure
(comment
  (def data-dir "dev-resources/data")
  (get-raw-products data-dir 2)
  (get-product-groups data-dir)
  (do
    (delete-products!)
    (delete-product-groups!))
  (vals (get-users data-dir))
  (load-users (vals (get-users data-dir)))
  (import-data)
  (db-get-all-product-groups)
  (db-get-all-products)
  )
```

The `comment` block is here so that REPL does not run this code when reloading, of course. The function calls you see inside the comment block are just experiments added in no particular order. I can send any of these forms individually to be evaluated in the REPL - a typical Clojure trick when developing with REPL.


### Babashka Use Cases

I really like the idea that I can now use Clojure in shell scripting. Of course I could use Clojure in shell scripting also without Babashka but JVM boot takes quite a long time which makes testing of the script in command line a bit painful. Not so with Babashka - Babashka boots lightning fast:

```bash
λ> time bb '(println "Hello world!")'
Hello world!

real	0m0.006s
user	0m0.003s
sys	0m0.003s
```

The use cases using Babashka in my personal scripting probably is a bit like I used Babashka to import data into the Postgres database in this exercise:

```clojure
(defn run-sql [command]
  (sh/sh "psql" "--host" "localhost" "--port" "5532" "--username=simpleserver" "--dbname=simpleserver" "-c" command))

(defn insert-product-group! [product-group]
  (println "Inserting product-group: " product-group)
  (let [[id name] product-group
        command (str "INSERT INTO product_group VALUES ('" id "', '" name "');")]
    (run-sql command)))

(defn load-product-groups [product-groups]
  (doseq [pg product-groups]
    (insert-product-group! pg)))

(defn get-product-groups [data-dir]
  (let [raw (with-open [reader (io/reader (str data-dir "/product-groups.csv"))]
              (doall
                (csv/read-csv reader)))
        product-groups (into {}
                             (map
                               (fn [[item]]
                                 (str/split item #"\t"))
                               raw))]
    product-groups))

(defn import-data []
  (let [data-dir "dev-resources/data"
        product-groups (get-product-groups data-dir)]
;...
    (load-product-groups product-groups)
;...
```

I.e. parsing CSV, transforming data, and then call some program with the transformed data, possibly read what was returned and do other stuff. You could possibly do all this using plain old Bash but I never bothered to learn Bash in that level that I can do more than test some flags and call other programs using Bash.

### How Do I Use the Babashka Script in This Exercise?

I used Babashka to load development data into the Postgres data store. During development I built a custom Postgres image and provided a [Just](https://github.com/casey/just) recipe to start the data store (file [Justfile](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/Justfile)):

```bash
# Start local postgres
@postgres:
    cd postgres && ./run-docker-compose.sh
```

The [run-docker-compose.sh](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/postgres/run-docker-compose.sh) starts the Postgres docker container, creates the schema and finally calls [./run-bb-load-data.sh](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/postgres/run-bb-load-data.sh) which loads the data into the development Postgres data store:

```bash
#!/usr/bin/env bash

echo "NOTE: Remember to destroy the container if running again!"
echo "Starting docker compose..."
docker-compose -p ss-postgres -f docker-compose-setup-local-postgres.yml up -d
sleep 5
echo "Creating Simple Server schemas..."
./create-schema.sh
sleep 1
echo "Loading data..."
./run-bb-load-data.sh
sleep 1
docker logs -f ss-postgres_postgres_1
```

### Clojure (Babashka) vs Python in Shell Scripting

Let's finally compare Python and Clojure (Babashka) when doing some Linux shell scripting.

**Easiness**. Both languages are pretty easy and fast to work if you have used them. Developing Python scripts is pretty fast - you just run the script in command line. Working with Clojure has one additional plus: you can use the Clojure REPL. 

**Library support**: Python wins. When you are scripting in Python and you realize that it would be nice e.g. to use some AWS library - just use it (e.g. [table_importer.py](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/dynamodb/pysrc/table_importer.py) - the AWS [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) library). The library support for Babashka is not as extensive, of course - but Babashka supports quite many namespaces outside `clojure.core` and also some additional libraries: [Babashka built-in namespaces](https://github.com/borkdude/babashka#built-in-namespaces) - keep eye on that page, maybe Babashka library support is growing in the future!

So, the library support might not be as good as with Python. But I really do love Clojure and if I'm implementing apps using Clojure it is really nice to do some ad hoc scripting using Babashka.

### Conclusions

It's nice to have another scripting tool in my toolbox: [Babashka](https://github.com/borkdude/babashka). Time will tell if I start using Clojure instead of Python as my preferred scripting language, thanks to Babashka. At least in this exercise Babashka did really well.


*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
