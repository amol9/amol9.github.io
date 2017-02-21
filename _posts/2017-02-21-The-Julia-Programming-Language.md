---
layout: post
title: The Julia Programming Language
tags: julia, programming language
---

It's the first time I have programmed in a language having a girl's name. I have programmed in a language with a guy's name, rust. But, rust also has a general meaning. Most other languages take names from an animal, a beverage, a verb or a letter. It's okay, but, I think they should have stayed away from "j" as the first letter. The programming world is over populated with things starting with j.

I think I first noted the language in passing when scrolling through a drop down for selecting a programming language. After that, references to julia kept coming up here and there.

One day, I googled it. Then, I decided to try it. I read through the docs and installed the language.

Some of the features of julia are very impressive. It has a natural syntax for typing polynomial equations. E.g. X^2 + 2x + 2, etc. You can totally skip the "." Or "x" between the co-efficient and the variable name.

It has an interactive with a feature which would have prevented many headaches for me in the past when dealing with interactive shells of other languages. When you use the "up" arrow key to access previous code, it brings up an entire multi line code block if you typed one. And you can then scroll up and down in it to edit it.

It has very handy macros and functions to do common tasks very fast. Like, zeros() to fill up an array with zeroes. A dot , ".", Between a function name and an array does a map() operation on the array.

But, the real power is in the built in support for asynchronous and parallel programming.

* @parallel for can do a for loop in a parallel fashion
* @spawn / @spawnat can perform a task in a separate process and fetch() can bring you back the results
* pmap() lets you do a map() operation in parallel
* @async lets you define asynchronous tasks
* @sync lets you wait for all the enclosing @async tasks to complete

This feels like power. I wanted to try it out.

So, I wrote up a small program to asynchronously do multiple http requests to get me some data. It took some time to wrap my logic into the constructs provided by julia, but, it did not take many lines of code. I'll write about it in my next post.
