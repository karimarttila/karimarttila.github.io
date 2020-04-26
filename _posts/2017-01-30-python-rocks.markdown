---
layout:	post
title:	"Python Rocks!"
categories: [blog, aws]
tags: [aws]
date:	2017-01-30
---

  ![](/img/1*PPIp7twJJUknfohZqtL8pQ.png)[© Python Software Foundation](https://www.python.org/community/logos/)I have used Python for all kinds of short tasks for almost 20 years. Python is a great object-oriented dynamic scripting language which also scales for more complex tasks. I actually never bothered to master shell scripts since using Python as a shell wrapper has been so easy. If you realize that your script needs more functionality it is also a lot easier to use a full-fledged programming language than a shell script to implement more complex application logic.

**For What Kind of Tasks Python is Good for?**

I have used Python for many years for various tasks. Examples are:

* **Log Analyzers**. Sometimes there are cryptic problems in production environments that are hard to duplicate in development environments. A good way to find these problems is to configure the production to use debug level logging to get a snapshot what is happening and then investigate the logs for certain patterns. I have written short scripts that read log files and record certain events that I’m interested of and finally write down a summary of these events, how they are connected together and if there are any anomalies in the behavior related to those events.
* **Pollers**. It is pretty easy to write various queue and process pollers using Python.
* **String Manipulators**. This is one of my my favorite ways to use Python. Python has great regex capabilities and they are very easy to use. E.g. once I needed to gather all Finnish terms from resource bundles and generate an Excel to give it to the translator for Swedish translations. It was pretty easy with Python to walk through all resource bundles and create a CSV file with the required format (which then could be imported to Excel).
* **Rapid Prototyping**. I have used Python to create quick frontend and backend prototypes that we needed to demonstrate to our directors or clients before some investment decision was made.
* **Mock Servers. **I have used Python to create mock servers providing REST interface for front-end development. Once I implemented a frontend using Angular and I was also supposed to implement the backend with Java. I created the backend mock first using Python and I implemented the frontend which we iterated with the customer so that they were happy regarding the UI and its functionality. After the UI was pretty ready it was easy to implement the same REST using Java / Spring Boot and switch the frontend to use the real backend instead of the mock.
* **Test Data Generation**. In my latest project I was responsible of generating test data to AWS RDS before we got test data resetters using real data uploader applications implemented. Test data generation - perfect job for Python and its excellent string manipulation capabilities.
* **AWS CLI Integration**. In my latest project I have used Python as an [AWS CLI](https://aws.amazon.com/cli/) wrapper. It’s easy to use AWS CLI either using [Python SDK for AWS](https://aws.amazon.com/sdk-for-python/) directly or for shorter tasks just call AWS CLI using Python OS integration modules and read the returned JSON for further processing (and Python provides excellent JSON library for parsing JSON).
And a short example for quick AWS CLI hacking using OS call:

![](/img/1*oNGIMWtR0ga11XXrMXe-XQ.png)Python Example to Call AWS CLI.**Why Do I Love Python?**

The reason why I most love Python is that it is so easy. The syntax is really easy and the creator of the language, Guido van Rossum, wanted to make a language that is also easy to read (see more about the Python design decisions [here](http://python-history.blogspot.fi/2009/01/pythons-design-philosophy.html)). The easiness of the language makes it easy to be productive with the language even if you have been using other programming languages in your project for months and you suddenly need something quick to manipulate e.g. some strings or you realize that it would be nice to have a tool to restart certain AWS EC2 auto-scaling groups based on some criteria.

I also love the [Python REPL](https://docs.python.org/3.6/tutorial/interpreter.html) (read-eval-print-loop). In those languages that implement a good REPL (like Python, LISP languages (like [Clojure](https://clojure.org))…) the REPL is used for incremental program development: you experiment something small program fraction in REPL and once you are happy you integrate it to your program.

If you are interested, check it out: [https://www.python.org](https://www.python.org/) .

How have you been using Python? Share your experiences in comments!

  