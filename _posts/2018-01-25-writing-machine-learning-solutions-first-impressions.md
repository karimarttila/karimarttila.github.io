---
layout:	post
title:	"Writing Machine Learning Solutions — First Impressions"
categories: [blog, aws]
tags: [aws]
date:	2018-01-25
---

  ![](/img/2018-01-25-writing-machine-learning-solutions-first-impressions_img_1.png)Linear regression model created using TensorFlow

### Introduction

As I described in my previous blog article [Studying Machine Learning — First Impressions](https://medium.com/@kari.marttila/studying-machine-learning-first-impressions-a8008d68b847) I decided to dedicate this winter’s studies in Machine learning. I have been doing the excellent Machine learning [Coursera](https://www.coursera.org/) course and working with the exercises using [Octave](https://www.gnu.org/software/octave/). This was basically the first step (theory) in my Machine learning study path.

Now that I finally after many weeks managed to walk through all material and exercises in that Coursera course I decided to do the same exercises using some industry level ML library (step two in my Machine learning study path). I evaluated the libraries and finally decided to start using [TensorFlow](https://www.tensorflow.org/). Why TensorFlow? TensorFlow seems to be one of the most used ML libraries. You can create the models using [Python](https://www.python.org/) which makes the model creation pretty easy (using the Python REPL) if you are fluent in Python (see my previous [Python Rocks!](https://medium.com/tieto-developers/python-rocks-5dc453b5c222) blog article). And the good side is that you can use the ML model (you created using Python) in [Java](https://www.java.com/en/), and therefore also in [Clojure](https://clojure.org/) — my current choice of programming language (see more in my previous blog article [Clojure Impressions Round Two](https://medium.com/tieto-developers/clojure-impressions-round-two-f989c0945f4b)).

### You Also Need Other Libraries When Creating ML Models…

When I was doing the first exercise [Linear regression](https://en.wikipedia.org/wiki/Linear_regression) in TensorFlow I soon realized that you have to learn two other Python libraries as well:

* [NumPy](http://www.numpy.org/) (scientific computing with Python) — without NumPy various matrix operations would be rather painfull, and ML is basically mostly rolling various matrices).
* [Matplotlib](https://matplotlib.org/) (Python plotting library for creating graphics). It is important to visualize the data you are working with, and when working in Python Matplotlib is your friend. The graphic in the beginning of this blog article was generated using Matplotlib.
These libraries are not that difficult to learn, you can basically learn to use them on the fly when you are working with your ML model. And remember: Python REPL is your friend while learning any Python library.

### Python REPL

Let’s add a short Python REPL session example to visualize how you can use Python REPL to learn a new Python library and experiment with it before writing the actual code in your Python editor (by the way, my choice of Python IDE is excellent [PyCharm](https://www.jetbrains.com/pycharm/)).

>>> import src.ml\_course\_ex1a as ex1a  
>>> model = ex1a.ProfitPopulationLinearRegression("ml\_course\_ex1a.ini", True)  
>>> (populations,profits) = model.readCsvFile("data/ex1a-profit-population.csv")  
>>> len(populations)  
97  
>>> import numpy as np  
>>> populations\_array = np.asarray(populations)  
>>> populations\_array  
array([ 6.1101, 5.5277, 8.5186, 7.0032, 5.8598, ...   
>>> X\_train = np.asarray(populations)  
>>> X\_train\_bias = model.appendBias(X\_train)  
>>> X\_train.shape  
(97,)  
>>> X\_train\_bias.shape  
(97, 2)  
>>> X\_train\_bias  
array([[ 1. , 6.1101],  
 [ 1. , 5.5277],  
 [ 1. , 8.5186],So, what’s happening here? You can import your source code in Python REPL and then interact with the code. Here I import the exercise module and instantiate the Linear regression class I created in the module. I use the readCsvFile method of that class just to get the data read into a Python tuple. Then I import NumPy library and experiment with it using the data.

Python [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) (read/eval/print/loop) is pretty good. It is much faster to use it to create production software in small blocks than e.g. traditional write/compile/deploy/test cycle used in Java. BUT… If you have never used Clojure REPL you know nothing about how powerfull a REPL can be…

### TensorFlow

The real reason for this blog article was TensorFlow so let’s focus on that. The code snippet below shows a typical TensorFlow usage. In TensorFlow you have tensors (matrices of various order) and operations which let tensors to flow from one step to another, hence name “TensorFlow”. Basically tensors and operations provide a higher level abstraction for using matrices and also auxiliary functions for various Machine learning algorithms.

X = tf.placeholder(tf.float32, [None, n])  
y = tf.placeholder(tf.float32, [None, 1])  
W = tf.Variable(tf.ones([n,1]), name="weights")  
init = tf.global\_variables\_initializer()  
y\_prediction = tf.matmul(X, W) # As matrix multiplication in ex1.  
# Cost  
J = (1 / (2 * m)) * tf.reduce\_sum(tf.pow(y\_prediction - y, 2))  
step = tf.train.GradientDescentOptimizer(alpha).minimize(J)  
sess = tf.Session()  
sess.run(init)# For recording cost history.  
J\_history = np.empty(shape=[1],dtype=float)  
# Train iterations.  
for i in range(iterations):  
 sess.run(step,feed\_dict={X:X\_train\_normalized\_bias, y:y\_train})  
 J\_history = np.append(J\_history,sess.run(J,feed\_dict={X:X\_train\_normalized\_bias,y:y\_train}))In the above code we create a couple of TensorFlow variables, and then define the y\_prediction as matrix operation X * W (X = train set features, W = weights, or X * theta using Machine learning course terminology). Then we define the [cost function](https://en.wikipedia.org/wiki/Loss_function) as squared error (see in [Coursera](https://www.coursera.org/learn/machine-learning/lecture/rkTp3/cost-function) how prof. Ng describes it — the lectures were really good). Using TensorFlow to create a linear regression model like this is pretty straightforward once you have basic understanding of what linear regression is and how to get the weights (theta) using a [gradient descent](https://en.wikipedia.org/wiki/Gradient_descent) algorithm (here we use TensorFlow’s [GradientDescentOptimizer](https://www.tensorflow.org/api_docs/python/tf/train/GradientDescentOptimizer) method to do the heavy lifting; in the Coursera course exercises we did this part using matrix calculation— doing the same thing using basic matrix operations was amazingly simple in Octave).

If you are interested about the source code you can take a look in my Github repo in that exercise [1-ml-course-ex1](https://github.com/karimarttila/ml-exercises/tree/master/exercises/2-linear-regression/1-ml-course-ex1) directory. The readme.md in that directory gives a more detailed explanation regarding the exercise. I plan to create more ML exercises using TensorFlow and I add the exercises to that directory when I have done them.

### Conclusions

I’m pretty sure this combination is going to be a killer knowledge for a future developer:

* Fluent in several programming languages.
* Experience to implement transaction intensive and parallel backend applications.
* Understanding and experience to create Big data systems.
* Knowledge to create Cloud infrastructure for your systems.
* Analytics and data science.
* Machine learning and Artificial intelligence.
If you got interested I strongly recommend to start building your developer competence in that direction. You can start e.g. your Machine learning studies today — just sign in Coursera and start watching the first [Machine Learning](https://www.coursera.org/learn/machine-learning) video lecture.

  