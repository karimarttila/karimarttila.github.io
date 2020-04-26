---
layout:	post
title:	"Five Languages — Five Stories"
date:	2018-11-19
---

  ### Introduction

Last spring I wanted to see how easy it would be to implement a small web server using Clojure and to provide a simple frontend for the backend implementation using ClojureScript. This autumn I got a nice idea: why not implement the same server using 4 other languages, 2 of which I already knew quite well (Java and Python) and 2 of which I wanted to learn (Javascript/Node and Go) so that I could compare these 5 languages how they do the job to implement the exactly same thing. So, my exercise is now over and all the five stories are available in my Medium blog:

* **Clojure**: “[Become a Full Stack Developer with Clojure and ClojureScript!](https://medium.com/@kari.marttila/become-a-full-stack-developer-with-clojure-and-clojurescript-c58c93479294)”
* **Javascript/Node**: “[Java Man’s Unholy Quest in the Node Land](https://medium.com/@kari.marttila/java-mans-unholy-quest-in-the-node-land-958e61da0451)”
* **Java**: “[Java Man Returns Home Confused](https://medium.com/@kari.marttila/java-man-returns-home-confused-2fe951eb51a9)”
* **Python**: “[Java Man Converts to Python](https://medium.com/@kari.marttila/java-man-converts-to-python-166578beeb6b)”
* **Go**: “[Go — Good Productivity with Bare Metal](https://medium.com/@kari.marttila/go-good-productivity-with-bare-metal-4fa70e0177fb)”
So, in this blog article I’m going **to compare these five languages regarding my experiences** while implementing the same server over and over again. I’m also going to **score the languages** regarding various aspects using either the same scale (1–5, **1=best,** **5=worst**), or some other metrics I feel more suitable for the situation. The reader must understand that these scores and everything I have written here are based on a very personal taste and done tongue in cheek. So, let’s start!

![](/img/1*5ZofRqU-yJafiXN2brVoeQ.png)Five languages with their IDEs in a family portrait in my Ubuntu18.### Language Characteristics

The five languages are quite different from each other which is a good thing since they are now in my toolbox — I like to have different tools for different purposes. I think the main characteristics of these languages are:

* **Compiles to machine code vs. Has runtime:** Go is the only one of these languages which compiles to machine code. Java and Clojure compile to Java bytecode which is interpreted by the Java virtual machine (JVM). Python and Javascript are interpreted languages.
* **Statically vs. Dynamically Typed**: Java and Go are statically typed languages which provide excellent language support to catch typical type errors in compile time. Clojure, Javascript and Python are dynamically typed languages — it is easy to develop using these languages but there is more room for errors in production (you can mitigate some of these issues e.g. using type hints in these languages).
* **Functional vs. Procedural vs. Object Oriented**: Java obviously is very object oriented. Using Javascript and Node you can program mostly in a weird asynchronous model, using procedural style with some taste of functional and object oriented style. Python is mostly procedural language, though you can use it a bit functional and O-O way also. Go is clearly very procedural system language. And Clojure being a Lisp is very functional with immutability by default.
* **Garbage Collection vs. Manual Memory Management**. All languages use garbage collection. Forgotten are those days when real men programmed C and managed themselves the memory allocations of their programs.
* **Concurrency Support vs. Single-Threaded**. Java, Clojure and Go provide good concurrency support out of the box. Python and Javascript are essentially single-threaded, though there are libraries for concurrency support.
* **Ubiquitous vs. Niche**. Javascript is a true ubiquitous language since it dominates the frontend. There are a lot of frontend Javascript developers and their natural choice for backend is Node. Java is also pretty ubiquitous in the enterprise backend world. Python is pretty popular language in certain areas like ML. Go is a more niche language mostly in the system development world. Clojure is very niche — only for the enlightened developers.
### Language Design

The histories of these languages are quite different. Personally I think that the best designed languages are Clojure and Go. They really fit into their ecological compartments pretty well. Java as a programming language is well designed but it is a bit too verbose and bloated, however it provides excellent abstraction mechanisms which Go lacks. On the other hand Clojure being a Lisp provides what ever abstractions you can imagine, and the language design is also a very modern version of Lisp. Python is also pretty nicely designed language for the jobs it is mostly meant. The black sheep of the class is obviously Javascript which was designed and implemented in about 10 days if the Netscape story is true (and probably it is when you learn the language and its weird syntax). So, the verdict is (I’m mostly going to use these ascii tables since there is no table support in Medium):

**| Language | Design |  
| ------------|--------|  
| Java | 3 |  
| Go | 2 |  
| Javascript | 4 |  
| Clojure | 1 |  
| Python | 2 |**### Developer Productivity

For developer productivity I have some personal hard data. Let’s see the time it took me to implement the servers using these languages:

Headers used in the table:

- Language: language  
- Evenings: how many evenings I spent with the implementation  
- 2y: how many evenings I would spend with 2 years experience  
- MPI: Marttila Productivity Index (explained later)And the results are:

**| Language | Evenings | 2y | MPI |  
| ------------|-----------|----|-----|  
| Java | 18 | 12 | 4 |  
| Go | 8 | 6 | 2 |  
| Javascript | 18 | 9 | 3 |  
| Clojure | 13? | 3 | 1 |  
| Python | 3 | 3 | 1 |**I have to be honest and I didn’t bother to check in Github how many evenings I programmed Clojure but I estimate it was something like two weeks. So, the “Evenings” are not that comparable since I had very different experiences for the languages (Java & Python: some 20 years, Javascript: some weeks, Node: zero, Clojure: about 1 year, Go: zero). But this is also pretty interesting. With some 20 years of experience with Java I manged to hassle some 18 evenings with the job (well, I hadn’t done serious Java programming for some 1,5 years and I spent quite a lot of time exploring new Java 10 features, new Spring features, new IDEA features, how to use Java REPL etc…). But with zero experience with Go and I managed to do the same server in 8 evenings. Let’s iterate this:


> 20 years of experience in Java and 18 evenings spent in implementation vs. zero knowledge of Go and 8 evenings spent in implementation. Wtf?Because the “Evenings” are not that comparable regarding my background on those languages I therefore added column “**2y**” which is my very personal feeling of *how many evenings I would do the same implementation with that language again if I had some 2 years of experience regarding that language*. So, I dropped some 6 evenings away from Java considering that I don’t spend any extra time to study new language, Spring etc. features. I think I could also squeeze Javascript to half and Clojure from 13 to 3 since it really is a very productive language once you master it. Python is already in its optimal state: 3. I could squeeze at least a couple of evenings away from Go as well.

To make this blog post look more professional and scientific I invented a new index: “*Marttila Productivity Index*” (**MPI**) so that I divided all language results of column 2y using the shortest evenings of column 2y (Python: 3). This method provides a very scientific index for each language’s productivity (tongue in cheek).

### Productivity Tools

This category gives good scoring for a language which provides excellent productivity tools. ***Lisp REPL is by far the best productivity tool I have ever used in any language***, so Clojure naturally wins this category. Python REPL is also pretty good but it is a code snippet REPL and not a real system REPL like Lisp REPLs. Java REPL is a bad joke in this category. All dynamically typed languages also provide a good feeling to create code fast, so they have good overall scores. All languages provide good IDEs with excellent debuggers, so there is no diversion there. The build tools are a bit different in these languages, most complex being Java with Maven and Gradle which are not that easy to learn as the equivalent tools in other languages.

**| Language | Productivity tools |  
| ------------|--------------------|  
| Java | 3 |  
| Go | 3 |  
| Javascript | 2 |  
| Clojure | 1 |  
| Python | 2 |**### Lines of Code

This part is probably the most scientific part in this blog post — everyone can verify the results cloning the repos in [my Github](https://github.com/karimarttila) and running “find . -iname <language-extension> | xargs wc -l”.

Headers used in the table:

- Language: language  
- P-F: production files (i.e. not including test files)  
- P-LoC: production files total lines of code (i.e. not including test files)  
- T-F: test files  
- T-LoC: test files total lines of code  
- A-F: all files together  
- A-LoC: all lines of code together  
- MLCI: Marttila Lines of Code Index (divide all language’s A-F with the lowest A-F)  
- MPI: Marttila Productivity Index (from previous chapter)And the table is:

**| Language | P-F | P-LoC | T-F | T-LoC | A-F | A-LoC | MLCI | MPI |  
| ---------|-----|-------|-----|-------|-----|-------|------|-----|  
| Java | 30 | 1612 | 4 | 440 | 34 | 2052 | 2.4 | 4 |  
| Go | 8 | 1063 | 7 | 508 | 15 | 1571 | 1.9 | 2 |  
| Javascr. | 7 | 674 | 4 | 396 | 11 | 1070 | 1.3 | 3 |  
| Clojure | 6 | 612 | 4 | 337 | 10 | 949 | 1.1 | 1 |  
| Python | 8 | 530 | 5 | 317 | 13 | 847 | 1.0 | 1 |**Let’s visualize the data to make it more readable.

![](/img/1*PJLYpebyVo9K7dtBqLFWZg.png)Lines of code.![](/img/1*UctmxepEfkzsdwziB-rAzA.png)Source code files.So, Python is the winner in both *Marttila Lines of Code Index* and *Marttila Productivity Index*. Clojure doesn’t lose that much, productivity being the same, but MLCI being just 10% higher. Javascript loses in productivity quite a bit taking some 3x more implementation time and MLCI is some 30% higher. Interestingly Go breaks the rule of correlation between MLCI and MPI — Go’s MLCI is 90% higher but MPI is only 2x. And Java, poor old Java performs worst of all: MLCI is 140% higher and MPI is 4x. You can see a clear trend — dynamically typed languages tend to have less lines of code than statically typed languages. Source code file count — Java with class this and class that in dedicated files is rather horrific.

### Performance

Go performance is excellent since Go compiles to bare metal and there is no runtime between the metal and the language as is the case of other languages (Java & Clojure: compiles to JVM bytecode, and JVM runs the bytecode, Python & Javascript: interpreted languages and the runtime runs the code (though also Python compiles to bytecode). Go also compiles extremely fast. Node’s performance is actually exceptionally good in those servers which do little CPU processing but just provide e.g. simple REST responses due to its asynchronous event model. Python’s performance is known to be quite bad for the infamous [GIL](https://en.wikipedia.org/wiki/Global_interpreter_lock) — Python’s use cases are not in the server implementations. Java’s performance is pretty good when you know how to do synchronous programming and are able to use the multi-core processors with threads. Clojure can basically perform as well as Java since JVM runs bytecode, and bytecode is bytecode whether compiled from Java or Clojure (roughly speaking).

**| Language | Performance |  
| ------------|-------------|  
| Java | 2 |  
| Go | 1 |  
| Javascript | 4 |  
| Clojure | 2 |  
| Python | 4 |**### Concurrency Support

Go’s concurrency support is excellent — it provides go routines and channels as part of the language. Clojure’s concurrency support is also excellent being a functional and immutable language (concurrency is no issue in those situations) with specific and simple mechanisms to handle concurrency when you need to share something. Java also provides good concurrency support using synchronization mechanism but it is a bit difficult to use for novice programmers. Python and and Javascript are essentially single-threaded languages that need to provide concurrency support with libraries.

**| Language | Concurrency |  
| ------------|-------------|  
| Java | 2 |  
| Go | 1 |  
| Javascript | 5 |  
| Clojure | 1 |  
| Python | 5 |**### Library Support

Java, Go, Python and Clojure provide very good standard libraries. Javascript on the other hand provides rather poor standard library and for this reason you typically are running npm installations once in a while when implementing a Node server (see next chapter “Dependencies). I’ll drop a couple points from Java since you typically do not use “pure EE”, but you have a lot of dependencies to Spring framework, which draws more dependencies along with itself and so on (more in the next chapter). So, the “Library support” metric in the table below measures two things: 1. How good the standard library is, and 2. How well you can manage just using the standard library?

**| Language | Library support |  
| ------------|-----------------|  
| Java | 3 |  
| Go | 1 |  
| Javascript | 3 |  
| Clojure | 2 |  
| Python | 2 |**### Dependencies

On the other hand — if the standard library support in Java is so good, why do you see a hideous list of over 500 lines of various dependencies when you give command: “./gradlew dependencies | wc -l” => 594, wtf? How do you even count how many dependencies are needed when you have direct dependencies A, B, C, D, E and then A and C draws J,K,L, and then B draws J,K,R,S, and then J draws R,S,T, and then R draws V,W and so on and so on (which is pretty much the case when you read that listing of over 500 lines of dependencies). And then the clashes between dependencies. You tend to write various exclusions in your build files because some dependencies R and S that are drawn by A are clashed by dependency F, which is drawn by B and so on. A real example from Java Simple Server:

testImplementation(“org.springframework.boot:spring-boot-starter-test”) {  
 exclude group: ‘junit.junit’  
 exclude group: ‘com.vaadin.external.google’  
 }To get some idea I added a task in the gradle.build file to gather all compilePath dependencies into one folder and then counted the jars: “ll build/deps/ | wc -l” => 51. There could be more test dependencies but let’s use this as a reference number for now.

For Clojure thinking about dependencies is a bit more problematic. Let’s make this simple and just list the dependencies in the project.clj file: 14.

For Python I installed pipreqs and ran it: 6.

For Javascript let’s just see the dependencies in the package.json file: 13 (not including development dependencies).

Let’s next list the dependencies and my verdict in this category:

**| Language | Dependencies | Points |  
| ------------|--------------|--------|  
| Java | 51 | 4 |  
| Go | 1 | 1 |  
| Javascript | 13 | 3 |  
| Clojure | 14 | 3 |  
| Python | 6 | 2 |**So, Go is a clear winner in this category. Python is second. Java is the ugly cousin of the family.

### Readability

There are clear differences how readable the languages are. Python is obviously the most readable language. Clojure is also pretty readable once you master the functional language. Go is not very readable because the language look a bit like C, but is pretty good anyway. Java is really verbose and loses a lot of points for bloated nature of the language and because you need to create class this and class that for everything (+ getters and setters and other boiler plate scheisse). Javascript readability is pretty horrific if you compare it to other languages, and Node’s asynchronous programming model makes it even more hideous — the very reason a linter is a mandatory tool while programming Javascript.

**| Language | Readability |  
| ------------|-------------|  
| Java | 4 |  
| Go | 3 |  
| Javascript | 5 |  
| Clojure | 2 |  
| Python | 1 |**### Abstractions

Java provides excellent abstraction mechanisms with interfaces, true class hierarchies, generics and everything. Go is on the other hand very minimalist (e.g. lacking generics) but provides some basic abstractions (e.g. simple interfaces etc.). Javascript is exceptionally moldable language after all (I read that the language designer used e.g. [Scheme](https://en.wikipedia.org/wiki/Scheme_%28programming_language%29) as an inspiration). I have never actually needed much of abstraction for those tasks that I use Python. Clojure on the other hand provides the best abstraction of all five languages being a Lisp — you can basically create what ever abstraction mechanisms you need using a Lisp (the power and curse of Lisp, see more in “[the Lisp Curse](http://www.winestockwebdesign.com/Essays/Lisp_Curse.html)” article). All other languages provide typical stream functions like filter, map and reduce except Go (I explained this in the Go Medium post article if you are interested).

**| Language | Abstractions |  
| ------------|--------------|  
| Java | 2 |  
| Go | 4 |  
| Javascript | 3 |  
| Clojure | 1 |  
| Python | 4 |**### Tooling

Tooling for all languages is pretty good. You can find good IDEs for all languages. There are a lot of libraries for each language. For Java being a mainstream enterprise language there is a lot of tooling (profilers, static code analysis tools etc.). Clojure being a niche language provides less tooling.

**| Language | Tooling |  
| ------------|---------|  
| Java | 1 |  
| Go | 2 |  
| Javascript | 2 |  
| Clojure | 3 |  
| Python | 2 |**### Testing

All languages provide good mechanisms / frameworks for testing. I wasn’t disappointed to any language when I implemented Simple Servers and created unit tests. Go and Clojure are of course a bit exceptional languages in that sense that the testing framework is part of the standard library. The only criticism I have is how easy the tests are to implement and how verbose the tests then look — there are obvious differences. Python and Clojure unit tests are very simple, short, concise and readable. Javascript and Go tests are a bit hard to read and verbose. Java is somewhere between, and Java’s weakness is that in tests you cannot treat data as data like in other languages (literal data in Python and Clojure, JSON in Javascript).

**| Language | Testing |  
| ------------|---------|  
| Java | 3 |  
| Go | 3 |  
| Javascript | 2 |  
| Clojure | 1 |  
| Python | 1 |**### Test Performance

Here we also have some hard data. I ran “time” for all projects when running the unit test suite. Here are the results:

**| Language | T-performance |  
| ------------|---------------|  
| Java | 5.8s |  
| Go | 1.9s |  
| Javascript | 0.8s |  
| Clojure | 6.0s |  
| Python | 0.4s |**It’s pretty obvious that Clojure and Java lose the contest because of the loading of JVM. Javascript and Python are pretty fast since they just start running the tests and hope that while interpreting the code there are no runtime errors. Go is a statically compiled language and needs to compile first before running tests.

### Error Handling

I kind of like Go’s error handling with error entities returned with the actual payload from functions. Java adopted the exception strategy from C++ but divided exceptions to runtime exceptions which you didn’t have to explicitly handle and checked exceptions which you had to explicitly handle — many consider this as a failed experiment since e.g. Spring exclusively uses just runtime exceptions. Python also provides exceptions and they are pretty easy to use. Though, I mostly return e.g. None from functions if something went wrong and then check if the returned value is None. Javascript as a language is a bit of a mess and so is its error handling practices. Clojure is somewhere between — there are exceptions but they are a bit cumbersome to use in a functional language so I mostly check the return values for errors.

**| Language | Error handling |  
| ------------|----------------|  
| Java | 2 |  
| Go | 1 |  
| Javascript | 4 |  
| Clojure | 3 |  
| Python | 3 |**### Language Error Messages

When programming a language you will see a lot of syntax errors. How good are these error messages for figuring out what’s the problem? A personal table follows:

**| Language | Error messages |  
| ------------|----------------|  
| Java | 1 |  
| Go | 2 |  
| Javascript | 4 |  
| Clojure | 4 |  
| Python | 1 |**So, based on my experience the best error messages are provided by Java and Python. Go is also pretty good. Javascript was pretty hideous. Clojure is also well-known of providing very cryptic error messages (some of which are Clojure and some of which are Java related — I come from the Java world so this made my life a bit easier; I can believe what a nightmare Clojure error messages are for a developer who wants to learn the language but has no knowledge of Java and JVM.

### Data Manipulation in the Code

This category relates to how well you are able to treat data as data in the code and not to be forced to use some cryptic classes to represent data. Example from server unit tests:

**Would you like to code data like this (Java):**  
 HashMap<String, String> productGroups = new HashMap<>();  
 productGroups.put("1", "Books");  
 productGroups.put("2", "Movies");  
**Or like this (Clojure):**  
 right-body {:ret :ok, :product-groups {"1" "Books", "2" "Movies"}}  
**... or this (Javascript):**  
 .expect(200, {ret: 'ok', product-groups': { 1: 'Books', 2: 'Movies'} }, done);Data should look like data in code. It’s pretty apparent that it’s more natural to treat data as data and not as some data structure. This is actually a pretty important aspect in the language and affects readability and joy of programming. Clearly Java does a terrible job here and all dynamic languages perform well (but we can’t blame Java — this is the way it is and it should be in a statically typed language; the flip side of the coin is that statically typed languages provide you type safety). Go’s structs are also a pretty good simplification for a statically typed language. Actually using Go you get the benefits of a statically typed language and treating data is not too bad. Special award goes to Javascript in which JSON (Javascript Object Notation) is built into the language and using JSON is very nice, of course (and JSON is pretty ubiquitous). Python also provides a simple way to treat data in code. As well as Clojure — using Clojure’s native types is very readable in code (see literal map in above example). So, my verdict is:

**| Language | Data in code |  
| ------------|--------------|  
| Java | 4 |  
| Go | 3 |  
| Javascript | 1 |  
| Clojure | 1 |  
| Python | 1 |**### Learning Curve

How easy is it to learn that language? I provide my very personal feeling after learning at least basics of these five languages and related tools:

**| Language | Learning curve |  
| ------------|----------------|  
| Java | 4 |  
| Go | 3 |  
| Javascript | 2 |  
| Clojure | 4 |  
| Python | 1 |**So, Python is the clear winner here. There hardly is another language which is so developer-friendly and easy to learn and use. Javascript is also pretty easy to learn. I could have given Go the same 2 points as for Javascript (and for a good reason: I implemented the Simple Server in Go in some 8 evenings — Javascript 18 evenings). I’ll give Go 3 points however, because it is a bit like C with pointers, structs and so on and I believe Go might be a bit hard to learn for some younger developers who have only programmed languages like Javascript which almost totally hides all memory management (and pointers/references). Java is so bloated with classes, exceptions, object oriented paradigm, frameworks (Spring is not that easy to learn either), build tools etc. that I have to give it 4 points. And I’m afraid I have to be honest here and give my favorite language Clojure 4 points as well — the functional paradigm, total immutability and also the JVM peculiarities in error messages might make the language very difficult to learn for some younger developers.

### Developer Pool

Java and Javascript provide the largest developer pools. There are also quite a lot of Pythonists out there but not that much as Java / Javascript programmers. Go is probably rising but I think it will mostly be a system’s language. Clojure will stay as a niche language which is a kind of pity since it is an excellent data processing language.

**| Language | Developer pool |  
| ------------|----------------|  
| Java | 1 |  
| Go | 3 |  
| Javascript | 1 |  
| Clojure | 5 |  
| Python | 2 |**### Joy of Programming

The final and most important category the “*Joy of Programming*” gives good scoring for a language that makes programming fluent and provides constant delights and new ways how the language surprises you (so, this category is very personal and also very important to me). Clojure is very good in this — it’s a real joy to program Clojure, the language surprises you almost every day and there is endless territory to learn for a person who loves learning new things. Python is also pretty nice as a language, you get to make things you want to do pretty easy (I’m referring to typical Python tasks (mostly simple scripts) — I wouldn’t use Python in a big enterprise system in which a dynamic language could turn out to be a nightmare beyond all your dreams with tens of developers shooting each others’ legs). Go is pretty good as well, but it is a bit of a system language. Java is just boring. And Javascript is more or less horrific with its weird syntax and asynchronous programming model (well, you can use Node efficiently to implement a server but why should you if there are languages you can enjoy working with while implementing the server).

**| Language | Joy of programming |  
| ------------|--------------------|  
| Java | 4 |  
| Go | 3 |  
| Javascript | 5 |  
| Clojure | 1 |  
| Python | 2 |**### Final Recommendations

I don’t provide any final scores (e.g. adding the category scores together and see which language got best overall points) — that wouldn’t make any sense. The reason is that all languages are good in the environment where they do their work best. So, I just provide some final recommendations in which environments to use a particular language (or at least I’m going to use them).

So, my final recommendations for these five languages are:

* **Python**: If you need to implement simple scripts, surrogates to aws cli etc: use Python. Python provides also excellent libraries for ML and mathematical libraries are actually implemented in C (and Python just provides frontend to the libraries) — so in ML Python is also pretty performant.
* **Clojure**: If you need to manipulate a lot of data and you need excellent concurrency support: use Clojure. The only downside with Clojure is the high learning curve for new developers (since the functional language can be rather mind bending) — therefore the developer pool is going to be always much scarcer than in other languages.
* **Go**: If you need bare metal performance with excellent concurrency support and you don’t need to manipulate a lot of data: use Go. Go probably is best as a system tool language — data manipulation is a lot more verbose than in Python and Clojure.
* **Java**: If you have a really big enterprise system and and offshore development with tens of developers working with the same code base, probably Java.
* **Javascript**: If you need a server which should serve thousands of concurrent clients really fast and processing the requests are not CPU intensive then Node and its asynchronous server model might be unbeatable.
### Disclaimer

Because I have noticed that there are many developers who think of their favorite language with great religious-like affection I must add this disclaimer. The opinions in this blog article are personal and I’m not going to read any comments that “*language X should have gotten more points because blaa, blaa, blaa*”. So, for this blog article I hope that you are not going to add any comments like that. If you didn’t like what you just read: relax, have a home brew and write your own blog article. Life is not that serious, dude. 😀

  