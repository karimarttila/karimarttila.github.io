---
layout:	post
title:	"Clojure First Impressions"
categories: [blog, aws]
tags: [aws]
date:	2017-08-29
---

![](/img/2017-08-29-clojure-first-impressions_img_1.png)

*Clojure code.*

### Introduction

Studying is my favorite hobby. Last winter I spent most of my evenings studying [Clojure](https://clojure.org/), the cool [Lisp](https://en.wikipedia.org/wiki/Lisp_%28programming_language%29) like language hosted on [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine). But the story started a long time ago at the Helsinki University of Technology where I started my Software Engineering studies in 1991. The professors of the university had decided to use [Scheme](https://en.wikipedia.org/wiki/Scheme_%28programming_language%29), a Lisp dialect, as an introduction language to programming — a cunning idea which put practically all freshmen at the same starting line regardless their previous programming background with C, Basic or something else. So, we read [Structure and Interpretation of Computer Programs](https://mitpress.mit.edu/sicp/full-text/book/book.html) and wrote Scheme.

Then fast forward a couple of decades with C, C++, Perl, Python, Java and some Javascript. Last winter [Timo Tapanainen](https://www.linkedin.com/in/timo-tapanainen/), my Brother in Arms in Java+AWS battle fields, told me: “Kari, have you taken a look of this new JVM language, Clojure?” I read language introductions and got interested: Lisp, running on JVM? Used in production in [Walmart](http://blog.cognitect.com/blog/2015/6/30/walmart-runs-clojure-at-scale), the top [Fortune 500 company](http://fortune.com/fortune500/)? I ordered a couple of Clojure books, [Clojure in Action](https://www.manning.com/books/clojure-in-action-second-edition) and [The Joy of Clojure](https://www.manning.com/books/the-joy-of-clojure-second-edition), read them, and experimented most of the examples with Clojure [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop). And got stunned.

### First Impressions

#### The IDE

When reading those books and experimented the examples with REPL I tried three IDE candidates. The first one was [Emacs](https://www.gnu.org/software/emacs/) with [Cider](https://github.com/clojure-emacs/cider). I have used Emacs a couple of decades in all kinds of servers (Emacs installs easily to any Solaris, Aix or Linux distro) so I had to give Emacs a chance. And it was pretty good. You could do paredit editing with Emacs, [slurping and barfing](http://danmidwood.com/content/2014/11/21/animated-paredit.html) worked fine, REPL integration was good and it even provided a crude Clojure debugger. Second candidate was [Eclipse](http://www.eclipse.org/) with [Counterclockwise](http://doc.ccw-ide.org/) plugin. It was also decent. But I had switched from Eclipse world to [IntelliJ IDEA](https://www.jetbrains.com/idea/) a bit earlier — I had used [PyCharm](https://www.jetbrains.com/pycharm/) for Python coding and decided that it makes sense to use IntelliJ IDEA for Java coding to have the same kind of look-and-feel in both IDEs. So, my third candidate was IntelliJ IDEA with [Cursive](https://cursive-ide.com/) plugin. I pretty soon fell in love with this candidate. IntelliJ IDEs were already pretty familiar to me, and Cursive provided excellent user experience for Clojure coding. Everything works just great. You have your favorite IDE and practically everything works exactly the same way as in IntelliJ IDEA and PyCharm. Cursive integrates to IDEA excellently and you can modify the IDE, e.g. keymaps, to work the way you want. I immediately turned on Emacs mode and configured slurping and barfing to work the same way I already used to work with them in Emacs + Cider.

Editing Clojure code with IDEA+Cursive is just a joy. Some might be intimidated by Lisp parentheses but using Paredit mode and slurping and barfing it soon comes as a second nature to be really productive to write Clojure code. IDEA syntax highlighting and intelligent code completion are there and working splendidly, of course. REPL integration is just great. You can create various Run configurations for REPL just like you do in IDEA for Java and PyCharm. And it is also pretty fast with my [Ubuntu](https://www.ubuntu.com/) [Linux](https://www.linux.org/) 32GB RAM machine.

#### The Language

Clojure is marvelous. Finally a Lisp you really can use to create enterprise software. As [Rich Hickey](https://en.wikipedia.org/wiki/Clojure), the creator of Clojure, said in [Software Engineering Radio](http://www.se-radio.net/2010/03/episode-158-rich-hickey-on-clojure/) interview: Earlier Lisp implementations were great but they were great isolated islands. Clojure runs on JVM and embraces the Java ecosystem with all its libraries. So, with Clojure you get the battle tested JVM which has been ported to practically any hardware and OS, and you can use all the existing Java libraries.

Clojure is dynamic, meaning it’s pretty easy to work with data using Clojure, e.g. you can query a relational database and get a list of maps returned. Clojure’s data structures are immutable by default making multi-threaded programming a lot easier than e.g. using Java’s synchronization. And one of the cool things is the language’s [homoiconicity](https://en.wikipedia.org/wiki/Homoiconicity): Clojure (like all Lisps) uses the same structure ([S-expression](https://en.wikipedia.org/wiki/S-expression)) for its program code and data structures. Clojure provides also excellent [macro](http://wiki.c2.com/?LispMacro) programming capabilities like all Lisps (don’t confuse Lisp macros with e.g. C macros, Lisp macros are a way to extend the language so that your own extensions work exactly like the core language). And the syntax is almost non-existing as in all Lisps.

#### The Programming Model

Clojure like all Lisps is a [functional language](https://en.wikipedia.org/wiki/Functional_programming). Functional paradigm sits pretty well in backend data programming. You create filter type functions and feed the data from one function to another. Functions are first-class citizens in Lisp worlds — you can assign a function to a parameter or variable and return functions from functions. It pretty soon becomes a second nature to create higher-order functions which take other functions as parameters.

Clojure provides an excellent REPL. I had already used REPL with Python, but with Python I had used REPL mostly experimenting some Python code structure before moving it to actual source code. Clojure REPL just blew my mind in a couple of days. You can import namespaces (kind of packages) in REPL, require other namespaces, integrate to your actual source code with REPL and experiment your functions in many ways using the REPL. Don’t confuse REPL to a debugger. There is also a debugger in Cursive, of course, if you want to step through your functions. I haven’t used it hardly ever since REPL provides so much interaction capabilities to experiment with the code that running a debugger is needed a lot less than e.g. with Java or Python.

### Conclusion

I have no doubt about it now - Clojure is going to be important. But only for the Brave and True (as an excellent free Clojure [book](https://www.braveclojure.com/) title says). Even though the language is just marvelous I fear that there will be some challenges to make it a mainstream language. The functional paradigm is so different from the Object-oriented paradigm. Jumping from C to Java, or from Javascript to Python is so much easier than jumping from Java to Clojure. But if you dare to take the challenge there is a great reward waiting for you: an excellent programming language that will [make you a better programmer](http://www.paulgraham.com/avg.html). I have tasted the better side of programming, and I’m not turning back.

  
