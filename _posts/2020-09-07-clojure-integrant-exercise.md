---
layout: post
title: Clojure Integrant Exercise
category: [clojure]
tags: [clojure, languages, cursive, intellij, dynamodb, postgresql, productivity, repl, testing]
date: 2020-09-07
---


![Clojure Integrant Exercise](/img/2020-09-07-clojure-integrant-exercise_img_1.png)

*Clojure Integrant Exercise in IntelliJ IDEA / Cursive IDE.*

### Introduction

Last December when I was having a vacation before joining [Metosin](www.metosin.fi) I decided to train my Clojure skills a bit by implementing my Clojure Simple Server exercise once more. You can read more about that exercise in my previous blog post [Clojure Impressions Round Three]({% post_url 2020-01-06-clojure-impressions-round-three %}). In that blog post I stated: *"You Can Do It Without Application State Management Libraries"*. I still agree with that statement, but... When I started working at Metosin last January I talked with the Metosin guys regarding state management in Clojure applications and most of Metosin clojurists were using [Integrant](https://github.com/weavejester/integrant). I had some conversations regarding whether to create your own state management (as I did in my previous exercise) or whether to use some state management library. According to those conversations I realized that since these guys are really good clojurists and they are using a state management library there must be compelling reasons that one should use them. So, I decided to implement my Clojure Simple Server exercise once more, this time with Integrant. At the same time I did some refactorings to make the code a bit simpler, and also added a new data store: Postgres (there was a dummy CSV and DynamoDB as data store options before that). The refactoring work was also a good test bench for using Integrant and how a good state management library helps in refactoring work.

You can find the project in [Github](https://github.com/karimarttila/clojure/tree/master/webstore-demo/integrant-simple-server).

Disclaimer: This was just a quick personal exercise to learn how to use Integrant so that it makes the Clojure development with state handling smoother. The exercise is by no means a perfect example how to setup a web shop or web application - there are many peculiarities for historical reasons (e.g. domain entities are passed with simple vectors - I should have used maps instead etc.).

### Why Integrant?

Why should you use Integrant or some other state management library and not handle state yourself? Well, as I said in my previous blog post Clojure is pretty flexible and you can quite easily handle state e.g. using a simple [atom](https://clojure.org/reference/atoms). But a good state management library like Integrant provides much more. With Integrant you can have a simple edn file to manage your components and how they relate to each other to make the overall state in your application. Another powerful feature of Integrant is the capability to easily create different states for different purposes (e.g. state for production and state for running tests) and easily to reset these states. Let's unfold these capabilities in the following chapters.

### Integrant State Configuration as Edn

You can create a configuration file to provide the initial setup of your state and how it is built with different components and their interactions. This is pretty nice. Look at the [config.edn](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/dev-resources/config.edn) file. You can see that I have created three "data stores": csv, dynamodb and postgres and an Integrant configuration for each of these:

```clojure
 :backend/csv {:profile #ig/ref :backend/profile
               :active-db #ig/ref :backend/active-db
               :data-dir "dev-resources/data"}

 :backend/ddb {:active-db #ig/ref :backend/active-db
               :ss-table-prefix "ss"
               :ss-env #profile {:prod "prod"
                                 :dev "dev"
                                 :test "test"}
               :endpoint {:protocol :http :hostname "localhost" :port 8000}
               :aws-profile "local-dynamodb"}

 :backend/postgres {:active-db #ig/ref :backend/active-db
                    :adapter "postgresql"
                    :username #or [#env DB_USERNAME "simpleserver"]
                    :password #or [#env DB_PASSWORD "simpleserver"]
                    :server-name #or [#env DB_HOST "localhost"]
                    :port-number #long #or [#env DB_PORT 5532]
                    :database-name #or [#env DB_NAME "simpleserver"]
...
```

Then the service component has references to these data store components:

```clojure
 ; Gather different data store services here.
 :backend/service {:profile #ig/ref :backend/profile
                   :active-db #ig/ref :backend/active-db
                   :csv #ig/ref :backend/csv
                   :ddb #ig/ref :backend/ddb
                   :postgres #ig/ref :backend/postgres
                   }
```

I'm omitting other components, jetty, nrepl etc. It would take quite an effort to create this kind of capability yourself. Sure, if your state is very simple you don't need this capability but when your state has many components and different interactions between them this feature is pretty nice.

For reading the Integrant configuration I use [Aero](https://github.com/juxt/aero). Aero provides a nice way to inject environment variables to the values inside components and mechanisms for simple conditionals and conversions from strings to numerics (e.g. `:port-number #long #or [#env DB_PORT 5532]` - i.e. we are using environment variable DB_PORT, and if it's not found, use default value 5532, and convert the string value to long).

### Different States with Integrant

Another powerful feature of Integrant is to create different states for different purposes. E.g. in this exercise I have created two states, one for running the system in production (and in development) and one for running tests. The **production / development state** reads the [config.edn](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/dev-resources/config.edn) configuration and constructs the actual state in [core.clj](/integrant-simple-server/src/clj/simpleserver/core.clj) file:

E.g. the different data stores each have a very different state manifestation:

```clojure
(defmethod ig/init-key :backend/csv [_ {:keys [profile active-db data-dir]}]
  (log/debug "ENTER ig/init-key :backend/csv")
  ; We simulate this data store using atom.
  ; We initialize the "db" from :data-dir.
  (if (= active-db :csv)
    (let [csv-data
          {:data-dir data-dir
           :db (atom {:domain {}
                      :session #{}
                      :user {}})}]
      ; Let's keep the test database empty.
      (if (not= profile :test)
        (csv-db-loader/load-csv-db csv-data))
      (:db csv-data))))

(defmethod ig/init-key :backend/ddb [_ {:keys [active-db ss-table-prefix ss-env endpoint aws-profile]}]
  (log/debug "ENTER ig/init-key :backend/ddb")
  (if (= active-db :ddb)
    (ddb-config/get-dynamodb-config ss-table-prefix ss-env endpoint aws-profile)))

(defmethod ig/init-key :backend/postgres [_ opts]
  (log/debug "ENTER ig/init-key :backend/postgres")
  (if (= (:active-db opts) :postgres)
    {:datasource (hikari-cp/make-datasource (dissoc opts :active-db)) :active-db (:active-db opts)}))
```

The "csv" database is initialized from the csv files and then we "simulate" the database using a simple Clojure atom. When we use DynamoDB as datastore we ask `get-dynamodb-config` function to initialize the state. When we use PostgreSQL datastore we just use `hikari-pc` library to initialize the database connection.

The [test_config.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/test/clj/simpleserver/test_config.clj) file comprises the **test state** initialization. It mainly just re-uses the actual state initialization but overrides some values:

```clojure
(defn test-config []
  (let [test-port (random-port)
        ; Overriding the port with random port, see TODO below.
        _ (log/debug (str "test-config, using web-server test port: " test-port))]
    (-> (core/system-config :test)
        ;; Use the same data dir also for test system. It just initializes data.
        (assoc-in [:backend/csv :data-dir] "dev-resources/data")
        (assoc-in [:backend/jetty :port] test-port)
        ;; In Postgres test setup use simpleserver_test database.
        (assoc-in [:backend/postgres :database-name] "simpleserver_test")
        ;; No nrepl needed in tests. If used, use other port than the main system.
        (assoc-in [:backend/nrepl :bind] nil)
        (assoc-in [:backend/nrepl :port] nil)
        (assoc-in [:backend/csv :port] nil))))
```

E.g. we use a different port for Jetty so that we can run Jetty for the development and tests at the same time and the ports won't clash. For tests we also use a different PostgreSQL database.

### Resetting the State

Integrant provides nice auxiliary functions to reset the states: go, reset, halt etc. - check the meaning of these functions in the [Integrant Repl documentation](https://github.com/weavejester/integrant-repl). I have created [Cursive](https://cursive-ide.com/) hot keys for the most used auxiliary functions, e.g. `(integrant.repl/reset)` is `alt-J`. So, when I do any changes in any namespace and hit `alt-J` Integrant takes care of reloading the affected namespaces and then resets the state. This happens in a blink of an eye so I'm hitting `alt-J` pretty often when doing Clojure. 

If I have the development state running I can query its state:

```clojure
(user/env)
=>
{:profile :dev,
 :active-db :postgres,
 :service {:domain #simpleserver.service.domain.domain_postgres.PostgresR{:db {:datasource #object[com.zaxxer.hikari.HikariDataSource
                          0x42ab0edd
                          "HikariDataSource (HikariPool-33)"],
...
```

The tests start the test state, but I can also start it manually and then query the test state:

```clojure
(simpleserver.test-config/go)
(simpleserver.test-config/test-env)
=>
{:profile :test,
 :active-db :postgres,
 :service {:domain #simpleserver.service.domain.domain_postgres.PostgresR{:db {:datasource #object[com.zaxxer.hikari.HikariDataSource
                          0x780ac829
                          "HikariDataSource (HikariPool-35)"],
...
```

As you can see the `HikariDataSource` is a different object in those states.

This is pretty nice. I usually experiment different stuff in my scratch file hitting the development state. When I'm happy I move the code from the scratch file to the production code. If I have some issues with some test, I can just manually start the test configuration and send the forms from the test code to be run in the REPL and the forms hit the test state.

### Running Tests

I have [Just](https://github.com/casey/just) recipes both for DynamoDB and Postgres databases. Let's start both of them and then run tests...

```bash
λ> just
Available recipes:
    backend      # Start backend repl.
    backend-kari # Start backend repl with my toolbox.
    dynamodb     # Start local dynamodb emulator
    lint         # Lint
    list
    postgres     # Start local postgres
    test db      # Test

# First Postgres...
λ> just postgres
NOTE: Remember to destroy the container if running again!
Starting docker compose...
Creating ss-postgres_postgres_1 ... done
Creating Simple Server schemas...
...

# ...and then DynamoDB...
λ> just dynamodb
Sending build context to Docker daemon  62.11MB
Step 1/15 : FROM python:3.8.2-slim-buster
 ---> e7d894e42148
Step 2/15 : RUN rm /bin/sh && ln -s /bin/bash /bin/sh
 ---> Using cache
...
Successfully tagged ss-uploader:0.1
Creating ss-dynamodb_local-dynamodb_1 ... done
Creating ss-dynamodb_uploader-app_1   ... done
Attaching to ss-dynamodb_local-dynamodb_1, ss-dynamodb_uploader-app_1
...
```

If I want to run the tests in IntelliJ IDEA / Cursive I can easily set the active data store in the [config.edn](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/dev-resources/config.edn):

```clojure
 ; csv, ddb, postgres
 ;:backend/active-db #or [#env SS_DB :csv]
 ;:backend/active-db #or [#env SS_DB :ddb]
 :backend/active-db #or [#env SS_DB :postgres]
...
```

... and then reset the Integrant state and run the tests in IntelliJ IDEA / Cursive:

![Running tests](/img/2020-09-07-clojure-integrant-exercise_img_2.png)

*Running tests in Clojure Integrant Exercise in IntelliJ IDEA / Cursive IDE.*

I have in the configuration that if my SimpleServer DB flag (`SS_DB`) is present then use it. So, as the Just suggested above I can also run the test suites in command-line, providing the data store with the command, [Justfile](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/Justfile):

```bash
# Test
@test db:
    ./run-tests.sh {{db}}
```

... and [run-tests.sh](https://github.com/karimarttila/clojure/blob/master/webstore-demo/integrant-simple-server/run-tests.sh) helper which populates the SS_DB flag according what was used in Just command:

```bash
...
MYDB=$1
if [[ "$MYDB" =~ ^(csv|ddb|postgres)$ ]]; then
    echo "Starting tests with $MYDB configuration..."
else
    echo "Unknown DB configuration: $MYDB, exiting..."
    exit 2
fi
SS_DB=$MYDB clojure -A:dev:test:common:backend -m kaocha.runner
```

Let's run the tests:

```bash
# First using DynamoDB as data store...
λ> just test ddb
Starting tests with ddb configuration...
[(........)(...........................)(...)(.......)]
14 tests, 45 assertions, 0 failures.
# ...and then Postgres:
λ> just test postgres
Starting tests with postgres configuration...
[(...)(...........................)(.......)(........)]
14 tests, 45 assertions, 0 failures.
```

... and Integrant takes care of configuring the test state so that the application uses the right data store.


### Conclusions

I'm happy I did this Integrant exercise. I now realize the benefits using a high-quality state management library versus implementing state management yourself. You can do state management quite easily yourself as well since Clojure as a Lisp is really powerful and flexible. But using a high-quality state management library gives you the basic plumming for free so that you don't need to worry about that but just write the state as a configuration and use different states for different purposes during development.


*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>

