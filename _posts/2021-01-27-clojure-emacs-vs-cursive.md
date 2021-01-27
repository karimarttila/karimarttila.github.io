---
layout: post
title: Comparing Clojure IDEs - Emacs/Cider vs IDEA/Cursive
category: [clojure]
tags: [clojure, cursive, emacs, programming]
date: 2021-01-27
---

![IntelliJ IDEA and Cursive and Emacs](/img/2021-01-27-clojure-emacs-vs-cursive_img_1.png)

*IntelliJ/Cursive and Emacs/Cider editors side by side in a family portrait.*

### Introduction

Recently I edited a blog post in which I interviewed Metosinians regarding their [favorite Clojure editors](https://www.metosin.fi/blog/metosin-favorite-editors/). It was quite interesting to see that there is a diverse group of editors in use. In that blog post I told myself that I have configured my Emacs/Cider setup to be as close to IDEA/Cursive regarding look-and-feel. Someone in Reddit asked about this and I got an idea that I look at my latest Cursive and Dygma related configurations and check if there is some need for fine-tuning regarding my Emacs setup to make both Cursive and Emacs as similar regarding their look-and-feel as possible - and write a new blog post about this experience. There are a couple of earlier blog posts in which I have touched this topic a bit:

- [Clojure Impressions Round Three]({% post_url 2020-01-06-clojure-impressions-round-three %})
- [Dygma Raise Keyboard Reflections Part 1]({% post_url 2020-09-28-dygma-raise-reflections-part-1 %})


### Versions and Repository

I experimented with Cursive and Emacs editors using my latest [World Statistics exercise](https://github.com/karimarttila/clojure/tree/master/statistics/worldstat) - you can find e.g. the [deps.edn](https://github.com/karimarttila/clojure/blob/master/statistics/worldstat/deps.edn) (e.g. `emacs` profile) and [Justfile](https://github.com/karimarttila/clojure/blob/master/statistics/worldstat/Justfile) (e.g. commands `backend-debug-reveal-kari-port` for starting the repl for Cursive, and `backend-debug-kari-emacs` for starting the repl for Emacs). If you are interested in reading about the World Statistics exercise itself, I have another blog post about that: [World Statistics Exercise]({% post_url 2021-01-19-world-statistics-exercise %}) - I just updated the above mentioned deps.edn and Justfile files for this new blog post.

**Versions** used in this blog post:

- IntelliJ IDEA & Cursive: [IntelliJ IDEA](https://www.jetbrains.com/idea/) 2020.3 Ultimate, [Cursive](https://cursive-ide.com/) 1.10.0-2020.3, and [nrepl](https://github.com/nrepl/nrepl) 0.8.2.
- Emacs & Cider: [Emacs](https://www.gnu.org/software/emacs/) 26.3, [emacs-cider](https://github.com/clojure-emacs/cider) 1.0.0, and [cider-nrepl](https://github.com/clojure-emacs/cider-nrepl) 0.25.5.

My workstation is Ubuntu 20.

### Emacs and Cider

[Emacs](https://www.gnu.org/software/emacs/) is an old editor - the first versions were created in the 1970's, the GNU Emacs development started in the 1980's. I started using Emacs back in my studies at the Helsinki University of Technology in the 1990's - and have been using Emacs ever since. I really like Emacs, though I'm not by any standard an Emacs or eLisp guru. Emacs is implemented using Lisp ([Emacs Lisp](https://www.gnu.org/software/emacs/manual/html_node/elisp/)) which makes it a nice editor for writing Lisp code, e.g. Clojure. Having said that I must emphasize that Emacs is a general-purpose editor, not specifically meant for Lisp programming - you can find a so called [Emacs major mode](https://www.gnu.org/software/emacs/manual/html_node/emacs/Major-Modes.html) basically for any programming language. The best way to describe Emacs is that it is an extensible, customizable editor - you can customize it any way you like using the eLisp language. And since eLisp is a Lisp you can add new editing commands while editing. There are also a lot of [Emacs packages](https://melpa.org/#/) someone has written for you using the eLisp language - the best way to extend Emacs is first search a melpa package, and if you can't find one that suites your needs, only then write one for yourself.

[Cider](https://github.com/clojure-emacs/cider) is an Emacs package that extends Emacs into a full-blown Clojure/script development environment. Using Cider you can do the same REPL wizardry as with any Clojure REPL. The trick actually is that you run a nREPL server and then connect Emacs via Cider to this server - Cider is the nREPL client that communicates with the nREPL server. This way you can send any form inside your Clojure code for evaluation to the nREPL and it sends back the results to Emacs.


### IntelliJ and Cursive

[IntelliJ IDEA](https://www.jetbrains.com/idea/) just turned 20 years old, but I haven't been using IDEA that long. I have been programming with Java and Python for some 20 years, but I started programming Java / Python with Emacs (with Java and Python major modes), and a few years after that started using [Eclipse](https://www.eclipse.org/) for Java (and continued using Emacs for Python). I used Eclipse quite a few years but at some point I started using IntelliJ IDEA for both Java and Python programming - kind of nice to use the same editor with the same look-and-feel for both programming languages, and I have sticked with IntelliJ IDEA ever since, though I still use Emacs for various other programming / editing tasks. (Lately I have also started using [Visual Studio Code](https://code.visualstudio.com/), but that's another story.) 

IntelliJ IDEA is a really good IDE, I really love it. It's not so bloated as Eclipse, and I really like the layout of the IDE tools more in IDEA than in Eclipse (which I always found difficult to use). [Cursive](https://cursive-ide.com/) is an IDEA plugin and provides a really enjoyable Clojure programming environment. So, nowadays I can use IDEA for all of my favorite programming languages.

At this point I must admit that Clojure is my favorite programming language, no question about it. I used to use Python as a quick scripting language, but with [Babashka](https://github.com/babashka/babashka) you can write shell scripts also using Clojure without the long start-up time of the JVM. Java is a bit yesterday - I strongly recommend using Clojure in the JVM, and if Clojure is not an option, then [Kotlin](https://kotlinlang.org/) which is a kind of "Java done right". Btw, you can read more about my experiences regarding different programming languages in my blog post [Five Languages - Five Stories]({% post_url 2018-11-19-five-languages-five-stories %}) and [Kotlin - Much More Than Just a Better Java]({% post_url 2019-12-15-kotlin-much-more-than-just-a-better-java %}) - I really should update the "Five stories" blog post into "Six stories" blog post one day.

### What Is a Clojure REPL?

The REPL is the secret weapon of the Lisp world - a way to interact with the Lisp program under development. Other programmers might say that their favorite language also has a "repl" - it's nothing compared to a real Lisp REPL - you really need a [homoiconic language](https://en.wikipedia.org/wiki/Homoiconicity) to implement a real powerful REPL, and Lisp (and Clojure) is a homoiconic language.

A REPL is an acronym for [Read–eval–print loop](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop). When programming Lisp (and Clojure) a seasoned programmer always keeps a REPL open, and evaluates various forms (the whole namespace, top level form, or some S-expression inside a top level form) in the source code. There are good introductions for the [Clojure REPL](https://clojure.org/guides/repl/introduction) - I strongly recommend reading them and learn how to use the REPL.

All right! I guess that's enough for the introduction. Let's get into the business.

### Starting the REPLs for the Editors

For Clojure newbies I explain that there are a couple of ways to use the REPL - either start the repl as part of your IDE, or start an external REPL and connect to it from your editor - I'm using this second way.

I have in my [deps.edn](https://github.com/karimarttila/clojure/blob/master/statistics/worldstat/deps.edn) the Clojure, nrepl and cider-nrepl versions defined in the dedicated aliases:

```clojure
{:paths ["resources"]
  :deps {org.clojure/clojure {:mvn/version "1.10.1"}}
  :aliases {
...
    :backend {:extra-paths ["src/clj"]
              :extra-deps {metosin/ring-http-response {:mvn/version "0.9.1"}
...
                          nrepl/nrepl {:mvn/version "0.8.2"}
...
    ;; Emacs Cider specific.
    :emacs {:extra-deps {cider/cider-nrepl {:mvn/version "0.25.5"}}}}
```

The `backend` alias was the actual alias for backend development - I created the `emacs` alias for this blog post.

The [Justfile](https://github.com/karimarttila/clojure/blob/master/statistics/worldstat/Justfile) provides the commands for starting the repls with the aliases: `backend-debug-reveal-kari-port` for starting the repl for Cursive, and `backend-debug-kari-emacs` for starting the repl for Emacs:

```bash
# For Cursive
@backend-debug-reveal-kari-port:
    # clj -J-Dvlaaad.reveal.prefs="{:theme :light}" -M:dev:test:common:backend:reveal:kari -m nrepl.cmdline --middleware '[com.gfredericks.debug-repl/wrap-debug-repl vlaaad.reveal.nrepl/middleware]' -p 44444 -i -C
    clj -M:dev:test:common:backend:reveal:kari -m nrepl.cmdline --middleware '[com.gfredericks.debug-repl/wrap-debug-repl vlaaad.reveal.nrepl/middleware]' -p 44444 -i -C

# Start backend repl with my toolbox for Emacs.
@backend-debug-kari-emacs:
    PROFILE=emacs clj -M:dev:test:common:backend:reveal:kari:emacs -m nrepl.cmdline --middleware '[com.gfredericks.debug-repl/wrap-debug-repl vlaaad.reveal.nrepl/middleware cider.nrepl/cider-middleware]' -p 55555 -i -C
```

I created both [Just](https://github.com/casey/just) recipes for this blog post - the idea is that I added an explicit port for both repls so that I can be sure that Cursive and Emacs are connecting to the respective repls (Cursive port 44444, and Emacs port 55555). The first command has another version with a ligth theme that I tried but didn't like it that much. In both REPLs I start the nrepl (`-m nrepl.cmdline`) and then add some middleware (`'[com.gfredericks.debug-repl/wrap-debug-repl vlaaad.reveal.nrepl/middleware]'` , and `cider.nrepl/cider-middleware` for Emacs). The first one is a debug helper and the next is a REPL output tool, more about that later.


### Connecting to the REPLs from the Editors

In **IDEA / Cursive** create a Run/Debug Configuration as in the picture below:

![Cursive Run Configuration](/img/2021-01-27-clojure-emacs-vs-cursive_img_2.png).

In **Emacs / Cider** give command: `cider-connect` and give the localhost and port you used when starting the repl previously (`55555`).

### The REPL Output in the Editors

The picture below shows the Cursive REPL output window. It is important to mention here that Clojurians do not write in the REPL. You write in the editor and send the forms for evaluation to the REPL, typically using some hotkey (more about that later). So, in the picture below I have first reseted my [Integrant]({% post_url 2020-09-07-clojure-integrant-exercise %}) state (using a hotkey, of course). Then I have evaluated the forms (e.g. `(set! *print-length* 100)`) one by one with a dedicated hotkey.

![Cursive REPL output window](/img/2021-01-27-clojure-emacs-vs-cursive_img_3.png)

I actually have three monitors at my table, and I keep my Cursive REPL output window in another monitor than the editor itself. Recently I have been experimenting with the [Reveal](https://vlaaad.github.io/reveal/) REPL tool and I keep both output windows side by side when experimenting stuff in the Reveal:

![Cursive and Reveal REPL output windows](/img/2021-01-27-clojure-emacs-vs-cursive_img_4.png)

Let's next see the same thing using Emacs:

![Emacs REPL output window](/img/2021-01-27-clojure-emacs-vs-cursive_img_5.png)

In Emacs you see 4 buffers. The top buffer shows a Clojure file (one of my scratch files in which I do some experiments). Below that are side by side the cider-repl buffer which is connected to the REPL I described in the previous chapter, and a message buffer next to it (REPL output echoed). Bottom is the command buffer which also echoes the REPL output. Emacs also nicely echoes the REPL output also in the editor buffer:

```clojure
=> {:country_name "Finland", :country_code :FIN, :series-name "Hospital beds (per 1,000 people)", :series-code :SH.MED.BEDS.ZS, :year 2002, :value 7.4, :country-id 246}
```

You can start the same way the Reveal REPL output tool, as previously (it's a REPL middleware and not connected to the editors) - I have the Reveal window next to Emacs.

Now I have explained how to start the REPLs and how to connect to the REPLs using the editors. Let's next dive into the look-and-feel.

### Look-And-Feel - the Look

The look is basically the theme of the editor - how colors are used, font ligature etc.

**IDEA/Cursive**.

When I started Clojure programming with IDEA/Cursive I wanted to find a light theme which would be nice to my eyes. I have never liked the dark themes that much. I actually thought that there was a Leuven theme in IntelliJ but now that I checked it I noticed that I have created a custom "KariLeuven" theme - possibly used some existing Leuven-like theme as a basis, can't remember anymore. Anyway the color theme is rather simple: a light theme with just some coloring (`def`, `defn` and language macros like `comment` using blue, keywords using violet, and strings using green). The picture below shows my theme customizations.

![Cursive Clojure theme](/img/2021-01-27-clojure-emacs-vs-cursive_img_6.png).


**Emacs**.

I took the initial setup for my Clojure Emacs configuration from [flyingmachine's Emacs Cursive setup](https://github.com/flyingmachine/emacs-for-clojure/). Then I fine-tuned it a bit, e.g. using the Leuven theme which I initially got from [fniessen](https://github.com/fniessen/emacs-leuven-theme), and fine-tuned it a bit further.


### Look-And-Feel - the Feel

The Feel part is how one navigates in the editor and manipulates S-expressions. Since Clojure is a Lisp and therefore a homoiconic language all expressions are so called [S-expressions](https://en.wikipedia.org/wiki/S-expression) - S-expressions can be constructed from other S-expressions. That's why Lisps have a lot of parentheses - but this also makes the language really powerful (macros etc) and nice to edit. There are two major schools related how to edit Lisp code: the older paredit style and the newer parinfer style. Using paredit you have certain commands that you use to manipulate the S-expressions, e.g. move this S-expression inside the next S-expression etc. When using parinfer you don't have to remember any special commands but you achieve the same results by indenting Lisp code.

Here are a couple of web pages that illustrate the difference nicely:

- [paredit](http://danmidwood.com/content/2014/11/21/animated-paredit.html)
- [parinfer](https://shaunlebron.github.io/parinfer/)

I use paredit myself. I tried parinfer but it felt a bit odd. I have the same hotkeys for slurping and barfing for both IDEA/Cursive and Emacs and it is therefore pretty effortless for me to edit Lisp code using paredit. When [interviewing my colleagues at Metosin](https://www.metosin.fi/blog/metosin-favorite-editors/) most of the programmers used paredit. But if you are a newcomer to the Lisp / Clojure land I suggest using parinfer - it is easier to start editing the code when you don't have to learn any special commands.

### Hotkeys for Slurping and Barfing

To understand my slurping and barfing hotkeys the reader needs to read my previous blog post [Dygma Raise Keyboard Reflections Part 1]({% post_url 2020-09-28-dygma-raise-reflections-part-1 %}) first. In that blog post I explain how I have configured the CapsLock key to function as AltGr key in order to use it to get the various parentheses without twisting my right thumb. Then the reader needs to understand the two layers I have configured in my Dygma Raise. I really recommend [Dygma Raise](https://dygma.com/) - the best keyboard I have ever used - a perfect keyboard for a programmer.

The following two pictures show my favorite hotkeys in **IDEA/Cursive**:

![Cursive REPL hotkeys](/img/2021-01-27-clojure-emacs-vs-cursive_img_7.png).

The most used REPL hotkeys are the `Integrant reset` = `Alt-J` and `Send Form Before Caret to REPL` = `Alt-L`. 

Then the paredit manipulation hotkeys:

![Cursive Paredit hotkeys](/img/2021-01-27-clojure-emacs-vs-cursive_img_8.png).

Then the same settings in **Emacs**:

```elisp

;; override the default keybindings in paredit
(eval-after-load 'paredit
  '(progn
     (define-key paredit-mode-map (kbd "C-M-j")  nil)
     (define-key paredit-mode-map (kbd "C-M-l")  nil)
     (define-key paredit-mode-map (kbd "C-M-j")  'paredit-backward-slurp-sexp)
     (define-key paredit-mode-map (kbd "M-<right>") 'paredit-forward-slurp-sexp)
     (define-key paredit-mode-map (kbd "C-M-l")  'paredit-backward-barf-sexp)
     (define-key paredit-mode-map (kbd "M-<left>")  'paredit-forward-barf-sexp)
     (define-key paredit-mode-map (kbd "C-<right>") 'right-word)
     (define-key paredit-mode-map (kbd "C-<left>")  'left-word)
     ))
...
(eval-after-load 'cider
  '(progn
...
     (define-key cider-mode-map (kbd "M-l")  'cider-eval-last-sexp)
     (define-key cider-mode-map (kbd "M-ö")  'cider-eval-defun-at-point)
     (define-key cider-mode-map (kbd "M-n")  'cider-repl-set-ns)
     (define-key cider-mode-map (kbd "M-m")  'cider-load-buffer)
     (define-key cider-mode-map (kbd "M-{")  'cider-format-buffer)
     (define-key cider-mode-map (kbd "M-å")  'cider-test-run-ns-tests)
     (define-key cider-mode-map (kbd "M-ä")  'cider-test-run-test)
```

As you can see it is pretty simple to configure the same REPL hotkeys and the same paredit hotkeys for both IDEA/Cursive and Emacs/Cider. When reading this you have to remember that I'm not using arrow keys but my arrow keys are defined as CapsLock + i/j/k/l, so when I'm forward barfing I actually have my left little finger in CapsLock and my left thumb in one of the Dygma thumb keys (which is Alt) and hit `J` with my right index finger. This may sound very complicated, but actually, it isn't - I have considered various layouts and the current layout is a kind of evolutionary result of my keyboard layout experiments. I really like the system since it resembles a bit playing the classical guitar: I hit the "control" (ctrl, shift, alt, CapsLock) key combinations using my left hand, and the navigation (arrow), manipulation (barfing/slurping), and evaluation (REPL) keys with my right hand. All these key combinations are in my muscle memory and I don't consciously think about them - I just edit code. Even though this system suits me nicely I wouldn't recommend it to someone else - you have to experiment and find your own keyboard, keyboard layout, and finally the hotkeys for your favorite programming language in that layout.


### Conclusions

Both IDEA/Cursive and Emacs/Cider are excellent editors and Clojure integrated development environments. If you want to switch from one to another you can quite easily configure both editors to have pretty much same look-and-feel.


*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
