---
title: On Descriptive Conditionals
date: 2011-06-24
layout: post
categories:
- Coding
tags:
- code
- codestyle
- conditionals
---

```
if ( ((x > y) || (y == z)) && isSomethingTrue() )  {
    //Do Something
}
```

OK, what just happened there? This type of conditional can be painful to read and debug. Is there a better way?

```
boolean isXBigEnough = x > y;
boolean isYValid     = y == z;
boolean isInCorrectState = isXBigEnough || isYValid;

if ( isInCorrectState && isSomethingTrue() )  {
    //Do Something
}
```

Pulling anonymous conditionals out into named variables can make reading and debugging code much easier. You can come back to this code a year later and know exactly what it was attempting to do based on the names of the variables. Walking through with a debugger is a piece of cake; we can see exactly which conditional was true or false at runtime without having to perform the checks manually. The second format also allows us to easily modify the conditional values via the debugger to change the execution path at run-time - something difficult to accomplish otherwise. But as with anything in coding, there are tradeoffs. In this particular case, we have a performance hit, and a pretty large one at that.

I've taken the following Java code and examined the bytecode produced to get a better idea of what is happening. First up is the straight conditional.

```
private boolean methodOne(int a, int b, int c, int d)  {
    if ( a < b && b < c && c < d )  {
        return true;
    } else {
        return false;
    }
}
```

On compilation, this will produce the following bytecode
```
private boolean methodOne(int, int, int, int);
  Code:
   0:	iload_1
   1:	iload_2
   2:	if_icmpge	18
   5:	iload_2
   6:	iload_3
   7:	if_icmpge	18
   10:	iload_3
   11:	iload	4
   13:	if_icmpge	18
   16:	iconst_1
   17:	ireturn
   18:	iconst_0
   19:	ireturn
```

Reading this, you can see that the compiler takes advantage of <a href="http://en.wikipedia.org/wiki/Short-circuit_evaluation" title="short-circuit evaluation" target="_blank">short-circuit evaluation</a>. At first, only the first two arguments are loaded and compared. If the comparison is true, it goes to the return statement. If it's false, the next two arguments are loaded and evaluated, and so on.

Now let's compare to the documented version:

```
private boolean methodTwo(int a, int b, int c, int d)  {
    boolean first  = a < b;
    boolean second = b < c;
    boolean third  = c < d;

    if ( first && second && third )  {
        return true;
    } else {
        return false;
    }
}
```

Here, we have separated the conditionals out and described what they actually mean. This is a very simple example, and arguably useless in the real world, but the idea is the same. Below is the bytecode produced:

```
private boolean methodTwo(int, int, int, int);
  Code:
   0:	iload_1
   1:	iload_2
   2:	if_icmpge	9
   5:	iconst_1
   6:	goto	10
   9:	iconst_0
   10:	istore	5
   12:	iload_2
   13:	iload_3
   14:	if_icmpge	21
   17:	iconst_1
   18:	goto	22
   21:	iconst_0
   22:	istore	6
   24:	iload_3
   25:	iload	4
   27:	if_icmpge	34
   30:	iconst_1
   31:	goto	35
   34:	iconst_0
   35:	istore	7
   37:	iload	5
   39:	ifeq	54
   42:	iload	6
   44:	ifeq	54
   47:	iload	7
   49:	ifeq	54
   52:	iconst_1
   53:	ireturn
   54:	iconst_0
   55:	ireturn
```

The code is now noticeably longer. Breaking it down, you can see that 0-10 stores the <code>first</code> boolean's conditional result. 12-22 store's <code>second</code> and 24-35 stores <code>third</code>. Now to the important conditional, 37-55. This looks almost exactly like <code>methodOne</code>'s bytecode but it only loads a single value versus two values in <code>methodOne</code>. It still provides for short-circuit evaluation of the final conditional.

The issues should be obvious at this point. <code>methodTwo</code> must perform all of the conditional checks, whereas <code>methodOne</code> takes advantage of short-circuit evaluation to only perform work as necessary. We must also store an additional value during runtime for each conditional, and with that comes the cost of cleaning it up when it goes out of scope.

To compare the run-time performance of the two approaches, I benchmarked the simplest cases that do not take advantage of short-circuit evaluation since the short-circuit may or may not run.

```
    private boolean methodOne(int a, int b)  {
        if ( a < b )  {
            return true;
        } else {
            return false;
        }
    }
    
    private boolean methodTwo(int a, int b)  {
        boolean first  = a < b;
    
        if ( first )  {
            return true;
        } else {
            return false;
        }
    }
```

Benchmarking this simple code shows that the mere act of storing a new boolean slows the method down by <b>18%</b>. With more conditionals and use of short-circuit evaluation, the performance hit will only go up from there.

With that said, this is still extremely fast code; on my laptop running each of these methods 10 million times each, <code>methodOne</code> was only faster than <code>methodTwo</code> by 14 ms. That's only a 1.4 nanosecond difference between each method.

So the question remains, is descriptively naming your conditionals a worthy practice? The answer will depend on your exact situation. For me, I prefer the more descriptive format that naming conditionals provides, and the debugging aspect really puts it over the top. There have been several times where I was able to effectively identify and resolve buggy code by breaking down large and obscure conditionals into their component parts and labeling them with descriptive variables. 

My general approach to writing complex conditionals is to use descriptive naming first, then optimizing later <a href="http://c2.com/cgi/wiki?PrematureOptimization" target="_blank">if it is needed</a>. The majority of the time, it's not, and I'd rather be able to read and debug my code than worry about a few nanoseconds performance hit. Refactoring into the optimized form is a trivial exercise. And with the knowledge gained by actually testing both approaches and looking at the bytecode that is produced, you should be able to make a more informed decision as to which is better for any given situation.