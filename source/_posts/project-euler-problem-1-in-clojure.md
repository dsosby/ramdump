---
title: Project Euler Problem 1 in Clojure
date: 2011-07-15
layout: post
categories:
- Clojure
tags: []
---

I've been trying out <a href="http://clojure.org" target="_blank">Clojure</a> lately. Reasons, non-exclusive, for choosing Clojure?

<ul>
<li>It's a <strong>functional</strong> programming language. I'm unfamiliar with this paradigm and want to expand my horizons.</li>
<li>It provides for <strong>easy concurrency</strong>. With multi-core processors the norm now, I want to make sure I get the most out of them.</li>
<li>It <strong>runs on the JVM</strong>, which <i>should</i> mean that I can tie it in to my everyday work, and if I get really stuck provide a mechanism for me to code in something more familiar.</li>
<li><strong>It's new</strong>, which satisfies my desire for shiny things.</li>
</ul>

So with that, the best way to learn a language is to try it out, and <a href="http://projecteuler.net/" target="_blank">Project Euler</a> provides many fun challenges to get you going. <a href="http://projecteuler.net/index.php?section=problems&id=1" target="_blank">Problem 1</a> is summarized as such: "Find the sum of all the multiples of 3 or 5 below 1000." Remember, I'm new to Clojure and <a href="http://en.wikipedia.org/wiki/Functional_programming" target="_blank">functional programming</a> in general, so if you see issues, feel free to comment.

The first thing that crosses my mind is how I would do this in procedural languages. First, the naive approach in Java

```
public int sumOfMultiples(int[] multiples, int max)  {
    int sum = 0;
    for ( int i=1 ; i```1000 ; i++ )  {
        for ( int j=0 ; j```multiples.length ; j++ )  {
            if ( i % multiples[j] == 0 )  {
                sum++;
                break;   //don't sum the same multiple twice
            }
        }
    }

    return sum;
}

public void printTest()  {
	System.out.println("Result = " + sumOfMultiples([3,5], 1000));
}
```

Like I said, this is a naive approach and has several issues that I will cover later in the post. But as a first shot, I try porting the idea to Clojure.

```
;Project Euler Problem 1
;Return the sum of all unique multiples of 3 and 5 up to 1000

(defn is-multiple [x y]
  (zero? (mod x y)))

(defn multiple3? [x] (is-multiple x 3))
(defn multiple5? [x] (is-multiple x 5))

(defn sum-it-up [sum adder max]
  (if (``` adder max)
	(let [sumadder (if (multiple3? adder) adder (if (multiple5? adder) adder 0))]
	  (sum-it-up (+ sum sumadder) (inc adder) max))
	sum))

(defn do-summation [max]
  (sum-it-up 0 1 max))

(println "Result=" (do-summation 1000))
```

The idea here is the same as the procedural style: loop through numbers 1-1000, and if the number is a multiple of 3 or 5, add it up. Since I don't want an external state to manipulate throughout the loop, I use recursion and keep track of the state within the function itself. This is the code the got me the correct answer first, so I kept refactoring it as I read more about Clojure.

```
(defn is-multiple? [x y]
  (zero? (mod x y)))

(defn sum-it-up
  [max]
  (loop [sum 0 adder 1]
	(if (< adder max)
	  (let [mult3 (is-multiple? adder 3)
			mult5 (is-multiple? adder 5)
			sumadder (if (or mult3 mult5) adder 0)]
		(recur (+ sum sumadder) (inc adder)))
	  sum)))

(time (println "Result=" (sum-it-up 1000)))
```

After a few iterations, this was the result. Things I added include:

 * Use <a href="http://clojure.github.com/clojure/clojure.core-api.html#clojure.core/loop" target="_blank">loop</a>/<a href="http://clojure.org/special_forms#recur" target="_blank">recur</a> - this allows me to define a single function versus two (one to recurse and one to setup) and also allows for <a href="http://en.wikipedia.org/wiki/Tail_call" target="_blank">tail call optimization</a>. Without the use of recur, the JVM would quickly run out of stack space.
 * Removed the specialized is-multiple functions - they were more for experimentation anyways.
 * Added a time function to keep track of optimizations. Something like this is a pain in Java, full of boilerplate.

However, this solution has a few things that I don't like: it still requires iterating over all numbers 1 to max, the conditional for multiples looks "wonky" (that's an official term by the way) and out of place, and it's still <i>long</i>. There has to be a cleaner way.

Clojure has the ability to generate <a href="http://clojure.org/sequences" target="_blank">lazy sequences</a> - you define the sequences via functions and the values are generated only when retrieved. If I can create a sequence consisting of all multiples of 3 and 5 from 1 to max, I could do a simple call to sum them all. After playing around with the different sequence functions available, I came up with the following:

```
(defn is-multiple? [x y] (zero? (mod x y)))
(defn sum-it-up [max]
  (reduce + (filter #(or (is-multiple? % 3) (is-multiple? % 5)) (range (inc max)))))

(time (println "Result=" (sum-it-up 1000)))
```

So what does it mean? It's easiest to read the code from the innermost forms to the outermost. So I'll start:

```
(range (inc max))
```

generates a lazy-sequence from [0-max] (note inclusive). By lazy, it means it's not actually in memory and will only be created as called. Note that I could have also done the following:

```
(map #(inc %) (range max))
```

I actually think this makes a little more sense -- it generates 1-max inclusive since we really don't want 0. However, since 0 doesn't matter in a summation I went ahead and left it in. Timing showed a difference of 3 ms favoring my chosen implementation.

Next up we filter that range to only include our multiples using the following:

```
(filter #(or (is-multiple? % 3) (is-multiple? % 5)) RANGE)
```

Note that this also returns a lazy sequence as well. Filter takes two arguments, a predicate function that should return true or false and a collection. In this case, the predicate function is an anonymous function that returns true if the value is a multiple of 3 or 5.

Now that we have a sequence of all numbers [1-max] that are multiples of 3 or 5, it's time to sum them up. The code
```(reduce + SEQUENCE-OF-MULTIPLES)``` is fairly straightforward. It acts like a looping accumulator taking each item off of the sequence and sending it, along with the accumulated value, to the predicate function (plus in this case) and then storing the result into the accumulator. Once all items have been processed, the accumulated result is returned.