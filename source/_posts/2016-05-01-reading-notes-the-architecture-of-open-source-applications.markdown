---
published: false
layout: post
title: "Reading Notes: The Architecture of Open Source Applications II"
date: 2016-05-01 16:37
comments: false
categories: 
---

Earlier this year, I read [*The Architecture of Open Source Applications, Volume II: Structure, Scale, and a Few More Fearless Hacks*](http://aosabook.org/en/buy.html#vol2).

I found it to be an excellent read, filling a gap in the market for software engineering books. The synopsis makes it clear this is was intentional:

> Architects look at thousands of buildings during their training, and study critiques of those buildings written by masters. In contrast, most software developers only ever get to know a handful of large programs well - usually programs they wrote themselves - and never study the great programs of history. As a result, they repeat one another's mistakes rather than building on one another's successes. This second volume of The Architecture of Open Source Applications aims to change that. In it, the authors of twenty-four open source applications explain how their software is structured, and why. What are each program's major components? How do they interact? And what did their builders learn during their development? In answering these questions, the contributors to this book provide unique insights into how they think.

There simply isn't enough time in the day to do a thorough reading of all of the worthy open source projects in existence.
And while code reading can be a very worthwhile exercise, you first have to mentally piece together the higher level architecture from the lower level implementation details,
and then have to guess for yourself why the designers made certain choices over the possible alternatives.

In AOS2 every chapter is a guided tour of interesting codebases, with all of the hard-earned design wisdom distilled down to specific advice.

While the chapters are mostly free-form, I noticed that a few repeating themes crop up in many of the chapters.
None are surprises, but I found it very enlightening to see specific examples from so many projects:  
1. Modularity  
2. Testing  
3. The danger of expecting to be able to predict the future  

While the entire book is a great read, a few of the chapters stood out as being espcially thought-provoking, if you're looking for a place to start:  
1. Scalable Web Architecture and Distributed Systems  
2. FreeRTOS  
3. The Glasgow Haskell Compiler  
4. GPSD  
5. nginx  
6. Twisted  
7. Yesod  
8. ZeroMQ  

The full content is available to [read for free online](http://aosabook.org/en/index.html), and all profits from [hard copy sales](http://aosabook.org/en/buy.html#vol2) go to charity.
