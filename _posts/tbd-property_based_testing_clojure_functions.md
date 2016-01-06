---
layout: post
title: Property-Based Testing in Clojure
tagline: Testing implementations of core clojure functions   
include_social: true
---
{% include JB/setup %}

A few months ago I wrote a blog post about <a href="http://jonathangraham.github.io/2015/09/01/Clojure%20functions/">implementing my own versions of some core Clojure functions</a>. I used a TDD approach to design my functions and ended with a test suite that gave me some confidence that they performed as they should. However, I knew the tests only covered a small extent of the possible scope, and I always had a nagging feeling that there may be bugs in my code that my tests hadn't picked up.

Whilst at <a href="http://melbourne.yowconference.com.au/">Yow! Australia</a> in December, I went to a talk by <a href="http://brisbane.yowconference.com.au/speakers/">Amanda Laucher</a> on Property-Based Testing. Rather than asserting that specific inputs to your code should result in a specific output, as with unit tests, property-based tests make statements about the expected behaviour of the code. These statements are verified for many different possible inputs, with the inputs randomly generated and covering many edge cases. Just what I wanted! Furthermore, <a href="https://twitter.com/reiddraper">Reid Draper</a>, who wrote the property-based testing framework for Clojure, <a href="https://github.com/clojure/test.check">test.check</a> was also at Yow!, so it seemed wrong not to check it out!

To test that I have correctly implemented my own versions of the Clojure functions, I just need to confirm that they behave the same as those in the native languauge. This is testing against the oracle. Therefore, for each function that I am testing, I will assert that the core function and my implementation give the same result for all possible valid inputs. We do not need to worry about what the results are, just that they are both the same.

The source code for my Clojure functions Leiningen project and associated tests are in <a href="https://github.com/jonathangraham/clojure_functions">github</a>. The test framework for the unit tests is speclj, and they can be run using ```lein spec```. Although I wasn't intending on changing the production code, I wanted to keep my suite of unit tests available, at least until the property-based tests were complete.

Test.check integrates with ```clojure.test``` using the ```defspec``` macro, and so the tests can be run using ```lein test```. This meant it was trivial to run the unit and property-based tests independently, which I found useful.

I started with the ```count``` function, as this seemed the most trivial to reason about, and created a ```clojure-functions.count-prop``` namespace.

<pre><code>(ns clojure-functions.count-prop
  (:require [clojure-functions.count :refer :all]
            [clojure.test :refer :all]
            [clojure.test.check.clojure-test :refer (defspec)]
            [clojure.test.check.generators :as gen]
            [clojure.test.check.properties :as prop]))</code></pre>









My implementations were designed to give the same results, but not necessarly be optimised for the same efficiency  

