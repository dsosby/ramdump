---
title: ClojureScript Gotchas & Tips
date: 2012-09-18
layout: post
categories:
- Clojure
- ClojureScript
tags: []
---

I've been playing a lot lately with ClojureScript, especially with regards to mobile development (more on that to come later). Clojure has never been known for great error messages, and ClojureScript is probably even worse in that regard.

Included below are a list of things that gave me a rough time. Some are pretty simple, but needless to say they got me. Hopefully they won't get you.
<h1>Gotchas</h1>
<h3>namespaces etc</h3>
The ns function is very special in ClojureScript when compared to Clojure. I won't go over all the differences, those have been covered, but I'll highlight a few things quickly: only one namespace per file; use requires only and require requires as; REPL requires load-namespace instead of ns call; invalid require statements don't give compile errors.

A big one that got me is requiring/using the Google Closure libs and classes. When you require them in your namespace, the ClojureScript compiler just outputs goog.require("goog.foobar"). The :as makes no difference here. When you refer to the functions/objects, you have to use the fully qualified name, e.g. (goog.foobar/baz 42).

To sum that annoying issue up, you have to both require and use the fully-qualified name.
<h3>Interop With JS Objects</h3>
Sometimes you just have to fall back. Use the js* function to create some straight-up JavaScript:
<pre>((js* "alert") "Hello")</pre>
There will also come a time when you have to create an object to interop with a JavaScript library. It would be nice if ClojureScript maps/lists could be output as an object, but that may just be wishful thinking. In other words, you'll have to copy and paste one of the dozen or so implementations into your code.

You can also use the goog.object package for help which is wrapped by several CLJS functions, the primary one being js-obj which takes arguments and simply applies them to goog.object/create.

Example:
<pre>(js-keys (js-obj "name" "Ben Franklin" "age" 88 "height" 60))</pre>
<h3>Arity With Functions</h3>
When you define a function using defn, it creates a JavaScript function in the background. JavaScript functions do not enforce arity. This means you can do something like this without any failure, only unexpected results.
<pre>(defn tip-cows [])
(tip-cows 1 "zebra")</pre>
<h3>The Compiler Doesn't Care</h3>
Don't expect the compiler to help you out chasing and picking out errors. The compiler only ensures the code can be read and checks for a few minor things (like :require has :as, etc). The vast majority of errors will be found at runtime. You'll have to change your mindset from compiled language to interpreted language.
<h1>Tips</h1>
<a href="http://himera.herokuapp.com" target="_blank">Himera ClojureScript REPL</a> - An online REPL. Very useful! Don't miss the Summary page.

Don't expect (= :ClojureScript :Clojure). They are different.

Surf the <a href="https://github.com/clojure/clojurescript/blob/master/src/cljs/cljs/core.cljs" target="_blank">source</a>!

When you get a "call undefined" error, you most likely forgot your require, misspelled it, or similar.

When you get an error, or something goes bad, load up the JavaScript console and play around. It obviously helps to know JavaScript to some extent here. For example, if you get an undefined error in something like:
<pre>goog.foobar.baz.call(null, helloworld.core.mydef);</pre>
You could use the console to probe and find the culprit, e.g. type
<pre>goog;
goog.foobar;
goog.foobar.baz;</pre>
Keep probing the environment to see what is not as is expected.