---
layout: post
title: Clojure Power Tools Part 2
category: [clojure]
tags: [clojure, repl, productivity, languages, cursive, intellij, programming]
date: 2020-10-29
---

![IntelliJ IDEA and Cursive](/img/2020-10-29-clojure-power-tools-part-2_img_1.png)


### Introduction

This is the second part of my Clojure Power Tools series (I'm a bit interested myself how many blog posts I will write to this series). If you haven't read the first part I recommend you to read it first: [Clojure Power Tools Part 1]({% post_url 2020-10-26-clojure-power-tools-part-1 %}). In this second blog article, I list a couple of new power tool tricks for debugging. I first introduce a poor man's debug repl, and then the real debug repl.

### Poor Man's Debug Repl

I was refactoring some Clojure tests for the second version of a complicated application. There were quite a few structural changes in the application logic, and the domain was rather complicated comprising a lot of various recursive data structures. In some tests, I was quite puzzled about what kind of state there was in several places of the application. The tests were also rather complicated and I wanted a simple way to record certain bindings in several places of the application when I was running a certain area of the test - and only that area. I quickly implemented a poor man's debug recorder:

```clojure
(ns recorder)

(def value (atom {}))
(def counter (atom 0))
(def flag (atom false))

(defn next-id []
  (swap! counter inc))

(defn assoc-value-with-id! [k v]
  (let [id (next-id)
        new-k (keyword (str (name k) "-" id))]
    (swap! value assoc new-k v)))

(defn add-value-if-recording! [k v]
  (when @flag
    (assoc-value-with-id! k v)))

(defn start! []
  (reset! value {})
  (reset! counter 0)
  (reset! flag true))

(defn stop! []
  (reset! flag false))
```

The idea is to provide a way to add stuff to the value map with unique keys - but only when I'm recording.

(I'm using my own exercises as an example here, the data here is ridiculously simple - no need for a recorder here, but I guess you get my point if you change the data to several hundreds of lines of recursive maps and vectors and complex business logic...)

So, let's add the start/stop recording to the test we are interested:

```clojure
(deftest product-groups-test
  (log/debug "ENTER product-groups-test")
  (testing "GET: /api/product-groups"
    (let [_ (re/start!)
          login-ret (ss-tc/-call-api :post "login" nil {:email "test-kari.karttinen@foo.com" :password "Kari"})
          _ (log/debug (str "Got login-ret: " login-ret))
          json-web-token (get-in login-ret [:body :json-web-token])
          params (-create-basic-authentication json-web-token)
          get-ret (ss-tc/-call-api :get "/product-groups" params nil)
          status (:status get-ret)
          body (:body get-ret)
          right-body {:ret "ok", :product-groups {:1 "Test-Books", :2 "Test-Movies"}}
          _ (re/stop!)]
      (is (= (not (nil? json-web-token)) true))
      (is (= status 200))
      (is (= body right-body)))))
```

See the `(re/start!)` and `(re/stop!)` commands in the test - we are recording only during that time.

Then I can add stuff to my recorder where-ever I want:

```clojure
(defn -valid-token?
  "Parses the token from the http authorization header and asks session ns to validate the token."
  [env req]
  (log/debug "ENTER -valid-token?")
  (let [basic (get-in req [:headers "authorization"])
        _ (re/add-value-if-recording! :valid-token-basic basic)
        basic-str (and basic (last (re-find #"^Basic (.*)$" basic)))
...
```

I.e. `(re/add-value-if-recording! :valid-token-basic basic)`.

Run the tests.

After the tests I can examine the recorder in my scratch file:

```clojure
  (require '[recorder])
  (keys @recorder/value)
  @recorder/value
  (require '[portal.api :as portal-api])
  (portal.api/clear)
  (portal.api/open)
  (portal.api/tap)
  (tap> @recorder/value)
```

You can also use the excellent [portal](https://github.com/djblue/portal) visualization tool to examine your complicated data you recorded (well, in this example, not so complicated):

![Panel](/img/2020-10-29-clojure-power-tools-part-2_img_2.png)

### Real Debug-Repl

The example above was a simple trick but a real Clojurian *stops the world* if he is interested to see what is happening in a particular moment of time instead of just recording it. We are going to use Gary Fredericks's [debug-repl](https://github.com/gfredericks/debug-repl) library for it. You need the dependency - I have these tool dependencies in my `~/.clojure/deps.edn` file:

```clojure
{
 :aliases {
           :kari {:extra-paths ["scratch"]
                  :extra-deps {hashp/hashp {:mvn/version "0.1.1"}
                               com.gfredericks/debug-repl {:mvn/version "0.0.11"}
                               djblue/portal {:mvn/version "0.6.1"}
                               }
                  }
           }
 }
```

Then we need to be able to start the backend REPL with this debug repl middleware, I have this in my [Justfile](https://github.com/casey/just) as a Just recipe which I use to start my backend repl with various options:

```bash
# Start backend repl with my toolbox and with debug-repl capability.
@backend-debug-kari:
    clj -M:dev:test:common:backend:kari -m nrepl.cmdline -m com.gfredericks.debug-repl/wrap-debug-repl -i -C
```

Then you can add a debug-repl breakpoint in your code, run your code, stop the world and examine the snapshot context of your world just before the breakpoint. Let's see this in action:

![debug-repl](/img/2020-10-29-clojure-power-tools-part-2_img_3.png)

I have added `_ (break! "Yihaa!")` breakpoint in one of the tests. Then I run the test and the world stops at the breakpoint (`Hijacking repl for breakpoint: Yihaa!` output in the REPL output window). Then I have moved the cursor in various let-bindings: `json-web-token`, `params` and `get-ret` and evaluated the forms: you can see the evaluated values in the REPL output window on the right. If I try to evaluate `status` I get an error "Unable to resolve symbol: status in this context" - of course, because it's outside the context.


### Conclusions

There are a lot of other Clojure tricks and tools - maybe I'll write "Clojure Power Tools Part 3" blog post in the future.


*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
