---
published: false
layout: post
title: "Git Hash Collisions"
date: 2016-04-29 01:13
comments: false
categories: 
---

While re-reading *The Architecture of Open Source Applications, Volume II*, a line in the [chapter on git](http://aosabook.org/en/git.html) caught my eye:

> if two objects are different they will have different SHAs.

This didn't seem to be strictly correct to me. I expected there to be some very very small but nonzero chance of a hash collision. If different objects always had different SHAs, git would need to be doing something behind the scenes to handle collisions.

It turns out that hash collisions are possible.

As expected, an accidental collision is very unlikely: on even a large and active repo, a collision is unlikely to occur before [the end of the universe](http://diego.assencio.com/?index=eacd6eedf46c9dd596a5f12221ad15b8).

Even if an attacker could break SHA-1 or brute-force a hash, [git always prefers the older version of an object](http://stackoverflow.com/a/9392525), so the attacker would be unable to replace an existing file with a tainted version.
