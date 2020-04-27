---
layout:	post
title:	"Go — Good Productivity with Bare Metal"
categories: [blog, aws]
tags: [aws]
date:	2018-11-13
---

### Introduction

This is the fifth blog article (and at least for now the last one) in my series to implement the same web server in five different languages. The previous articles are:

* **Clojure**: “[Become a Full Stack Developer with Clojure and ClojureScript!](https://medium.com/@kari.marttila/become-a-full-stack-developer-with-clojure-and-clojurescript-c58c93479294)”
* **Javascript/Node**: “[Java Man’s Unholy Quest in the Node Land](https://medium.com/@kari.marttila/java-mans-unholy-quest-in-the-node-land-958e61da0451)”
* **Java**: “[Java Man Returns Home Confused](https://medium.com/@kari.marttila/java-man-returns-home-confused-2fe951eb51a9)”
* **Python**: “[Java Man Converts to Python](https://medium.com/@kari.marttila/java-man-converts-to-python-166578beeb6b)”
This blog article is about my fifth language, [**Go**](https://golang.org/) (or ‘golang’ for search engines). This time I did this exercise just to learn Go programming language and the Go tools. I have been reading about Go for a while and I was pretty interested to learn Go. This exercise of implementing the same web server with different languages has taught me that if you want to learn a language the best way is just to start implementing something — this way you have to find ways of molding the language to your needs and also learn the libraries and tools related to that language. I implemented the Simple Server in about 8 evenings, which is not that bad if you compare it to some 18 evenings I used for implementing the same server in Java (and comparing to 3 evenings in Python). So, Go provides pretty good productivity being a statically typed language that compiles straight to the machine code.

Once again, you can find the project in [**Github**](https://github.com/karimarttila/go/tree/master/simpleserver).

And once again I tried to replicate the file/namespace/class names so that it is easy to compare the implementations (e.g. [server.go](https://github.com/karimarttila/go/blob/master/simpleserver/app/webserver/server.go), [server.py](https://github.com/karimarttila/python/blob/master/webstore-demo/simple-server/simpleserver/webserver/server.py "server.py"), [server.js](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/src/webserver/server.js), [server.clj](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/src/simpleserver/webserver/server.clj) and [Server.java](https://github.com/karimarttila/java/blob/master/webstore-demo/simple-server/src/main/java/simpleserver/webserver/Server.java)).

**Important note! **I’m not comparing these five languages between each other in this blog post but mostly just focus on Go. I’m planning to write another blog post in which I’ll reflect my experiences implementing the same web server in five different languages and comparing the languages — so stay tuned!

![](/img/1*HJhQxqySlzFaIAuwMTlNkw.png)GoLand hacking session running server unit tests.### Learning Process

It’s really fun and interesting to learn a new programming language. When you have zero knowledge before taking the challenge learning the new language is a kind of exploration expedition to a new land with different language and customs — the only way to effectively learn it is to go there, learn the language and its idioms.

I had practically zero knowledge of Go when I started this project. I watched one short “Go Basics” type Pluralsight video before starting to program Go. Mostly everything I just learned on the fly while doing programming. E.g. I was wondering what to do in a situation in which some Go function returns three distinct return values but I need only one (and the compiler complains if the other two variables are not used)? I handled these situations just by googling, e.g. “golang function returns multiple values unused variable” — and you pretty soon find a page advising to use underscore for those variables you don’t need.

For my own learning purposes I commented quite a few of these lines so that I can use this Go code of Simple Server for my future reference implementation of Go. Example:

// NOTE: In Go public variables and functions start with capital letter.  
var MyLogger = initLogger()  
…  
// NOTE: Use underscore ‘\_’ when you don’t need to reference certain return values.  
pc, \_, \_, \_ := runtime.Caller(1)### Go

I was using [Go](https://golang.org) version 1.11 on Ubuntu18 when implementing this Simple Server.

When programming Go you have to set the $GOPATH and $GOROOT environmental variables to point to your Go project directory and where your Go installation is. See a more detailed example in [setenv.sh](https://github.com/karimarttila/go/blob/master/simpleserver/setenv.sh).

I used [dep](https://github.com/golang/go/wiki/PackageManagementTools) tool to mangage Go packages. I had to use only one package (for JSON web token handling) — everything else was possible to implement using just the Go standard library. The dep tool is not that much of a package manager. For package management Go is one of the worst I’ve seen — package management is basically “go get <some-git-repo>” and you get the source files from that git repo to your $GOPATH and Go compiler compiles and links those sources as part of your binary. A rather weird system if you have used e.g. Java’s [Java ARchive](https://en.wikipedia.org/wiki/JAR_%28file_format%29) files and dependencies to Maven repositories. But I understood that this is going to change in some future Go version.

### GoLand

I used [GoLand](https://www.jetbrains.com/go/) as my Go IDE. I use [IntelliJ IDEA](https://www.jetbrains.com/idea/) for Java programming, [PyCharm](https://www.jetbrains.com/pycharm) for Python programming and IntelliJ IDEA with [Cursive](https://cursive-ide.com/) plugin for Clojure programming. Since GoLand, PyCharm and IDEA are provided by the same company (JetBrains) they provide very similar look-and-feel. So, there are a lot of synergy benefits to use the same IDE for several programming languages (I have promoted JetBrains so much in my blogs that they should at least send me the IDE stickers for my laptop, I wouldn’t mind a nice coffee mug either).

GoLand is really great for Go development (as JetBrains products are for other languages). I created a test run configuration for each package and while I was developing that package I once in a while ran the equivalent GoLand test run configuration. If there were some errors it was very fast to add a debugger breakpoint to the failed test, hit the debugger and check the system state (variable values…) in the breakpoint. Go compiles extremely fast and the GoLand debugger starts blazingly fast so developing Go code like this was really fast. Using the GoLand debugger it’s also a nice way to look what’s inside the standard library entities.

### Code Format

Go is an interesting language in that sense that formatting of the Go code is very opinionated. Very opinionated in that sense that the Go compiler even requests code to be in certain format or it doesn’t compile the code even though the code would otherwise be syntactically right. Formatting the Go code is provided by the Go basic tools (see: [format](https://golang.org/pkg/go/format/)). You can run the **fmt** tool using command: “go fmt github.com/karimarttila/go/simpleserver/app/…”.

### Static Code Analysis

Go provides a simple static code analysis tool in the standard Go toolbox: [**vet**](https://golang.org/cmd/vet/).

I also found another interesting tool: [Staticcheck](https://staticcheck.io/docs/). Install the package and you can run various tools like staticcheck, gosimple and unused in one go for all Go code files: using command: “megacheck github.com/karimarttila/go/simpleserver/app/…”

Staticcheck open source version is free. If you find the tool useful you should consider buying the commercial version.

### Http and REST Handling

Go is an unusual language in that sense that it provides excellent standard library and therefore you seldom need to import extra dependencies. There are some external http routing libraries in Go, e.g. [Gorilla](https://github.com/gorilla/mux) but why to introduce external dependencies to your project if you can manage with the standard library? So, for http / REST handling I just used the Go standard library [net/http](https://golang.org/pkg/net/http). It was pretty straightforward to use net/http, see example below.

func getProductGroups(writer http.ResponseWriter, request *http.Request) {  
 util.LogEnter()  
 writeHeaders(writer)  
 if request.Method == “OPTIONS” {  
 return  
 }  
 parsedEmail, errorResponse := isValidToken(request)  
 var productGroups domaindb.ProductGroups  
 if !errorResponse.Flag {  
 util.LogTrace(“parsedEmail from token: “ + parsedEmail)  
 productGroups = domaindb.GetProductGroups()  
 encoder := json.NewEncoder(writer)  
 encoder.SetEscapeHTML(false)  
 err := encoder.Encode(productGroups)  
 if err != nil {  
 errorResponse = createErrorResponse(err.Error())  
 }  
 }  
 if errorResponse.Flag {  
 writeError(writer, errorResponse)  
 }  
 util.LogExit()  
}### Error Handling

I kind of like Go’s error handling. I have done production software with C some 20 years ago and you always had to be pretty careful with returned error codes. In C++ you could define exceptions and throwing and catching them which kind of simplified error handling but also with a certain price. Java adopted the exception strategy but divided exceptions to [runtime exceptions](https://docs.oracle.com/javase/10/docs/api/index.html?java/lang/RuntimeException.html) which you didn’t have to explicitly handle and [checked exceptions](https://docs.oracle.com/javase/10/docs/api/index.html?java/lang/Exception.html) which you had to explicitly handle — many consider this as a failed experiment since e.g. [Spring](https://spring.io/) exclusively uses just runtime exceptions. Go has a different strategy. There are no exceptions in the language (well, there is one exception — [panic](https://blog.golang.org/defer-panic-and-recover)) but in functions you can return many return values. An idiomatic way is to return the actual return value and an error — if there are no errors then the error value is nil (or e.g. you can use your own error response and set a flag), if there were errors the error value provides indication of the error. This is kind of nice but once again comes with a price — makes the error handling more explicit but creates more manual work for the programmer to handle errors.

It is pretty tedious to format code in Medium, so I advice readers to read the longer story in the project [README.md](https://github.com/karimarttila/go/tree/master/simpleserver) file.

### Go Interfaces

Go interfaces are actually pretty nice minimalist way to provide abstraction and reuse to Go code. See an example in [server.go](https://github.com/karimarttila/go/blob/master/simpleserver/app/webserver/server.go). Also there is a longer chapter regarding Go Interfaces in the [README.md](https://github.com/karimarttila/go/tree/master/simpleserver) file if you are interested.

### Testing

Go is pretty amazing in that sense that you have also [the Go testing framework](https://golang.org/pkg/testing/) in the standard library.

Go tests are pretty easy to create. You don’t have asserts but instead you just write standard application logic to test whether your package works as expected. Example:

func TestGetProductGroups(t *testing.T) {  
 util.LogEnter()  
 port := util.MyConfig[“port”]  
 token, err := CreateJsonWebToken(“[kari.karttinen@foo.com](mailto:kari.karttinen@foo.com)”)  
 if err != nil {  
 t.Errorf(“Failed to get test token: %s”, err.Error())  
 }  
...  
 response := recorder.Body.String()  
 if len(response) == 0 {  
 t.Error(“Response was nil or empty”)  
 }  
 pgMap := make(map[string]map[string]string)  
 err = json.Unmarshal([]byte(response), &pgMap)  
 if err != nil {  
 t.Errorf(“Unmarshalling response failed: %s”, err.Error())  
 }  
 pg, ok := pgMap[“product-groups”]  
 if !ok {  
 t.Errorf(“Didn’t find ‘product-groups’ in response”)  
 }  
 pg1, ok := pg[“1”]  
 if !ok {  
 t.Errorf(“Didn’t find product group 1 in response”)  
 }  
 if pg1 != “Books” {  
 t.Errorf(“Product group 1 should have been ‘Books’”)  
 }  
 util.LogEnter()  
}You can run all the tests with command: “go test github.com/karimarttila/go/simpleserver/app/…”

### Map, Reduce and Filter

There are no map, reduce and filter implementations in the Go standard library because Go is a statically typed language which does not provide generics — you either should have a dynamically typed language (like Clojure, Javascript or Python) or a statically typed language with generics (like Java) to have real map, reduce and filter functions. I googled this a bit and found one of Go’s inventors, Rob Pike’s [filter](https://github.com/robpike/filter) implementation in which he says:


> “I wanted to see how hard it was to implement this sort of thing in Go, with as nice an API as I could manage. It wasn’t hard. Having written it a couple of years ago, I haven’t had occasion to use it once. Instead, I just use “for” loops. **You shouldn’t use it either.**”So, let’s just use for loops while programming Go. This is a bit of a pity since map, reduce and filter are a good abstraction and very idiomatic e.g. in functional languages like Clojure. But you just have to accept that when in Rome do as the Romans do.

### Go Playground

There is no REPL in Go (very difficult to make for a statically typed language — it took some 20 years before we got some sort of very simple REPL for Java).

But there is some sort of workaround: [Go Playground](https://play.golang.org/). Gophers have created various examples in the Playground and you can try to google them. Example: [How to use map of maps](https://play.golang.org/p/pQsoB02pDl).

### Logging

My good friend and Go guru Tuomo Varis told me not to use external libraries but to do everything using Go standard library (to learn it better). I considered this a moment but first decided not to follow his good recommendation. The rationale being that I wanted to quickly implement the core functionalities of a web server and e.g. not to reinvent a logging framework myself. Therefore I first started to use one of the most used Go logging frameworks, [Logrus](https://github.com/sirupsen/logrus).

But when discussing with Tuomo he convinced me that implementing a simple logger based on the Go standard library logger should be a rather simple task. So, I took the challenge and implemented my own custom [logger.go](https://github.com/karimarttila/go/blob/master/simpleserver/util/logger.go) based on the Go standard library logger. Basically I just implemented various log levels, my custom function entry/exit logging and some custom formatting of log entries.

### Productivity

Go productivity is not as good as in Python (Simple Server implementation took some 3 evenings in Python) and in Clojure (I would now do it in three evenings using Clojure), but Go productivity is much better than in Java (took some 3 weeks in Java even though I have programmed some 20 years of Java) and Javascript (took me also some 3 weeks and I had to learn the language and Node and all related tools while implementing Simple Server). I did the Go implementation in about 8 evenings. So, Go’s productivity seems to be somewhere between these languages. I’ll write a deeper analysis in the next blog post in which I compare these languages. If you are anxious to read more about this you can take a peek at the [README.md](https://github.com/karimarttila/go/tree/master/simpleserver) file in which I have written down some notes for the next blog article as well.

### Performance

Go performance is excellent since Go compiles to bare metal and there is no runtime between the metal and the language as is the case in other languages I used in this exercise (Java & Clojure: compile to [JVM bytecode](https://en.wikipedia.org/wiki/Java_bytecode), and [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) runs the bytecode, Python & Javascript: interpreted languages and the runtime runs the code (though also Python compiles to bytecode).

Go also compiles extremely fast. This was a major design driver when Google designed Go — it had to compile fast since there were major compile time issues with C and C++ because of Google’s huge code base (read more in Rob Pike’s excellent article “[Go at Google: Language Design in the Service of Software Engineering](https://talks.golang.org/2012/splash.article)”, especially chapter 5).

### Concurrency Support

One thing that I did’n have chance to use in this exercise is the concurrency support provided by the Go language. I read about Go’s concurrency support and it seems to be pretty good. The language provides a simple abstraction — goroutines and channels — for concurrency. There is a nice saying among Gophers: “Do not communicate by sharing memory; instead, share memory by communicating.” (see Go blog: “[Share Memory By Communicating](https://blog.golang.org/share-memory-by-communicating)”). I.e. in the Java world if you want your threads to share something you share memory and you have to use Java’s synchronization mechanisms to provide multi-thread safe execution. In Go another strategy is used — various entities are collaborating concurrently by sending messages using channels. This is something I definitely want to delve deeper. Clojure also provides pretty good concurrency support since language is immutable by default and you share entities by certain language primitives (like atoms).

### Conclusions

I fell in love with Go. Go is really a very concise and productive language if you need a robust and performant statically typed language with excellent concurrency support. Much better than Java which compared to Go is verbose, non-productive and concurrency support is far behind Go. I have done quite a lot of C++ and Java programming and I must say that Go’s error handling with idiomatic error entity as paired with the actual return value from functions is really great and simple. Go is definitely going to be my choice of statically typed language in my future projects. But still, if I need to create a quick script, e.g. a surrogate script for handling aws cli calls and process returned json — I will choose Python. And if I need to process a lot of data — Clojure. But when you need statically typed language and excellent performance with great concurrency support — Go.

I started my programming career some 25 years ago with programming C. Hacking Go is a bit like coming home, except you don’t need to be meticulous with memory allocation / deallocation. I think Go gives all the goodies from C programming but takes care of the heavy lifting of what’s difficult in C. Go code is really simple and elegant — the language provides the exact support for those things that you really need and doesn’t add anything extra to the language (like Einstein put it: “Everything should be made as simple as possible, but not simpler”).

The feeling was actually quite amazing. I started my Go hacking with practically zero Go knowledge on Monday, and already on Saturday I felt like all pieces of the puzzle just locked in to the right places and creating code was really fluent and easy — and next Monday evening a week later when I started my Go exercise I was already done. A programming language should be that easy — Go is.

  