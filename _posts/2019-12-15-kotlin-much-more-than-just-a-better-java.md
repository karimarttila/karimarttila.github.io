---
layout: post
title: "Kotlin - Much More Than Just a Better Java"
category: [languages]
tags: [languages, kotlin]
date: 2019-12-15
---

### Introduction

This [Kotlin](https://kotlinlang.org/) version of my exercise server “Simple Server” is now the sixth language that I have used to implement the same server as an exercise (previous implementations: Clojure, Javascript, Java, Python and Go, see: my previous blog post regarding these exercises: [Five Languages - Five Stories]({% post_url 2018-11-19-five-languages-five-stories %}). I’m planning to implement the Simple Server also using Typescript — maybe after that implementation I update that blog post to “Seven Languages - Seven Stories" :-).

You can find the project in [Github](https://github.com/karimarttila/kotlin/tree/master/webstore-demo/simple-server).

Once again I tried to replicate the file/namespace/class names so that it is easy to compare the implementations (e.g. [server.kt](https://github.com/karimarttila/kotlin/blob/master/webstore-demo/simple-server/src/main/kotlin/simpleserver/webserver/server.kt) — [server.py](https://github.com/karimarttila/python/blob/master/webstore-demo/simple-server/simpleserver/webserver/server.py) — [server.js](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/src/webserver/server.js) — [server.clj](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/src/simpleserver/webserver/server.clj) — [Server.java](https://github.com/karimarttila/java/blob/master/webstore-demo/simple-server/src/main/java/simpleserver/webserver/Server.java) — [server.go](https://github.com/karimarttila/go/blob/master/simpleserver/app/webserver/server.go)).

![](/img/2019-12-15-kotlin-much-more-than-just-a-better-java_img_1.png)

*Kotlin Code in IntelliJ IDEA.*

### Tools

I used the following tools in this exercise:

* [OpenJdk 11](http://openjdk.java.net/): openjdk 11.0.4
* [Gradle](https://gradle.org/): 5.6.3 / Kotlin: 1.3.41 / Groovy: 2.5.4
* [Kotlin](https://kotlinlang.org/): 1.3.50
* [Ktor](https://ktor.io/): 1.2.6


### Kotlin vs. Java

I actually started to learn Kotlin already some four years ago. But after a short introduction I thought that “Kotlin is just Java done right” and decided to deep dive into Clojure instead (which actually was a good decision — Clojure is an excellent language and very different when compared to Java and Kotlin). But this autumn I was “forced” to deep dive to Kotlin as well: I worked this autumn with an interesting customer which had a team of extremely competent developers who were also a bunch of real language enthusiastics. They had implemented many microservices using different languages, one of the languages being Kotlin. In that assignment I had a chance to convert one old Java application to Kotlin. Therefore I had a good reason to learn Kotlin and I also decided to implement my exercise server using Kotlin.

After doing some diving into Kotlin I realized that Kotlin actually is much more than just “Java done right”. Kotlin is a very well designed language and you can use Kotlin quite naturally either using object-oriented paradigm or using functional paradigm. I decided to implement this Kotlin version of Simple Server using functional paradigm to compare this side also to Clojure.

My main impressions regarding Kotlin as a language are:

* **Kotlin is really easy**. If you have used Java you have absolutely no issues to learn Kotlin. Kotlin has actually simplified creating backend systems using a JVM language quite a bit compared to Java.
* **Kotlin is pretty concise**. Example: data class User(val email: String, val firstName: String, val LastName: String, val password: String), i.e. you have a valid data class ready to be used without any Java getter/setter boilerplate. Since Kotlin is both easy and concise new developers will be productive in a short period of time.
* **Expressions**. I really liked Kotlin’s idea that most of the things that are statements in Java (if-else, try-catch…) are expressions in Kotlin (i.e. you return values from those constructs) — this idea supports a lot using Kotlin as a functional language.
* **Mixing Java and Kotlin**. You can mix Java and Kotlin files without any extra boilerplate in your project. This makes migration from Java to Kotlin extremely easy: just convert the files one at a time as a need basis. I also tried IntelliJ IDEA’s Java-to-Kotlin migration tool and it was astonishingly good — there were only a few cases where the tool couldn’t figure out how to do the conversion. See: [Mixing Java and Kotlin in one project](https://kotlinlang.org/docs/tutorials/mixing-java-kotlin-intellij.html) in Kotlin documentation.
* **Null pointer safety**. Kotlin provides language constructs to avoid most of the null pointer exception errors. See: [Null Safety](https://kotlinlang.org/docs/reference/null-safety.html) in Kotlin documentation.
* **IDE tooling**. IntelliJ IDEA is a great Kotlin IDE, of course since the same company — JetBrains — is behind the language and the IDE. See: [Getting Started with IntelliJ IDEA](https://kotlinlang.org/docs/tutorials/getting-started.html) in Kotlin documentation.
* **Gradle and Kotlin**. You can use Kotlin as a Gradle domain specific language (DSL). See: [Using Gradle](https://kotlinlang.org/docs/reference/using-gradle.html) in Kotlin documentation.
* **Lesser need for exceptions**. Kotlin provides nice constructs that you can use to pass information e.g. whether you have found results for some query or not using sealed class. See [Sealed Classes](https://kotlinlang.org/docs/reference/sealed-classes.html) in Kotlin documentation for more information. Example:

```kotlin
sealed class ProductsResult
data class ProductsFound(val data: List<Product>) : ProductsResult()
object ProductsNotFound :
    ProductsResult()Then you can use these sealed classes quite nicely to process various happy-day scenarios and exceptional scenarios, example:

val ret = when (val products = getProducts(pgId)) {
    is ProductsNotFound -> ProductNotFound
    is ProductsFound -> {
        when (val p = products.data.firstOrNull { it.pId == pId }) {
            null -> ProductNotFound
            p -> ProductFound(p)
            else -> ProductNotFound
        }
    }
}
```

 This makes code pretty readable: “If you didn’t find any products you cannot find an individual product. If you found products you can try to find an individual product by pId field.” Because when is an expression you finally return some value to variable ret.

### Kotlin vs. Clojure

When comparing Kotlin to Clojure my first impression is however: “Kotlin is pretty good but compared to Clojure Kotlin is just Java done right”. With this statement I mean that Clojure being a Lisp has all the power of Lisp including a real REPL and a real functional language with immutability by default, and excellent data oriented language structures and standard library to process data. If I started a new project with competent developers who already know Clojure or are willing to learn it my choice of language would definitely be Clojure since with competent developers Clojure (Lisp) is like a [secret weapon](http://www.paulgraham.com/avg.html) when considering developer productivity. But if Clojure is not an option and I had to choose either Java or Kotlin I would definitely choose Kotlin. So, considering JVM development I’m starting to think that there is no going back to Java. Sorry Java, you had your glory days but now it is time for new guys to continue the JVM journey.

  
