---
layout: post
title: Malli Clojure Library
category: [clojure]
tags: [clojure, repl, malli, schema, spec, data, validation]
date: 2020-12-06
---

![IntelliJ IDEA and Cursive](/img/2020-12-06-malli-clojure-library_img_1.png)

*Using `malli` library to validate POST body data in the REST API.*

### Introduction

At [Metosin](https://www.metosin.fi/en/) we use our own libraries quite a lot in our projects - therefore a good reason to learn to use these libraries. For learning to use the [malli](https://github.com/metosin/malli) library I experimented with it using the Clojure REPL, and I also added malli validation to my previous `re-frame` exercise that can be found in my [Clojure](https://github.com/karimarttila/clojure) repo, in directory [re-frame](https://github.com/karimarttila/clojure/tree/master/webstore-demo/re-frame-demo).

I did most of the experimentation in my REPL scratch file (not included in the repo). I then wrote the observations regarding this learning project in this blog post mostly for myself - verbalizing what one has discovered is a good learning method.

### What is Malli?

[malli](https://github.com/metosin/malli) is a Clojure/Script library for data validation and specification. It provides tools for validation, coercion and other aspects related to the question whether the data in hand is valid for your purposes. `malli` also integrates nicely with the [reitit](https://github.com/metosin/reitit) routing library providing nice tools to validate and coerce data coming to the REST API.

If you are interested to learn more about `malli` I list below some resources I used myself while experimenting with `malli`:

- [malli Github repository](https://github.com/metosin/malli)
- [Tommi Reiman's blog post regarding malli](https://www.metosin.fi/blog/malli/)
- [Data-driven Rapid Application Development with Malli](https://www.youtube.com/watch?v=ww9yR_rbgQs) in Youtube
- [Inside Data-driven Schemas presentation](https://www.youtube.com/watch?v=MR83MhWQ61E) in Youtube
- [malli.io playground](https://malli.io)

My colleague [Miikka Koskinen](https://miikka.me/) has written a couple of excellent blog posts regarding `malli` and other data validation libraries:
- [Schema, Spec, and Malli](https://quanttype.net/posts/2020-05-03-schema-spec-and-malli.html)
- [Essential features of data specification libraries](https://quanttype.net/posts/2020-05-10-essential-features.html)

### Why Data Validation?

Clojure is a dynamic language and there is no static type checking by the Clojure compiler. Some developers see this as a downside but it actually makes Clojure both simple and also very expressive and powerful regarding how to manipulate data. One could say that in Clojure we don't have types - we have something better: data. You can do a lot more with data than with types. And if you are interested whether your data conforms to a certain data shape - you have validation.

### Malli, Spec or Schema?

There are three major data validation libraries in the Clojure ecosystem:

- [schema](https://github.com/plumatic/schema): older data validation library which was the de facto schema library before `spec`.
- [spec](https://clojure.org/about/spec): the data validation library that comes with Clojure 1.9 or higher and is also used to describe the Clojure core language and Clojure library apis.
- [malli](https://github.com/metosin/malli): the data validation library provided by [Metosin](https://www.metosin.fi/en/).

I tried to verbalize to myself what is the purpose and strengths of these libraries, their sweet spots in projects and how they differ from each other. I wrote about this blog post in the Metosin Slack and one of my colleagues hinted that our colleague Miikka Koskinen already has written a blog post that does a very good job describing these libraries, I really recommend reading it: [Schema, Spec, and Malli](https://quanttype.net/posts/2020-05-03-schema-spec-and-malli.html). So, I'm not going describe the differences between those data validation libraries - Miikka has already done a much better job describing them than I ever could have done.

### Experimenting with Malli Using the Clojure REPL

It is quite nice and effortless to experiment with the Malli library using a clojure REPL: just add `metosin/malli {:mvn/version "0.2.1"}` (0.2.1 is the latest version as of writing this blog post - check for newer versions in [malli github repo](https://github.com/metosin/malli) ) to your `deps.edn` file and then require `malli` and start experimenting with it:

```clojure
(ns malli
  (:require [malli.core :as m]))

(m/validate
  [:map {:closed true} [:ping string?]]
  {:ping "hello"}) ; => true

(m/validate
  [:map {:closed true} [:ping string?]]
  {:wrong-key "hello"}) ; => false

(m/validate
  [:map {:closed true} [:ping string?]]
  {:ping "hello" :extra-key "hello"}) ; => false
```

A common workflow in the Clojure land is to use a scratch REPL file to experiment with the data validation and once you are confident your validation works as it should you just move the validation code to the production code base.

### Malli with Reitit

As an exercise I implemented `malli` validations to the REST api of the SimpleServer. In this chapter I provide a few examples. The REST api can be found in the [server.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/src/clj/simpleserver/webserver/server.clj) namespace.

Let's first use the `ping` api as an example.

```clojure
   ["/api"
    {:swagger {:tags ["api"]}}
    ; For development purposes. Try curl http://localhost:6161/api/ping
    ["/ping" {:get {:summary "ping get"
                    ; Don't allow any query parameters.
                    :parameters {:query [:map]}
                    :responses {200 {:description "Ping success"}}
                    :handler (fn [_]
                               (-> {:ret :ok :reply "pong"}
                                   (ri-resp/response)
                                   (-set-http-status :ok)))}
              :post {:summary "ping post"
                     :responses {200 {:description "Ping success"}}
                     ;; reitit adds mt/strip-extra-keys-transformer - probably changes in reitit 1.0,
                     ;; and therefore {:closed true} is not used with reitit < 1.0.
                     :parameters {:body [:map {:closed true} [:ping string?]]}
                     :handler (fn [req]
                                (let [body (get-in req [:parameters :body])
                                      myreq (:ping body)]
                                  (-> {:ret :ok :request myreq :reply "pong"}
                                      (ri-resp/response)
                                      (-set-http-status :ok))))}}]
...
```

For `ping` GET I define that we don't allow any query parameters, the query map is empty: `:parameters {:query [:map]}`. For `ping` POST we define the only allowed body parameter `:ping`: `:parameters {:body [:map {:closed true} [:ping string?]]}`. `malli` is open by default (allows extra keys in maps), but you can configure your schema as closed using `{:closed true}`. The comment block mentions that at the moment `reitit` adds `mt/strip-extra-keys-transformer` and therefore we don't have any extra keys when we hit the validation - I left the `{:closed true}` here just as an example.

The [server_test.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/test/clj/simpleserver/webserver/server_test.clj) provides tests to validate that our `malli` validation works. First the GET `ping`:

```clojure
(deftest ping-get-test
  (log/debug "ENTER ping-get-test")
  (testing "GET: /api/ping"
    (let [ret (ss-tc/-call-api :get "ping" nil nil)]
      (is (= (ret :status) 200))
      (is (= (ret :body) {:reply "pong" :ret "ok"})))))

(deftest failed-ping-get-extra-query-params-test
  (log/debug "ENTER failed-ping-get-extra-query-params-test")
  (testing "GET: /api/ping"
    (let [ret (ss-tc/-call-api :get "ping?a=1" nil nil)]
      (is (= (ret :status) 400))
      (is (= (ret :body) {:coercion "malli"
                          :humanized {:a ["disallowed key"]}
                          :in ["request"
                               "query-params"]
                          :type "reitit.coercion/request-coercion"})))))
```

When calling just GET `ping` api we must succeed. When adding any query parameters (`"ping?a=1"` in the example) we must fail. `malli` also provides nice humanized error messages regarding the validations: `:humanized {:a ["disallowed key"]}`. You get the humanized error messages by providing `:humanized` in the `:error-keys` set when you create the coercion to be used with reitit:

```clojure
    (re-ring/ring-handler
      (re-ring/router routes {
                              :data {:muuntaja mu-core/instance
                                     :coercion (reitit.coercion.malli/create
                                                 {;; set of keys to include in error messages
                                                  :error-keys #{:type :coercion :in :humanized ...
```

Then the POST `ping` tests:

```clojure
(deftest ping-post-test
  (log/debug "ENTER ping-post-test")
  (testing "POST: /api/ping"
    (let [ret (ss-tc/-call-api :post "ping" nil {:ping "hello"})]
      (is (= (ret :status) 200))
      (is (= (ret :body) {:reply "pong" :request "hello" :ret "ok"})))))

(deftest failed-ping-post-missing-key-test
  (log/debug "ENTER failed-ping-post-missing-key-test")
  (testing "POST: /api/ping"
    (let [ret (ss-tc/-call-api :post "ping" nil {:wrong-key "hello"})]
      (is (= (ret :status) 400))
      (is (= (ret :body) {:coercion "malli"
                          :humanized {:ping ["missing required key"]}
                          :in ["request"
                               "body-params"]
                          :type "reitit.coercion/request-coercion"})))))
```

The POST `ping` must succeed if we provide the required `:ping` key in the body map. If the required key is missing we get a humanized error: `:humanized {:ping ["missing required key"]}`.

Let's provide another example, POST `signin`. The meaning of this API is to provide the user a method to sign-in to the web-store. The API requires the user to send his first-name, last-name, email and password:

```clojure
    ["/signin" {:post {:summary "Sign-in to get an account"
                       :responses {200 {:description "Sign-in success"}}
                       :parameters {:body [:map
                                           [:first-name string?]
                                           [:last-name string?]
                                           [:email string?]
                                           [:password string?]]}
                       :handler (fn [{ {:keys [first-name last-name password email]}
                                      :body-params}] (-signin env first-name last-name password email))}}]
```

In real-world projects if you have a big structured POST body, you should define the validation map as a `def` making the routes part more readable. This way you can also compose validations from other validations. Let's imagine instead of the previous flat representation we could have a more structured POST body, like `{:name {:first-name "mikko" :last-name "mikkonen"} :password "salainen" :email "mikko.mikkonen@foo.com"}`. We could have defined  the validation schema separately as:

```clojure
(def name-schema [:map {:closed true} [:first-name string?] [:last-name string?]])
(def sign-in-schema [:map [:name name-schema] [:email string?] [:password string?]])
...
;; and then in the routes:
    ["/signin" {:post {:summary "Sign-in to get an account"
                       :responses {200 {:description "Sign-in success"}}
                       :parameters {:body sign-in-schema}
...
```

Finally let's experiment with our `name-schema` and `sign-in-schema` a bit:

```clojure
(ns malli
  (:require [malli.core :as m]
            [malli.error :as me]))

(-> name-schema
    (m/explain {:first-name "mikko" :last-name "mikkonen"})
    (me/humanize)) ; => nil (ok)

(-> name-schema
    (m/explain {:first-name "mikko" :middle-name "pekka" :last-name "mikkonen"})
    (me/humanize)) ; => {:middle-name ["disallowed key"]}

(-> sign-in-schema
    (m/explain {:name {:first-name "mikko" :last-name "mikkonen"} :password "salainen" :email "mikko.mikkonen@foo.com"})
    (me/humanize)) ; => nil (ok)

(-> sign-in-schema
    (m/explain {:name {:first-name "mikko" :last-name "mikkonen"} :id 1 :password "salainen" :email "mikko.mikkonen@foo.com"})
    (me/humanize)) ; => nil (also ok)
```

I.e. Our `name-schema` is closed - no middle-name allowed. But we have left the `sign-in-schema` open - therefore the user of the schema can add extra keys (as `:id` in the example above). If you want to close the whole schema `malli.util` provides a function for this: `closed-schema`: 

```clojure
(ns malli
  (:require [malli.core :as m]
            [malli.error :as me]
            [malli.util :as mu]
            [clojure.data :as c-data]))

(c-data/diff (m/schema sign-in-schema) (mu/closed-schema sign-in-schema))
;; =>
;[[:map 
;  [:name [:map {:closed true} [:first-name string?] [:last-name string?]]] 
;  [:email string?] 
;  [:password string?]]
; [:map
;  {:closed true}
;  [:name [:map {:closed true} [:first-name string?] [:last-name string?]]]
;  [:email string?]
;  [:password string?]]
; nil]
```

### Conclusions

I'm sure the `spec` library is great for spec'ing the Clojure itself. But for providing schema for REST interfaces I would use `malli`. `spec` uses macros for specifying schemas - `malli` uses pure data: vectors and maps. In my opinion this makes `malli` nicer to work with schemas and also providing various technical advantages, e.g. you can manipulate `malli` schemas using any standard library functions that operate on vectors and maps.


*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>

