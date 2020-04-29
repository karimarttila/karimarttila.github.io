---
layout: post
title: "Become a Full Stack Developer with Clojure and ClojureScript!"
category: [clojure]
tags: [clojure, clojurescript, web, fullstack]
date:	2018-04-22
---

### Introduction

I’m an old backend developer. I have implemented various backends using C, C++, Java, Python and now recently Clojure. I have implemented some frontends too, of course. Mostly with some Java backend templating technologies like JSP, Struts and JSF. A couple of years ago I wanted to learn how to implement a [Single Page Application](https://en.wikipedia.org/wiki/Single-page_application) (SPA) and I implemented one production system with Javascript and Angular. There was quite a bit of learning since besides Javascript and Angular you had to learn also the browser programming pecularities and various Javascript tools like Grunt etc. And I must say I never learned to like Javascript as a programming language.

### ClojureScript

Then came a chance to try [ClojureScript](https://clojurescript.org/). I had already seriously fallen in love with the [Clojure](https://clojure.org/) programming language (see my previous blog articles regarding Clojure). Clojure is a modern Lisp implementation that is hosted on the Java Virtual Machine. Clojure as a language is just superb, the best language I have ever used. Elegant, efficient and productive, and a real joy to use as a programmer. ClojureScript compiles to Javascript so with ClojureScript you can use the same efficient and beautiful language to implement SPAs that run in the browser. And now that I have some experiences implementing a SPA both with Javascript and ClojureScript — ClojureScript wins hands down. Some of my personal opinions why this is so:

* You can implement SPA using the same language you are familiar with backend (sure, you can do this using Javascript+Node, but JVM in the backend is just superior in most cases).
* Clojure(Script) as a programming language is just more efficient and elegant than Javascript.
* You can use the same tooling (e.g. Leiningen) in the frontend side as in the backend side.
* Using Clojure’s data oriented programming language implementing HTML code is much cleaner that with Javascript. E.g. the [React](https://reactjs.org/) library wrapper [Reagent](http://reagent-project.github.io/index.html) maps directly to Clojure data structures.
I’m not going into Javascript vs. ClojureScript discussion in more detail, you can google the subject if you are interested (a good post is e.g. “[Why I chose ClojureScript over JavaScript](https://m.oursky.com/why-i-chose-clojure-over-javascript-24f045daab7e)”).

If you want to read comparison regarding various Javascript libraries I suggest this article: “[A Real-World Comparison of Front-End Frameworks with Benchmarks (2018 update)](https://medium.freecodecamp.org/a-real-world-comparison-of-front-end-frameworks-with-benchmarks-2018-update-e5760fb4a962)” — ClojureScript won the Lines of Code category.

### Clojure / ClojureScript Development

For my own learning purposes I implemented a hypothetical web store which sells movies and books. I implemented a server with a REST interface for serving various requests for product groups, products and product info. I implemented also crude sign-in and login functionality for the server and for the frontend. At the same time when implementing the server side I implemented the frontend using ClojureScript. The demonstration can be found in my Github account: [clj-ring-cljs-reagent-demo](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo). (Disclaimer. I didn’t worry too much e.g. of the visual look-and-feel of my demo since I wanted to concentrate on the frontend-backend interaction, see more in [README.md](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-frontend/README.md).)

I’ll explain some typical Clojure + backend / ClojureScript + frontend development scenarios. I gathered the most relevant tools in one screen for illustration purposes (in real life I have three screens and all these windows are spread in those three screens so that I can look at them at the same time). For illustration purposes I have also added numbers with red circles for each window (I’ll refer to those numbers in the text like “#1”).

![](/img/2018-04-22-become-a-full-stack-developer-with-clojure-and-clojurescript_img_1.png)

*Clojure / ClojureScript Development.*

The windows in the screen are:

* #1. Clojure Backend IDE
* #2. Clojure Ring Server Console
* #3. ClojureScript Frontend IDE
* #4. ClojureScript Leiningen Figwheel Console
* #5. Browser
I’ll next explain how to use these tools when implementing a server using Clojure and a frontend using ClojureScript.

### Backend: #1. Clojure IDE and #2. Server Console

#### #1. Clojure IDE and REPL

I use [IntelliJ IDEA](https://www.jetbrains.com/idea/) + [Cursive plugin](https://cursive-ide.com/) as my Clojure IDE (#1.). I evaluated other Clojure IDEs as well but I just fell in love with Cursive and its REPL (see my previous article “[Using Clojure to Implement a Web Service Server](https://medium.com/@kari.marttila/using-clojure-to-implement-a-web-service-server-53f62dca964f)” regarding my favourite Cursive configurations to boost Clojure programming productivity). As you can see in the picture you can have various REPLs in the IDE. The one that shows in the picture is a local REPL — local to your code. It is easy and fluent to test Clojure functions in isolation in a local REPL (as you can see the “(get-product 1 2001)” function call in the local REPL. You can also easily configure a remote REPL — a REPL that is connected to your running server. In a remote REPL after sign-in and login (either using the real Frontend, [Postman](https://www.getpostman.com/) or [Curl](https://en.wikipedia.org/wiki/CURL) — for Curl see the [scripts directory](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server/scripts) in which I provide some examples how to curl the server REST interface) you can check the sessions atom:

simpleserver.webserver.session/my-sessions  
=>  
#object[clojure.lang.Atom  
 0x3593d88  
 {:status :ready,  
 :val #{"eyJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6InUiLCJleHAiOjE1MjQzMDUyNDZ9.Exz5iyNR9aK5pkH2lBbCqxD-didkGmRXCQ4mU1LSzIQ"}}]… to check that the server has created a JSON Web Token and stored it in the server’s atom for sessions.

See more examples how to use the REPLs for exploratory testing in Simple Server’s [README.md file](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/README.md).

#### #2. Server Console

The Simple Server is implemented using Clojure and some popular Clojure server side libraries like:

* [Ring](https://github.com/ring-clojure): Various web application libraries for Clojure.
* [Compojure](https://github.com/weavejester/compojure): A REST routing library for Clojure.
* [Buddy](https://github.com/funcool/buddy): A security library for Clojure.
You can start the server with command:

SIMPLESERVER\_CONFIG\_FILE=resources/simpleserver.properties lein with-profile +log-dev ring server-headlessThis starts the server in development mode which hot reloads the code while you are changing it in the IDE (no need for any specific hot reloading tools like in the Java side). I always implement a rather detailed trace log to my servers to make development easier (you can keep the console in one screen to see what’s happening in the server side while you are doing exploratory testing).

If you are interested about the server side development have a look at the session handling example I have written in the Simple Server’s [README.md — Session Handling chapter](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/README.md#session-handling).

### Frontend: #3. ClojureScript IDE, #4. Figwheel REPL and #5. Browser

#### #4. Figwheel REPL

Sorry about the numbering but I realized that it is better to describe the Figwheel window first. So, for ClojureScript frontend development your definite friend is [Figwheel](https://github.com/bhauman/lein-figwheel). You start the Figwheel REPL like:

lein figwheelFigwheel compiles your ClojureScript code on the fly to Javascript, builds your application and hot reloads the code changes to the browser. It works really well — it is pretty easy to write ClojureScript code and do exploratory testing in the browser since Figwheel takes care of everything. And an extra cool thing is a remote REPL to your Single Page Application in the browser — you can call any function or check the status of application data structures in the live browser application! I have an Ubuntu workstation and using Terminator console the font colors for some reason are rather invisible in the Figwheel console. I didn’t bother to fix this since it is easier to use Cursive IDE as a remote REPL anyway (more about in the next chapter).

#### #3. ClojureScript IDE and Remote REPL

What is really cool is that you can have the same superb IDE in both server and frontend side.

So, as I already started to tell in the previous chapter you have a remote REPL in the Figwheel console but I feel it easier to use my IDE’s own REPL since it is easy to use my favorite hotkeys to jump from code editor window to REPL window and back. I have provided detailed instructions how to configure Cursive IDE to connect to a remote REPL for your SPA running in a browser, see Simple Frontend’s [README.md](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-frontend/README.md). In the above picture I have queried the Frontend’s session atom:

@simplefrontend.session/app-state  
=> {:page :productgroups,  
 :token "eyJhbG...How cool is that! You can have two remote REPLs: one connected to your live server and one connected to your live SPA in the browser! Web application development is a real breeze using Clojure!

Some libraries I found helpful in the ClojureScript frontend development:

* [Reagent](https://reagent-project.github.io/): An excellent ClojureScript wrapper to the popular [React](https://reactjs.org/) Javascript library.
* [Secratary](https://github.com/gf3/secretary): A client-side router for ClojureScript.
* [cljs-ajax](https://github.com/JulianBirch/cljs-ajax): An Ajax client (for calling server’s REST interface).
The Reagent code maps directly to the Clojure data structures so it is very easy to use it in the ClojureScript code. A code example using Reagent:

; An example of a React component.  

```clojure
(defn input  
 "Input field component e.g. for First name, Last name, Email address and Password."  
 [label name type my-atom]  
 (fn []  
 [:div  
 [:label label]  
 [:input {:id name  
 :name name  
 :type type  
 :value [@my](http://twitter.com/my "Twitter profile for @my")-atom  
 :on-change #(reset! my-atom (-> % .-target .-value))}]])); ... and later using the input component in the Login page:(fn []  
 [:div  
 [:h1 "Login"]  
 [:form  
 [:div [(sf-components/input "Email address: "  
 "email-address"  
 "text"  
 email-address-atom)]]  
 [:div [(sf-components/input "Password: "  
 "password"  
 "text"  
 password-atom)]]]  
 [:div [:input {:type "button" :value "Submit"  
 :on-click #(-submit-form [@email](http://twitter.com/email "Twitter profile for @email")-address-atom  
 [@password](http://twitter.com/password "Twitter profile for @password")-atom)}]]
```

You can bind the React component to an atom in your ClojureScript code — React then takes care of the two way updates between your html entity and the internal ClojureScript atom data structure in your code.

#### #5. Browser and Developer Tools

I used Chrome while developing the application. I’m not that fluent frontend developer so I can’t tell you the most bleeding edge frontend development tricks but let’s tell some basic stuff here anyway. What you really should do while doing exploratory testing in the browser is to use the Developer Tools of your browser — in Chrome the three dots in the upper right corner and then — More tools — Developer tools. I’m sorry I didn’t show that in the picture above so let’s show the most important browser Developer tools here:

![](/img/1*RGcCr_LTjPBLOAzcQ1naHA.png)

*Chrome Developer tools — Console.*

![](/img/2018-04-22-become-a-full-stack-developer-with-clojure-and-clojurescript_img_2.png)

*Chrome Developer tools — Network.*

So, in the Developer tools you can see e.g. the Browser SPA console trace log (in Console tab). What is also helpful is the Network tab in which you can check the http request/response headers.

**CSS**. I later implemented simple CSS for the Simple Frontend using popular [Bootstrap](https://getbootstrap.com/) library (the main picture still shows the old UI, let’s show one screen of the new UI below).

![](/img/2018-04-22-become-a-full-stack-developer-with-clojure-and-clojurescript_img_3.png)

*A bit cleaner UI.*

### Getting Help

If you get stuck in something (like I did with [CORS issues](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/README.md#cors-issues)) don’t spend too much time hitting your head on the wall, just google or ask for help. A great place for clojurians to ask help is the [Clojurians Slack](https://clojurians.slack.com) (use #beginners channel). My CORS issues were pretty soon solved once I stopped hitting my head on the wall and asked help.

### Conclusions

Clojure and ClojureScript are real alternatives to e.g. Java / Javascript in web application development. You can use the same beautiful and efficient language with the power of JVM in the server side and full SPA capabilities in the frontend side.

After learning Clojure an old backend developer can learn ClojureScript and basic frontend techniques in a few days — this is pretty nice since you can then easily implement yourself some crude e.g. administrative frontends to your own backends.

### Shameless Advertisement

I’m currently looking for an interesting project in which I could use Clojure to implement a backend and ClojureScript to implement a frontend. I’m also pretty fluent with AWS so I can create the AWS infrastructure for my system as well. Now that I have paid my mortgage I’m not that interested to do bulk Java programming any more — let’s start enjoying life and do Clojure!

  
