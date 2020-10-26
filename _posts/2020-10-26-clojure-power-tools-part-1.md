---
layout: post
title: Clojure Power Tools Part 1
category: [clojure]
tags: [clojure, repl, productivity, languages, cursive, intellij, programming]
date: 2020-10-26
---

![IntelliJ IDEA and Cursive](/img/2020-10-26-clojure-power-tools-part-1_img_1.png)


### Introduction

I have been working at [Metosin](https://www.metosin.fi) for some months now and I'm learning new Clojure tricks almost every day. I list here some of my favorite Clojure tricks that I have learned either from various Clojure presentations or at Metosin. I know that for a newcomer Clojure might be a bit weird language and newcomers usually don't know simple productivity tricks. Hopefully this presentation makes it a bit easier for newcomers to start being productive with Clojure.


### Start Doing It

If you are a complete newcomer in the Clojure land I have one recommendation: Just start doing it. Choose an editor, install required tools and start learning. The most important thing is that you start learning with a Clojure REPL. And with an editor that has good integration with the Clojure REPL. There are several options:

- [Emacs](https://www.gnu.org/software/emacs/) with [Cider](https://github.com/clojure-emacs/cider)
- [IntelliJ IDEA](https://www.jetbrains.com/idea/) with [Cursive](https://cursive-ide.com/)
- [Visual Studio Code](https://code.visualstudio.com/) with [Calva](https://marketplace.visualstudio.com/items?itemName=betterthantomorrow.calva)
- [Vim](https://www.vim.org/) with [fireplace.vim](https://github.com/tpope/vim-fireplace)
- [Atom](https://atom.io/) with [Proto-Repl](https://atom.io/packages/proto-repl)

I believe those are the most used editors with good REPL integrations, at least according to the latest [State of Clojure](https://clojure.org/news/2020/02/20/state-of-clojure-2020).

If you are a complete newcomer a good idea is to choose an editor that you already know. Install the Repl plugin to that editor and learn to use it. Especially learn how to send the forms (Clojure expressions) from the editor to the REPL for evaluation.

Once you have a working development environment start learning Clojure. A good resource is [Brave Clojure](https://www.braveclojure.com/). And remember to do the exercises and experiments with your editor and send the forms to REPL for evaluation using your hotkey!

### Use a REPL Scratch File

I believe that newcomers often write their experiments in the REPL editor, at least I did. Don't do it. Instead create a scratch file and write your experiments there. I watched some Stuart Halloway presentation in which he told that he has a dedicated clojure directory in which he has written all his Clojure experimentations for several years - must be pretty nice to be able to [grep](https://en.wikipedia.org/wiki/Grep) when looking for something you have written years ago. I have another habit. I have in my `~/.clojure/deps.edn` file:

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

Focus on row `:kari {:extra-paths ["scratch"]`. This line adds directory `scratch` into the Clojure path in any project in which I add my personal profile `kari`, e.g.:

```bash
clj -M:dev:test:common:backend:kari -m nrepl.cmdline -i -C
```

I.e. All projects have various aliases that you need to use when starting the REPL. I just add my personal alias `kari` at the end and then I'm able to create `scratch.clj` file in that `scratch` directory and write all project specific Clojure experimentation there. I think this is also a nice way - I have all my project related Clojure experimentation in one place. A screenshot from my IntelliJ IDEA showing the `scratch` directory and two scratch files: one for backend experimentation (`scratch.clj` - content showing in the editor window) and one for frontend experimentation (`scratch-cljs.cljs`).

![Scratch directory in IntelliJ IDEA](/img/2020-10-26-clojure-power-tools-part-1_img_2.png)

### Rich Comments

Clojurians use so called `rich comments`. These are typically small code snippets at the end of the file surrounded in `comment` form. This way the Clojure code inside the rich comment is valid Clojure code that *is read* by the Clojure reader but it is not *evaluated*. Therefore you can put all kinds of namespace specific experimentation or code examples in the rich comment block. Example from one of my exercises: [domain_postgres.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/src/clj/simpleserver/service/domain/domain_postgres.clj#L48):

```clojure
(comment
  (simpleserver.test-config/go)
  (simpleserver.test-config/halt)
  (simpleserver.test-config/test-env)
  (let [db (get-in (simpleserver.test-config/test-env) [:service :domain :db])]
    (sql-get-products db {:pg-id (str 2)}))
  (let [db (get-in (simpleserver.test-config/test-env) [:service :domain :db])]
    (sql-get-product db {:pg-id (str 2) :p-id (str 4)}))
  )
```

It's pretty easy later on to remember stuff in those rich comments. E.g. in the above mentioned example I have some functions to start/halt the Integrant test state, and some functions in which I test some sql operations in the domain using the database connection from the test state.

### Use Defs Inside Functions When Debugging

This is a classic Clojure trick. If you are wondering what is happening inside some function - just take a snapshot of the most important entities in that function. Let's look at the same file, [domain_postgres.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/src/clj/simpleserver/service/domain/domain_postgres.clj#L37):

```clojure
(if-let [kv (sql-get-product db {:pg-id (str pg-id) :p-id (str p-id)})]
```

I wonder what kind of data structure is returned from the function and bound to `kv`? Let's see:
```clojure
    (if-let [kv (sql-get-product db {:pg-id (str pg-id) :p-id (str p-id)})]
      (let [_ (def todo-kv kv)]
        [(:id kv)
```

The magic is in the `let` I added: `(let [_ (def todo-kv kv)]`: we just create a *var* (`todo-kv`) and bind the value of `kv` to it.

Then let's run the tests so that this function gets called and after the test let's examine `todo-kv`:

```clojure
todo-kv
=>
{:id "4",
 :pg_id "2",
 :title "Test Once Upon a Time in the West",
 :price 14.40M,
 :a_or_d "Leone",
 :year 1968,
 :country "Italy-USA",
 :g_or_l "Western"}
```

### Use Hashp for Printing Values

Sometimes you want to preserve the data structure and examine it in your scratch file - then the `def` trick described in the previous chapter is all you need. But sometimes you just want to see the value. You could use e.g. [clojure.pprint](https://clojure.github.io/clojure/clojure.pprint-api.html) to print the value of the var but there is another nice power tool for it: [hashp](https://github.com/weavejester/hashp). E.g. in your rich comment evaluate: `(require '[hashp.core])` and then add `#p` before the function you are interested, e.g.:

```clojure
(if-let [kv #p (sql-get-product db {:pg-id (str pg-id) :p-id (str p-id)})]
```

Evaluate the namespace (or reset Integrant state or something similar) and run the tests and you will see an output like:

```clojure
Testing simpleserver.service.domain.domain-test
#p[simpleserver.service.domain.domain-postgres.PostgresR/fn:37] (sql-get-product db {:p-id (str p-id), :pg-id (str pg-id)}) => 
{:a_or_d "Leone",
 :country "Italy-USA",
 :g_or_l "Western",
 :id "4",
 :pg_id "2",
 :price 14.40M,
 :title "Test Once Upon a Time in the West",
 :year  1968}
```

### Use Panel to Visualize Data

This is pretty cool and I learned this only this week from one Metosin Clojure guru (aah, I need remember to write a chapter: "Find a Clojure Shop").

If you have a complex domain in which you have a lot of data with a lot of children and those children having children and so on - it is a bit taunting to try to visualize this in the REPL output window or using the REPL to navigate in the data. [portal](https://github.com/djblue/portal) to the rescue!

Write the following forms in your scratch file or in the rich comment of your namespace - I'll use the same [domain_postgres.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/re-frame-demo/src/clj/simpleserver/service/domain/domain_postgres.clj) file as an example.

```clojure
  (require '[portal.api :as portal-api])
  (portal.api/open)
  (portal.api/tap)
  (tap> (let [db (get-in (simpleserver.test-config/test-env) [:service :domain :db])]
    (sql-get-products db {:pg-id (str 2)})))
```

So, the `(portal.api/open)` opens the visualization window, `(portal.api/tap)` adds portal as a tap, and then using [tap>](https://clojuredocs.org/clojure.core/tap%3E) you can send data to the visualization window. See example:

![IntelliJ IDEA and Cursive](/img/2020-10-26-clojure-power-tools-part-1_img_3.png)

This is a very simple example. But imagine that there are hundreds of lines of data, vectors, maps, more vectors and maps inside them etc. A visualization tool like Portal is a must-have tool. There are other similar visualization tools - the Clojure itself has one nowadays: [clojure.inspector](https://clojure.github.io/clojure/clojure.inspector-api.html)


### Find a Clojure Shop

If you really want to learn Clojure I strongly recommend you to find a job in which you are able to spend 8 hours every work day with Clojure - with other seasoned Clojurians. I really can't overemphasize how important this is. I learned the basic stuff of Clojure alone, just reading books, googling stuff, doing exercises. I had a long career in the corporate world and one day I realized that I have paid my mortgage and my kids are adults and I can basically do whatever I like with my life - and I asked myself: What do you want really want to do? I realized that I want to do Clojure & cloud projects. I contacted the most prominent Clojure shops in Finland and got a job at the best of them, [Metosin](https://www.metosin.fi). Damn, it's been a jolly ride with Metosin. The company is full of world-class Clojurians, just look at some extremely popular Metosin libraries like [reitit](https://github.com/metosin/reitit). And I got a chance to work in the same project with guys who build these libraries. I should be paying them instead company paying me. What an incredible luck. In a few months I have already learned so much from these guys. And learning more and more every day.

So. If you really want to learn Clojure, it is really, really important to find a great Clojure shop where you can learn from the best.


### Conclusions

There are a lot of other Clojure tricks and tools - maybe I'll write "Clojure Power Tools Part 2" blog post in the future.


*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
