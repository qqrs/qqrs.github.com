---
published: true
layout: post
title: "Which Python operations are atomic?"
date: 2016-05-01 18:03
comments: false
categories: code python
---

A conversation with a coworker turned me on to the fact that a surprising range of operations in Python are atomic, even operations like dictionary and class member assignment.

This wasn't something I would have anticipated, given the number of machine language instructions that must ultimately be performed to complete an operation like hash table insertion.

Why?

The [Python FAQ](https://docs.python.org/2/faq/library.html#what-kinds-of-global-value-mutation-are-thread-safe) provides explanation and a full list of atomic operations, but the short answer is:

1. The Python bytecode interpreter only switches between threads between bytecode instructions
2. The Global Interpreter Lock (GIL) only allows a single thread to execute at a time
3. Many operations translate to a single bytecode instruction

It's easy to check whether an operation compiles to a single bytecode instruction with `dis`.

```
>>> def update_dict():
...     d['a'] = 1
...
>>> import dis
>>> dis.dis(update_dict)
  3           0 LOAD_CONST               1 (1)
              3 LOAD_GLOBAL              0 (d)
              6 LOAD_CONST               2 ('a')
              9 STORE_SUBSCR                        # single bytecode instruction
             10 LOAD_CONST               0 (None)
             13 RETURN_VALUE
```


So what are the caveats? Is it safe to rely on atomicity instead of using locks?

First, the linked FAQ above doesn't make it clear to what degree this behavior is considered part of the Python spec as opposed to simply a consequence of CPython implementation details.
It depends on the GIL, so it would likely be unsafe on GIL-less Pythons (IronPython, Jython, PyPy-TM).
Would it be safe on non-CPython implementations with a GIL (PyPy)?
I could certainly imagine possible optimizations that would invalidate the atomicity of these operations.

Second, even if not strictly necessary, locks provide clear thread-safety guarantees and also serve as useful documentation that the code is accessing shared memory.
Without a lock, care must be taken since it could be easy to assume operations are atomic when they are not (postmortem example: [Python's swap is not atomic](https://emptysqua.re/blog/pythons-swap-is-not-atomic/)).
A clear comment is probably also necessary to head off the "Wait, this might need a lock!" reaction from collaborators.

Third, because Python allows overriding of so many builtin methods, there are edge cases where these operations are no longer atomic. The [Google Python style guide](https://google.github.io/styleguide/pyguide.html#Threading) advises:

> Do not rely on the atomicity of built-in types.
> 
> While Python's built-in data types such as dictionaries appear to have atomic operations, there are corner cases where they aren't atomic (e.g. if __hash__ or __eq__ are implemented as Python methods) and their atomicity should not be relied upon. Neither should you rely on atomic variable assignment (since this in turn depends on dictionaries).

That pretty much settles it for the general case.

There may still be some cases where it would be necessary, such as when implementing new locking functionality or in cases where performance is critical.
Relying on atomicity of operations effectively allows you to piggyback on the GIL for your locking, reducing the cost of additional locks.
But if lock performance is so critical, it seems like it would be better to first profile hotspots and look for other speedups.

So does it make sense to rely on the atomicity of operations when accessing or modifying shared mutable state?

Short answer:  
1. you'd better have a good reason  
2. you'd better do some thorough research  

Otherwise, you're better off just using a lock.
