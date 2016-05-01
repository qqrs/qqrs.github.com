---
published: true
layout: post
title: "Git Hash Collisions"
date: 2016-05-01 14:40
comments: false
categories: code books
---

While re-reading *The Architecture of Open Source Applications, Volume II*, a line in the [chapter on git](http://aosabook.org/en/git.html) caught my eye:

> if two objects are different they will have different SHAs.

This was surprising, as there should always be be some very very small but nonzero chance of a hash collision. Either the book is simplifying for ease of explanation, or git is doing something behind the scenes that is more complicated than simply hashing the contents of the object.

It turns out that hash collisions are possible.

As expected, an accidental collision is very unlikely: on even a large and active repo, a collision is unlikely to occur in the timespan of the [age of the universe](http://diego.assencio.com/?index=eacd6eedf46c9dd596a5f12221ad15b8).

Even if an attacker could break SHA-1 or brute-force a hash, [git always prefers the older version of an object](http://stackoverflow.com/a/9392525), so the attacker would be unable to replace an existing file with a tainted version.
