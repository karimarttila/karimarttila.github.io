---
layout:	post
title:	"Clojure Impressions Round Three"
categories: [blog, aws]
tags: [aws]
date:	2020-01-06
---

  ![](/img/2020-01-06-clojure-impressions-round-three_img_1.png)The Clojure IDEs used in this exercise: Emacs + Cider and Intellij IDEA + Cursive

### Introduction

This Clojure Simple Server is a re-implementation of my original Clojure Simple Server I did some three years ago. I created the first version of that Clojure exercise server to learn to use Clojure. I created this new Clojure Simple Server version mainly to learn new Clojure libraries and new ways to work more efficiently with the Clojure REPL. If you are interested about my Clojure learning history you can read my earlier Clojure related blog posts:

* [Clojure First Impressions](https://medium.com/tieto-developers/clojure-first-impressions-2c6232f4b514)
* [Clojure Impressions Round Two](https://medium.com/tieto-developers/clojure-impressions-round-two-f989c0945f4b)
* [Using Clojure to Implement a Web Service Server](https://medium.com/@kari.marttila/using-clojure-to-implement-a-web-service-server-53f62dca964f)
* [Become a Full Stack Developer with Clojure and ClojureScript!](https://medium.com/@kari.marttila/become-a-full-stack-developer-with-clojure-and-clojurescript-c58c93479294)
* [Five Languages — Five Stories](https://medium.com/@kari.marttila/five-languages-five-stories-1afd7b0b583f)
In this new “Clojure Impressions Round Three” blog post I document some new Clojure programming practices I learned doing this new Clojure exercise.

You can find the project in [Github](https://github.com/karimarttila/clojure/tree/master/webstore-demo/simple-server).

### Using a REPL Scratch File

I watched [Chicago Clojure — 2017–06–21 — Stuart Halloway on Repl Driven Development](https://vimeo.com/223309989) presentation and I realized that previously I had done REPL driven development completely wrong: I had written all REPL experimentation in the REPL editor. Instead you should use some scratch file and just send the S-expressions to the REPL for compilation and evaluation. This way you can have all your experimentation saved in a scratch file. In this exercise I actually created three kinds of Clojure files:

1. [**src**](https://github.com/karimarttila/clojure/tree/master/webstore-demo/simple-server/src): the actual production source code. This code will be packaged as the production deployment that runs the application in production. Will be added also into the Git repository, of course.
2. [**test**](https://github.com/karimarttila/clojure/tree/master/webstore-demo/simple-server/test): the test source code. This code is for testing purposes and is not shipped into production but will be added also into the Git repository, of course (e.g. to be run in the CI server).
3. [**dev-src**](https://github.com/karimarttila/clojure/tree/master/webstore-demo/simple-server/dev-src): the development source code. This code is just for development purposes and you don’t necessarily add this code into the git repository (and you don’t package this code as part of the deployment unit, of course). I have added this code also in the Git repository for educational purposes. In this directory I have two files:
* [**mydev.clj**](https://github.com/karimarttila/clojure/blob/master/webstore-demo/simple-server/dev-src/mydev.clj): auxiliary functions to call APIs for experimental testing etc.
* [**myscratch.clj**](https://github.com/karimarttila/clojure/blob/master/webstore-demo/simple-server/dev-src/myscratch.clj): the REPL scratch file I talked earlier. This file is pure unstructured mind flow. The idea of this file is what Stuart Halloway is talking about in the above mentioned presentation: do not write code in the REPL editor but in a scratch file. So, in this file I experimented various things when developing the application — take a look to have an idea what I was thinking about when developing the application.
In my scratch file I have experimental code or various short Clojure code snippets e.g. to start the server and test something quickly. Example:

(do  
 (in-ns 'simpleserver.webserver.server)  
 (start-web-server (get-in simpleserver.util.config/config [:server :port]))  
 (let [  
 \_ (require 'mydev)  
 ret (mydev/do-get "/info" {})]  
 (prn (str "/info returned: " ret)))  
 (stop-web-server)  
 (in-ns 'user))I.e. jump into the right namespace, start the server, curl one api, stop the server and go back to user namespace. You can run the S-expression in the scratch file with one hotkey, of course (more about configuring Clojure hot keys in my next blog post, so stay tuned!).

So, if you are learning Clojure I strongly recommend to learn efficient REPL practices, e.g. watch the above mentioned Stuart Halloway’s presentation. Another great resource for learning efficient Clojure REPL practices is Eric Normand’s excellent [REPL Driven Development](https://purelyfunctional.tv/courses/repl-driven-development-in-clojure/) course. REPL is the very heart and soul of Clojure development — learn to use it.

### You Can Do It Without Application State Management Libraries

I realized that for a simple application like this exercise you don’t actually need some Clojure application state management library (like Component, Mount or Integrant). And personally I also realized that it is better to learn to compile your Clojure code in those three categories (individual S-expression, def and defn top forms and the whole namespace) when you change something than just blindly *“refresh all namespaces and rely some application state management library dependency graph magic to do it for you so that you don’t need fully to understand what actually happened”.* Well, this is my personal feeling and there might be reasons for more complex applications to use some real application state management library — let’s see when I have a chance to do something more complex using Clojure hopefully in the near future. So, my recommendation is first to try to manage without any application state management library and try to understand how to compile and load certain Clojure constructs manually.

### Rich Comments

This is a trick I learned from one of Stuart Halloway’s excellent videos. Add at the end of your Clojure (production) file a “rich comment” which demonstrates how to use the entities defined in that namespace. Example from: [domain\_single\_node.clj](https://github.com/karimarttila/clojure/blob/master/webstore-demo/simple-server/src/simpleserver/domain/domain_single_node.clj):

(comment  
 (-get-raw-products 1)  
)I.e. a comment S-expression defines valid Clojure code that gets parsed *but not evaluated*. This way the code doesn't have any effect in production but during development it provides a nice way to give examples how to use entities in your Clojure code and also quickly evaluate the S-expressions inside the comment block (efficiently and quickly using your Clojure editor hot keys, of course).

### Debugger

You don’t need a debugger with Clojure. :-) There is a debugger in IntelliJ IDEA + Cursive (and in Emacs + Cider), but I only once tried that it works. If you want to learn good debugging practices in Clojure you need to learn to use the REPL efficiently. Good resources are the above mentioned [REPL Driven Development](https://purelyfunctional.tv/courses/repl-driven-development-in-clojure/) course and [Chicago Clojure — 2017–06–21 — Stuart Halloway on Repl Driven Development](https://vimeo.com/223309989) video. E.g. in that video Stuart shows how to implement a simple breakpoint using Cursive — i.e. create your own debugger!

### Linting

I used [clj-kondo](https://github.com/borkdude/clj-kondo) as a linter. There are good instructions [how to use clj-kondo with IntelliJ IDEA](https://github.com/borkdude/clj-kondo/blob/master/doc/editor-integration.md). With IntelliJ integration clj-kondo is a really good tool you should use with IntelliJ + Cursive. There are good instructions also for [Emacs integration](https://github.com/borkdude/flycheck-clj-kondo). I tried clj-kondo also with Emacs and it was pretty easy to install and use.

### Unit Testing

To run the unit tests in command line use: [run-tests.sh](https://github.com/karimarttila/clojure/blob/master/webstore-demo/simple-server/run-tests.sh), example:

./run-tests.sh env-dev-single-nodeStarting the unit tests in command line takes some time since Clojure needs to boot a new JVM instance. I ran unit tests in command line usually only once per coding session: before commiting the code to Git. You seldom need to run the unit tests in command line since you have a REPL integration in your editor and you have a live REPL in your editor at all times when you are programming new Clojure code. So, you usually create new code and experiment with the code in your scratch file sending forms for evaluation to REPL (with your hotkeys). When you are happy with the functionality you move the production part of the code to src tree and the test part to test tree. And then run the unit tests in the test tree with your hot keys by sending the tests for evaluation to a running REPL session (i.e. no JVM boot — very fast).

### Using IntelliJ IDEA + Cursive and Emacs + Cider Interchangeably

I used [IntelliJ IDEA](https://www.jetbrains.com/idea/) + [Cursive](https://cursive-ide.com/) plugin when implementing this Clojure exercise but just out of curiosity I wanted to try how it feels like to use [Emacs](https://www.gnu.org/software/emacs/) +[Cider](https://github.com/clojure-emacs/cider) when programming Clojure. The experience was quite pleasant. I managed to configure Emacs with the same theme I use with IntelliJ ([leuven](https://github.com/fniessen/emacs-leuven-theme) — mostly the same colors for the same syntactic/semantic entities) and the same hotkeys to compile code regarding one S-expression or the whole namespace, run all tests in the namespace or just the test under cursor etc. It took some time and some googling regarding Emacs [elisp](https://www.gnu.org/software/emacs/manual/html_node/elisp/) but finally I was pretty satisfied: now I can quite effortlessly use either IntelliJ IDEA + Cursive or Emacs + Cider when developing Clojure code — and the look-and-feel is pretty much the same. Why? Well, e.g. you can use Emacs + Cider in headless environments where you don’t have GUI.

Emacs with various helpful packages like [company](https://company-mode.github.io/), [projectile](https://github.com/bbatsov/projectile), [clojure-mode](https://github.com/clojure-emacs/clojure-mode), [cider](https://github.com/clojure-emacs/cider), [eldoc-mode](https://www.emacswiki.org/emacs/ElDoc) turn Emacs to quite a powerful integrated development environment for Clojure. I recommend you to give Emacs a try. But I must say that I think I will stick with IntelliJ IDEA + Cursive when programming Clojure — IntelliJ is just a superb IDE.

By the way. **Stay tuned** in my blog, since my next blog post will provide the experiences how to configure IntelliJ IDEA + Cursive and Emacs + Cider with various hotkeys to make Clojure coding a breeze!

### For a Clojure Learner

I hope this exercise is helpful also for someone learning Clojure. I have tried to provide some efficient Clojure programming practices and links to various resources you can find more information. Try to configure e.g. Emacs the same way I did, git clone this repository, load various namespaces and try to send the S-expressions in the scratch file for evaluation to REPL.

*The writer is working in *[*Metosin*](https://www.metosin.fi/)* using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
  