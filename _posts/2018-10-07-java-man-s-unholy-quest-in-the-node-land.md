---
layout: post
title: "Java Man's Unholy Quest in the Node Land"
category: [languages]
tags: [languages, javascript]
date: 2018-10-07
---

### Introduction

So, I am the oldest developer in my unit. I've been in the software industry more than 20 years. I have seen technologies come and go. New languages emerged. In recent years I recognized a clear tendency: younger developers tend to favor [Javascript](https://developer.mozilla.org/bm/docs/Web/JavaScript)/[Node](https://nodejs.org/en/) and older guys like me Java in the corporation backend world. I don't mind learning new languages, I have learned quite a few during my 20 years IT journey. About three weeks ago I got a brilliant idea: let's learn Javascript/Node and re-implement the [Simple Server](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server) I implemented last spring using [Clojure](https://clojure.org/)! I get to learn a new language and backend runtime and ecosystem and also have a great chance to compare the two languages and their ecosystems ([Javascript](https://developer.mozilla.org/bm/docs/Web/JavaScript) vs. [Clojure](https://clojure.org/)) and runtimes ([Node](https://nodejs.org/en/) vs. JVM).

You can find the project in [**Github**](https://github.com/karimarttila/javascript/tree/master/webstore-demo/simple-server).

I tried to replicate the Clojure namespace structure to the equivalent Node structures so that it is easy to compare various parts of the application in these two implementations (e.g. [server.js](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/src/webserver/server.js) — [server.clj](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/src/simpleserver/webserver/server.clj))

![](/img/2018-10-07-java-man-s-unholy-quest-in-the-node-land_img_1.png)

*Visual Studio Code hacking session running server unit tests.*

### Visual Studio Code

While learning Javascript and Node I decided to use [Visual Studio Code](https://code.visualstudio.com/) as my editor when implementing this new version of the Simple Server. Visual Studio Code was highly recommended by my unit's Javascript developers. And I understand now why. It's really a superb editor and IDE. And I managed to configure my Visual Studio Code so that it almost works exactly like my [PyCharm](https://www.jetbrains.com/pycharm/) (for Python hacking), my [IntelliJ IDEA](https://www.jetbrains.com/idea/) (for Java hacking) and my [Cursive](https://cursive-ide.com/) (for Clojure hacking). Visual Studio debugger is pretty nice. Breakpoints etc. work as in other IDEs. There are also a lot of various extensions you can use to tweak VS Code for your own purposes (e.g. Emacs binding).

### Node Development

I'm not talking about basic stuff (nvm, npm…) in this blog article since most readers already know that. I focus on writing about those areas that I found interesting in Javascript/Node from an old Java mans' point of view. If you are interested also about those area that I skip in this blog article you can read the longer story in the [readme.md](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/README.md) of the project.

#### Linting

There are linters in other languages also, of course, but in Javascript programming linters have a special meaning because of the error-prone language syntax, various ES versions and other peculiarities. There are a few linters in the Javascript world to choose from. I used [ESLint](https://eslint.org/) with [Airbnb Style Guide](https://github.com/airbnb/javascript) which was recommended e.g. in one Pluralsight Javascript/Node tutorial I watched before starting this exercise. After this exercise I now understand why linters are more or less a mandatory part of Javascript programming — linters protect the developer to shoot himself/herself on the foot with the Javascript syntax. I also got a habit of calling my "./[eslint-all.sh](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/eslint-all.sh)" script after every function implementation and fixed the linter errors not to cumulate technical debt.

#### Unit Testing

I googled which frameworks are the most popular unit testing frameworks to be used with Node — [Mocha](https://mochajs.org/) seemed to be the one. You can see the unit tests that I implemented as an exercise in the [test](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/test) directory.

Testing is the area where Node really shines. I have never seen a web server starting so blazingly fast in API testing, and also shutting down after tests. Just look at the [server.js](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/test/webserver/server.js) — before and after functions and try to run 'npm test' — damn, it's fast.

```bash
2018-10-01 20:06:36.555 DEBUG Before tests start the webserver...  
2018-10-01 20:06:36.569 DEBUG ENTER server.getInfo  
2018-10-01 20:06:36.571 DEBUG EXIT server.getInfo  
 ✓ respond with json  
2018-10-01 20:06:36.576 DEBUG After tests shutdown the webserver
```

There were three libraries that were helpful when implementing the unit tests: The [assert](https://nodejs.org/api/assert.html) library which provided the overall unit testing framework, the [underscore](https://underscorejs.org/) library for checking object equality, and the [supertest](https://github.com/visionmedia/supertest) library which provided tools for checking http return values, returned response body etc.

The testing framework output is also pretty clear to read, e.g.:

```bash
SS_LOG_LEVEL=error npm test  
  
 DomainDB module  
 Should be two product groups in domain db  
 ✓ getProductGroups returns object with 2 items  
 ✓ getProductGroups second time (from cache), returns object with 2 items  
 Should be 35 products in product group 1 / domain db  
 ✓ getProducts for pg 1 returns list with 35 items  
 ✓ getProducts for pg 2 returns list with 400 items  
 Should find product for pgId=2 and pId=49 in domain db  
 ✓ getProduct for pgId 2 and pId 49 returns list with 8 items
```

Node is fast, that was my first observation. A short comparison running unit tests in Node vs. Clojure/Lein/JVM in command line:


* Node npm/Mocha (time npm test): 0m0.634s
* Clojure Leiningen (time lein test): 0m2.605s
I.e. Node starts immediately and runs the tests. JVM boots very, very slowly, then loads Clojure jar, then loads project class files, then runs tests, and some 2,5 seconds of my precious time has been consumed that I will never get back in my life.

Well, this comparison doesn't tell the whole truth. With Clojure REPL you don't actually run the whole project unit tests at once but you work on a namespace and load it onto REPL and experiment with it — which happens immediately since JVM and Clojure jar have already been loaded into the IDE.

#### Node REPL

Node REPL is pretty good, a bit like Python REPL, but not anything like a real Lisp REPL, of course. The following code snippet is copy-pasted from the Node REPL when experimenting if there are no nulls or empty strings in a list:

```javascript
!['a', '', 'c'].some(item => ((item == null) || (item == undefined) || (item == '')) )  
false
```

When comparing Node and Clojure REPLs, Clojure wins this round hands down. The Lisp REPL is a real REPL compared to code snippet REPLs of Javascript and Python. See e.g. integrated [Cursive REPL](https://cursive-ide.com/userguide/repl.html) in IDEA (my favorite Clojure IDE). If you have never used a real Lisp REPL you just don't understand how enormously productive it can be to interact with the program you are developing.

### Other Development Experiences

**Hot code reloading**. You can use [nodemon](https://github.com/remy/nodemon) to watch any changes in your code base and automatically restart your node server — excellent during development.

[**Cross-origin resource sharing (CORS)**](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing). I was a bit puzzled that there were no CORS issues. You just had to import the [cors](https://github.com/expressjs/cors) module and tell Express server to use it and that's it (compared to rather lengthy debugging session I had to spend in the Clojure side to get all CORS issues fixed).

**Simple Frontend demonstration**. Once the Javascript/Node version of my new Simple Server was ready I tested it using the [Simple Frontend](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-frontend) I implemented earlier using [ClojureScript](https://clojurescript.org/). After fixing two minor issues I could signin, login and browse the web store using the Javascript/Node version of Simple Server as a backend just like the old Clojure version of Simple Server.

**Session handling**. Session handling is pretty straightforward, basically I just copied the session handling idea from my previous Clojure version of the Simple Server. The session validation between Simple Server and Simple Frontend is handled using [JSON Web Token](https://en.wikipedia.org/wiki/JSON_Web_Token). In this Javascript/Node Simple Server version I used the [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) library. See [session.js](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/src/webserver/session.js) which has two public functions: createJsonWebtoken and validateJsonWebToken.

### Javascript Syntax

Javascript syntax is probably the number one thing that I don't like that much in the language. I must say that the syntax is not that good if you compare it e.g. to Python (easy and concise) or Clojure (Lisp and elegant). In Clojure the Lisp and very minimal syntax is really elegant. The homoiconic nature of Lisps let you do all kinds of cool stuff in the language and in the IDE (e.g. in IDEA/Cursive I created a hot key which kills everything in this S-expression to the end of this S-expression (compared to standard Emacs hot key which kill everything from the cursor point to the end of the line)).

But once you learn to read the Javascript syntax it's — well not nice but more or less readable. I must say here that after some Python and Clojure hacking the Java syntax (verbose, very verbose) does not appeal me that much either.

### Asynchronous Programming Model

The asynchronous programming model actually hit me hard only once. When I needed the JSON web token in the [server.js](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/test/webserver/server.js) unit test I first couldn't figure out how to get the json web token returned from the request function (i.e. I couldn't assign it to a variable in the outer scope). I pretty fast realized that this is the Javascript asynchronous programming model — the outer scope had run already before the asynchronous request part was ready — outer scope variable was undefined even though I tried to assign the value to it in the request function. Some googling and I figured out how to handle the asynchronous request using [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

The asynchronous programming model makes Node super fast in non-cpu-intensive tasks, so there are benefits to balance the more complex programming model. See more [here](https://nodejs.org/en/about/).

### Learning Curve

The learning curve for Javascript / Node was very gentle — you can learn the basic stuff of Javascript and Node in a couple of days and then be productive and start implementing e.g. a REST server just like I did in this exercise. The learning curve of Clojure was a lot steeper even though I used [Scheme](https://en.wikipedia.org/wiki/Scheme_%28programming_language%29) one year at the Helsinki University of Technology back in the 90's. I believe that the learning curve for Java is also a lot steeper. The basic stuff in Javascript is pretty simple and you can be productive also with the basic stuff and learn more on the fly when working in a real project (just like in Python). This is a good thing — a language should be easy, so that you can learn the basic stuff in a couple of days and then start working with the real thing and learn more when you need it.

### Error Messages

This area is rather bad. The Javascript/Node error messages are at times almost as bad or even worse than Clojure error messages which are known to be rather hideous. E.g. in Java and Python languages error messages are much more intuitive and helpful.

### Javascript as a Language

Javascript as a language is not bad at all. The productivity is pretty good since the entry barrier to the language is really easy: you can learn the basic stuff in a couple of days and start implementing a web server with an API. And surely with time you can learn more idiomatic ways to use Javascript and become more efficient Javascript developer.

The more I programmed Javascript the more I began to like the programming model: you create functions which create/manipulate data (JSON objects, lists etc.). Functions are first-class citizens and data model (JSON) is simple. The programming model is actually a bit like in Clojure (functions that manipulate data, in Clojure data is also intuitive and clear). Actually, the more I program Clojure, Javascript and Python the more I begin to loathe Java and its unholy mess of classes mixing methods and instance variables, some of which hold data and some of which hold instances of other classes. Java and its static typing has its places, though. Static typing provides excellent tooling for IDEs and protects developers in big projects not to make trivial mistakes assuming something about parameters and return values. But in smaller projects I would rather use a language which gives a shorter development feedback cycle.

### Javascript vs. Python

I understand that younger developers who have learned Javascript when implementing frontends like to use Javascript with Node also in the backend side and use it as a scripting language with shells. That's perfectly ok. But I would say that there is a much better language to be used as a bash surrogate: blog [Python](https://www.python.org/). I have used Python some 20 years (read more in my blog [Python Rocks!]({% post_url 2017-01-30-python-rocks %})) and it really is an excellent scripting language. It is always very easy to hack something quick in Python even if you haven't used the language for several months. The syntax is just so easy and clean and intuitive (which you really can't say about Javascript). That might be the most important reason why I have never bothered to learn bash properly — you can always install python in any Linux box with one yum/apt/whatever command and then hack the evil deed in Python and call the python script in the bash script.

### Javascript and Node — Is There a Place in My Toolbox for Them?

**Backend**. Definitely yes. If I can freely choose the backend stack I would probably go for Java/Spring in enterprise type of heavy stuff with a lot of developers, Clojure in a bit more relaxed data oriented backend system, probably Python when implementing short serverless functions in AWS/Azure. But there are a lot of Node implementations out there and if some team is already using Node — no problem, let's use Node.

**Frontend**. If I need to implement a simple admin type frontend for my own purposes I probably would use [ClojureScript](https://clojurescript.org/) since I really like the syntax and the real REPL when working with Lisp (read more in my blog [Become a Full Stack Developer with Clojure and ClojureScript!]({% post_url 2018-04-22-become-a-full-stack-developer-with-clojure-and-clojurescript %})). But the reality is that there are a lot of Javascript SPAs out there and if the team wants to use Javascript with Angular/React/whatever — no problem, let's use Javascript. One thing that I have learned anyway is that [SPA](https://en.wikipedia.org/wiki/Single-page_application) seems to be the future — I don't believe server side templating paradigm that much anymore (e.g. using Java with some templating libarary is a bit yesterday).

### What Next?

I get bored easily, I think I leave the Node land at least for a while. And I have to take one AWS recertification soon and make another certification in Azure, so there is plenty of other learning to do in the near future. After those efforts I thought that it might be a nice idea to implement the Simple Server in Java and Python just to compare the four languages with the same implementation: Java, Python, Clojure and Javascript. And perhaps I'm going to learn Go next and do the implementation in Go as a Go language exercise.

Let's see if there are chances to use Javascript/Node in my future corporation projects. Time will tell. But at least for now I say goodbye to Node land.

  
