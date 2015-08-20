---
layout: post
title: Implementation of Clojure Functions
tagline: Understanding count, filter, map and pmap   
include_social: true
image: Clojure-image
---
{% include JB/setup %}

Every programming language has a bunch of core functions already defined for us. This is great - it means we can quickly get on building what we want, without having to worry about writing everything from scratch.

This power and ease of use does come with a cost, though. Do we really understand how the functions are working, and if not, how do we really know the power of the magic that we are wielding? Are we in danger of breaking our code through the misuse of functionality that we thought, but didn't truly, understand? And are we utilizing the functions to their full effect, or missing out on some of the benefits of the language?

Lets consider, for a moment, <a href="https://en.wikipedia.org/wiki/BASE_jumping">BASE jumping</a>, and lets assume that you are quite the thrill-seaker and an avid skydiver. You have jumped from a plane many times, and feel in full control, from the free-fall, through chute deployment and onto landing. You understand your equipment, and are confident in how to maintain and prepare it, how to operate it, and how to ensure safety as you use it.

But, do you really understand your equipment? Do you know the opportunities that it provides, and the limitations of its use? Are you missing out on the ultimate thrill of BASE jumping because you don't realise that the parachute you already have will do the job? Or are you in severe danger, because you think your equipment will work the same, no matter the altitude of the jump, but you don't fully understand all of the constraints?

Before we take our first BASE jump, lets look at some clojure code. If clojure is new to you, then take a look at <a href="http://jonathangraham.github.io/2015/07/28/Book%20Review%20for%20Living%20Clojure/">Living Clojure</a> and get up to speed. In this post we are going to look at some core Clojure functions, and implement versions of them ourselves. Lets start up a new Leiningen project and get started... 

First up is ```count```. Easy, we just need to find out how many elements are in a list.     