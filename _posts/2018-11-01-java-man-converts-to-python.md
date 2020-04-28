---
layout:	post
title:	"Java Man Converts to Python"
categories: [blog, aws]
tags: [aws]
date:	2018-11-01
---

### Prologue

Three evenings versus three weeks.

### Introduction

This is the fourth blog article in my series to implement the same web server in five different languages. The previous articles are:

* **Clojure**: “[Become a Full Stack Developer with Clojure and ClojureScript!](https://medium.com/@kari.marttila/become-a-full-stack-developer-with-clojure-and-clojurescript-c58c93479294)”
* **Javascript/Node**: “[Java Man’s Unholy Quest in the Node Land](https://medium.com/@kari.marttila/java-mans-unholy-quest-in-the-node-land-958e61da0451)”
* **Java**: “[Java Man Returns Home Confused](https://medium.com/@kari.marttila/java-man-returns-home-confused-2fe951eb51a9)”
This blog article is about my fourth language, [**Python**](https://www.python.org/). I didn’t do this exercise to learn Python — I have been programming Python some 20 years now. I wanted to implement the same web server just to see how fast I can implement it compared to previous Java implementation. The result was about 3 evenings (Python) — 3 weeks (Java). A sad story for Java, a great victory for Python.

You can find the project in [**Github**](https://github.com/karimarttila/python/tree/master/webstore-demo/simple-server).

Once again I tried to replicate the file/namespace/class names so that it is easy to compare the implementations (e.g. [server.py](https://github.com/karimarttila/python/blob/master/webstore-demo/simple-server/simpleserver/webserver/server.py "server.py") — [server.js](https://github.com/karimarttila/javascript/blob/master/webstore-demo/simple-server/src/webserver/server.js) — [server.clj](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/src/simpleserver/webserver/server.clj) — [Server.java](https://github.com/karimarttila/java/blob/master/webstore-demo/simple-server/src/main/java/simpleserver/webserver/Server.java)).

![](/img/2018-11-01-java-man-converts-to-python_img_1.png)PyCharm hacking session running server unit tests

### Tools

I used [Python](https://www.python.org/ "https://www.python.org/") 3.6.6 on Ubuntu18 when implementing this Simple Server. I used Python virtual environment to keep my Ubuntu installation clean:

python3 --version => Check your Python version (mine was 3.6.6).  
virtualenv --version => Check your virtual env version (mine was 16.0.0).  
which python3 => Where is python3? (mine is in /usr/bin/python3).  
virtualenv -p /usr/bin/python3 venv => Create the virtual env.  
source venv/bin/activate => Activate your virtual env  
python3 --version? => Python 3.6.6 (in this virtual envinronment).  
pipenv install flask => Use pipenv to install packages.  
ls -l Pipfile* => List the generated Pipfile(s).  
deactivate => Leave the virtual environment.And so we finalized our short tour to “Python virtual environment and package management.”

For those who don’t know Python land I tell that there are two major versions of Python: 2 and 3 — and they are not compatible. The recommendation is always to use the newer version 3 if possible.

For accelerating the REST implementation I decided to use [Flask](http://flask.pocoo.org/ "http://flask.pocoo.org/") (version 1.0.2) since it is widely-used Python microframework for that purpose.

### PyCharm

I use [PyCharm](https://www.jetbrains.com/pycharm "https://www.jetbrains.com/pycharm") for Python programming (on remote *nix machines [Emacs](https://www.gnu.org/software/emacs/)). PyCharm is really good IDE for Python programming — the editor is great and there are a lot of utilities that make your Python programming more productive (code completion, test runners, automatic linter ([PEP](https://www.python.org/dev/peps/pep-0008/ "https://www.python.org/dev/peps/pep-0008/")), great debugger, Python console (REPL), etc).

I use [IntelliJ IDEA](https://www.jetbrains.com/idea/ "https://www.jetbrains.com/idea/") for Java programming and since PyCharm and IDEA are provided by the same company (JetBrains) they provide very similar look-and-feel. I also use IntelliJ IDEA with [Cursive](https://cursive-ide.com/ "https://cursive-ide.com/") plugin for Clojure programming and it also provides very similar look-and-feel. So, there are a lot of synergy benefits to use the same IDE for several programming languages.

### Linting

[PEP8](https://www.python.org/dev/peps/pep-0008/ "https://www.python.org/dev/peps/pep-0008/") provides a definitive style guide for Python programming. PyCharm provides nice automatic linting of Python code related to this style guide.

### Testing

The tests are implemented using [**pytest**](https://docs.pytest.org/en/latest/ "https://docs.pytest.org/en/latest/"). Pytest is pretty straightforward to use and PyCharm also provides nice integration to unit tests implemented using pytest (debugger etc.). In PyCharm just create a new pytest configuration and configure it to use your specific unit test file and you are good to go to run that test and the code it calls in your debugger.

Let’s compare the test performances between different implementations when running the whole test suite in console:

**Clojure**:

time ./run-tests.sh   
19:52:18.637 [main] INFO simpleserver.util.prop - Using poperty file: resources/simpleserver.properties  
lein test simpleserver.testutils.users-util  
lein test simpleserver.userdb.users-test  
lein test simpleserver.webserver.server-test  
lein test simpleserver.webserver.session-test  
Ran 11 tests containing 47 assertions.  
0 failures, 0 errors.  
real 0m6.027s**Java**:

$ time ./gradlew --rerun-tasks test  
Test result: SUCCESS  
Test summary: 15 tests, 15 succeeded, 0 failed, 0 skipped  
BUILD SUCCESSFUL in 5s  
5 actionable tasks: 5 executed  
real 0m5.757s**Javascript**:

time ./run-tests-with-trace.sh  
 28 passing (94ms)  
real 0m0.775s**Python**:

time ./run-pytest.sh   
========================================== test session starts   
platform linux -- Python 3.6.6, pytest-3.9.3, py-1.7.0, pluggy-0.8.0  
rootdir: /mnt/edata/aw/kari/github/python/webstore-demo/simple-server, inifile: setup.cfg  
collected 14 items   
tests/domaindb/test\_domain.py .... [ 28%]  
tests/userdb/test\_users.py ... [ 50%]  
tests/webserver/test\_server.py ...... [ 92%]  
tests/webserver/test\_session.py . [100%]======================================= 14 passed in 0.11 seconds   
real 0m0.416s**The results are:**

![](/img/2018-11-01-java-man-converts-to-python_img_2.png)It’s pretty obvious that Clojure and Java lose the contest because of the loading of JVM. But I was surprised that Python runs the tests that fast.

### Python REPL

Python REPL is one of the best REPLs I have used outside the Lisp world. In the Lisp world REPLs are real REPLs which allow you to experiment with the live system in ways no other REPL or debugger lets you do in other languages — it’s pretty impossible to explain this, you just have to learn some Lisp and try yourself (e.g. [Clojure](https://clojure.org/ "https://clojure.org/")). So, now that we have gone through my mandatory Clojure advertisement let’s go back to Python REPL. Compared to JShell Java REPL Python wins the fight hands down. Python REPL came with the first version of Python (we just had to wait some 20 years for the Java REPL) and because Python is dynamically typed language the REPL is pretty easy to use (compared to Java JShell which is really awkward to use, even with a good IDE).

PyCharm provides a nice REPL, an example follows:

>>> runfile('/mnt/edata/aw/kari/github/python/webstore-demo/simple-server/simpleserver/domaindb/domain.py', wdir='/mnt/edata/aw/kari/github/python/webstore-demo/simple-server')  
>>> myD = Domain()  
2018-10-30 18:40:11,769 - \_\_main\_\_ - \_\_init\_product\_db - DEBUG - ENTER  
2018-10-30 18:40:11,770 - \_\_main\_\_ - \_\_read\_product\_groups - DEBUG - ENTER  
...  
2018-10-30 18:40:11,771 - \_\_main\_\_ - \_\_read\_raw\_products - DEBUG - EXIT  
2018-10-30 18:40:11,771 - \_\_main\_\_ - \_\_init\_product\_db - DEBUG - EXIT  
>>> myD.get\_raw\_products(1)  
[['2001', '1', 'Kalevala', '3.95', 'Elias Lönnrot', '1835', 'Finland', 'Finnish'], ...]So, using the runfile method you are able to reload any module to PyCharm Python console and then try the methods there in isolation.

### Logging

What a relief Python logging configuration is after Spring hassle. You just create the [logging.conf](https://github.com/karimarttila/python/blob/master/webstore-demo/simple-server/logging.conf "https://github.com/karimarttila/python/blob/master/webstore-demo/simple-server/logging.conf") file and that’s about it.

### Readability

Python wins Javascript and pretty much any language in readability hands down. It is probably the most readable language there is. I would say that it is even more readable than Clojure which is also a very readable language (once you get used to read a functional language). Java loses to Python in readability just because of the monstrous verbosity of the language.

Let’s use Javascript and Python implementations as an examples of readability of those languages (you can check equivalent examples of Java and Clojure in my previous blog posts, see links in the beginning of this article):

**Javascript**:

describe('GET /product-groups', function () {  
 let jwt;  
 it('Get Json web token', async () => {  
 *// Async example in which we wait for the Promise to be*  
 *// ready (that i.e. the post to get jwt has been processed).*  
 const jsonWebToken = await getJsonWebToken();  
 logger.trace('Got jsonWebToken: ', jsonWebToken);  
 assert.equal(jsonWebToken.length > 20, true);  
 jwt = jsonWebToken;  
 });  
 it('Successful GET: /product-groups', function (done) {  
 logger.trace('Using jwt: ', jwt);  
 const authStr = createAuthStr(jwt);  
 supertest(webServer)  
 .get('/product-groups')  
 .set('Accept', 'application/json')  
 .set('Authorization', authStr)  
 .expect('Content-Type', /json/)  
 .expect(200, {  
 ret: 'ok',  
 'product-groups': { 1: 'Books', 2: 'Movies' }  
 }, done);  
 });  
 });**Python**:

def test\_get\_product\_groups(client):  
 myLogger.debug(ENTER)  
 token = get\_token(client)  
 decoded\_token = (b64encode(token.encode())).decode()  
 mimetype = 'application/json'  
 headers = {  
 'Content-Type': mimetype,  
 'Accept': mimetype,  
 'Authorization': 'Basic ' + decoded\_token  
 }  
 url = '/product-groups'  
 response = client.get(url, headers=headers)  
 assert response.status == '200 OK'  
 json\_data = json.loads(response.data)  
 assert json\_data.get('ret') == 'ok'  
 assert b"product-groups" in response.data  
 product\_groups = json\_data.get('product-groups')  
 assert product\_groups['1'] == 'Books'  
 assert product\_groups['2'] == 'Movies'  
 myLogger.debug(EXIT)I would say that Python is more readable.

### Productivity

What a joy it was to program Python after Java. Dynamically typed language! Concise! Clear syntax! Simple! The productivity of Python programming compared to Java is like from another planet — I explored PyCharm new features and Flask the first evening and in the second evening I implemented the domaindb module and most of the userdb module and related unit tests. The third evening I implemented the rest of the application and the unit tests and that was it. There were absolutely no hassle in Python code, configurations or in IDE (PyCharm).

Python (and especially PyCharm) **REPL** is definitely the best REPL I have used outside the Lisp world. The Python REPL makes exploring small code snippets a breeze.

Using **PyCharm debugger** is also so easy and fast. If you have even minor issues in your code you tend to add a breakpoint and hit the debugger. This is actually pretty interesting since in the Lisp world you hardly ever use the debugger — you tend to have a live REPL to your system while you add new functionalities. You can’t have a live REPL to your Python system in the same sense but PyCharm debugger is a pretty good second option. And when you compare Python debugger to Java debugger — Python is lightning fast to start. Creating Run configurations for your unit tests in PyCharm is also very easy and straightforward. PyCharm debugger is also a great tool to check what’s inside various entities (e.g. I just earlier used the debugger to check where the http status code is inside the Flask response entity and what its name is) — if you are lazy to search that information in the library API documentation. See an example below.

![](/img/2018-11-01-java-man-converts-to-python_img_3.png)Adding breakpoint and hitting debugger in PyCharm.In general I think Python must be the most productive language I have ever used. Clojure might win the case in productivity after a couple of years of serious Clojure hacking but Python is unbeatable in the scripting category — you may have months of gaps between your Python hacking sessions but the language is always easy to put into real work regardless how long it was you programmed Python the last time.

If you compare Python to Java — Python wins hands down. Java is verbose — Python is concise. Java has long development cycle (edit, compile, build, load to JVM, run) — Python has short development cycle (edit, run). Java has difficult syntax — Python has very easy syntax.

If you compare Python to Javascript/Node, Python wins in clean syntax and overall easiness to create and test code.

There is nothing inherently bad in Python. I would be cautious to use Python in a big project with tens of developers working in the same code base unless you have some strict rules how to protect collaboration from the typical mistakes using dynamically typed language in a big project (e.g. mandatory use of [type hints](https://docs.python.org/3/library/typing.html "https://docs.python.org/3/library/typing.html")).

### Lines of Code

The final contest! Lines of Code! Let’s once again compare the lines of code between different implementations of the Simple Server (production code, i.e. not including unit tests):

![](/img/2018-11-01-java-man-converts-to-python_img_4.png)If you drop the empty package files (‘\_\_init\_\_.py’) there are only 8 source code files in the production source tree and altogether only 582 lines of code. So, it seems that Python is the winner of this part of the contest. What could be a better language than one with a small amount of code and even that small amount of code being very simple, concise and easy to read.

### Performance

The [Global Interpreter Lock](https://wiki.python.org/moin/GlobalInterpreterLock) (GIL) might cause some issues if you try to create a system which should be responsive to a large amount of events / sessions. Node is also single-threaded but Node has a special architecture in which Node runs single thread in an event loop and delegates e.g. I/O work to worker pool threads. This makes Node extremely efficient in handling tasks which are not CPU intensive (on the other side CPU intensive tasks may degrade the Node performance quite a lot). Java system on the contrary typically spins a dedicated thread for each request. This is more expensive (consumes more machine resources) but one thread (for one client) does not block processing of another thread (client). Python has the infamous Global Interpreter Lock which has generated a lot of debate in the Python community during Python’s lifetime. In most cases this is not a problem since you usually use Python for small tasks. But if you use Python for CPU intensive work handling a huge set of tasks or requests in parallel you have to find some special solutions for it (and those do exist if you google them, see e.g. [“Efficiently Exploiting Multiple Cores with Python”](http://python-notes.curiousefficiency.org/en/latest/python3/multicore_python.html "http://python-notes.curiousefficiency.org/en/latest/python3/multicore_python.html")).

### Conclusions

I have used Python for some 20 years for various tasks. E.g. my personal backup scripts at home are implemented using Python. My virtual dog was implemented using Python (watched my IP cameras and started an audio file of an insane dog barking if any movement detected at my backyard — I can reveal this now since I have nowadays a real big insane dog watching the house). At work I have used Python for various log analysis work, watching processes, analyzing copy-pasting of Java classes between projects, gluing various [aws cli](https://aws.amazon.com/cli/) commands together and so on and so on. Python is really a good language for quick ad hoc scripts you may need for various purposes. Python is also pretty good language to implement a web server as I now did (performance testing is to be done).

This exercise to implement the same web server using five languages (one still to go) has been a real eye opener. I must say that I’m pretty disappointed regarding Java’s productivity. I thought that after 20 years I could have implemented the web server faster than with a totally new language (Javascript), but it took the same amount of time (about 3 weeks) for both of them. Python — 3 evenings. Now that those big enterprise monolithic systems are going to be history and the new era of Cloud serveless and microservices are emerging one can choose the language freely. I think I leave Java in that big monolithic enterprise world where it belongs. The new serverless implementations are best to implement using Python or Javascript. Data oriented microservices possibly using Clojure.

The last challenge is still ahead: [Go](https://golang.org/ "https://golang.org/"). It’s going to be pretty interesting since I know nothing of Go. So, one more time I will raise the old Simple Server ghost from the underworld and mold it into an implementation — this time using Go.

  