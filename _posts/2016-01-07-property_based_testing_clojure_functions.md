---
layout: post
title: Property-Based Testing in Clojure
tagline: Testing new implementations of core clojure functions   
include_social: true
---
{% include JB/setup %}

A few months ago I wrote a blog post about <a href="http://jonathangraham.github.io/2015/09/01/Clojure%20functions/">implementing my own versions of some core Clojure functions</a>. I used a TDD approach to design the functions and ended with a test suite that gave me some confidence that they performed as they should. However, I knew the tests only covered a small extent of the possible scope, and I always had a nagging feeling that there may be bugs in my code that my tests hadn't picked up. Of course, I could have just kept writing more and more unit tests, but when do you stop...?

Whilst at <a href="http://melbourne.yowconference.com.au/">Yow! Australia</a> in December, I went to a talk by <a href="http://brisbane.yowconference.com.au/speakers/">Amanda Laucher on Property-Based Testing</a>. Rather than asserting that specific inputs to your code should result in a specific output, as with unit tests, property-based tests make statements about the expected behaviour of the code. These statements are then verified for many different possible inputs, with the inputs randomly generated and covering many edge cases. This was just what I wanted! Furthermore, <a href="https://twitter.com/reiddraper">Reid Draper</a>, who wrote the property-based testing framework for Clojure, <a href="https://github.com/clojure/test.check">test.check</a> was also at Yow!, so it seemed wrong not to check it out! His <a href="https://www.youtube.com/watch?v=JMhNINPo__g">2014 Clojure/West presentation</a> is available to watch, and is well worth checking out as an introduction to test.check.

To test that my versions of the Clojure functions were correctly implemented, we just need to confirm that they behave the same as those in the native languauge. <i>Testing against the Trusted Implementation</i>. So, for each function we can assert that the core function and the new implementation give the same result for all possible valid inputs. We do not need to worry about what the actual results are, just that they are both the same.

The source code for my Leiningen project containing the Clojure functions and associated tests are in <a href="https://github.com/jonathangraham/clojure_functions">github</a>. The test framework for the unit tests is speclj, and they can be run using ```lein spec```. Although I wasn't intending on changing the production code, I wanted to keep my suite of unit tests available, at least until the property-based tests were complete.

Test.check integrates with ```clojure.test``` using the ```defspec``` macro, and so these tests can be run using ```lein test```. Having the different testing frameworks was actually quite useful, as it meant that it was trivial to run the unit and property-based tests independently.

Let's start with the ```count``` function, as this is the most trivial to reason about, and create a ```clojure-functions.count-prop``` namespace. We need to require our ```count``` namespace, where our implementation of count is defined, together with the relevant namespaces of clojure.test and test.check.

<pre><code>(ns clojure-functions.count-prop
  (:require [clojure-functions.count :refer :all]
            [clojure.test :refer :all]
            [clojure.test.check.clojure-test :refer (defspec)]
            [clojure.test.check.generators :as gen]
            [clojure.test.check.properties :as prop]))</code></pre>

In order to write our property-based tests, we need to be able to use and write generators (which we bound above as ```gen```) and properties (bound as ```prop```). You can watch James Trunk give a very clear introduction to both of these <a href="https://www.youtube.com/watch?v=u0TkAw8QqrQ">online</a>, and the <a href="https://github.com/clojure/test.check/blob/master/doc/cheatsheet.md">cheatsheet</a> for generators is very useful.

```prop/all``` is the universal quantification: for all valid inputs to the function this property should be true. It takes two arguments. The first is a binding of a generator to a variable name, in a similar way to a ```let``` clause. For our count property-based test we could start with strings, and use ```gen/string``` as a generator and bind it to ```string```. The second argument is the behaviour statement, and we'll assert that a boolean condition will return true. In our case, we want the same result when we apply ```string``` to both the Clojure ```count``` and our implementation, ```my-count```: ```(is (= (count c) (my-count c)))```. 

Using the ```defspec``` macro we can run our test with any number of generated inputs - let's choose 10000. Putting this all together gives us:

<pre><code>(defspec my-count-property-test 10000
  (prop/for-all [string gen/string]
    (is (= (count string) (my-count string)))))</code></pre>

We run using ```lein test```, and get the result ```{:result true, :num-tests 10000, :seed 1452110327848, :test-var "my-count-property-test"}```.

All 10000 tests passed! The generators are peudorandom, and start with the simplist cases (in this case an empty string) so that failures from common edge-cases can be found quickly. The seed for the generators is returned, so we can always re-run the exact same tests if we want. If not, each time we run lein test we will get a different set of tests run.

Of course, Clojure can apply ```count``` to more than just strings. We can count bytes (```gen/bytes```), and vectors. Vectors of intergers can be generated with ```(gen/vec gen/int)```, but vectors can also contain any clojure value: ```(gen/vector gen/any)```. We can do a similar thing with lists, sets and maps. Maps require both a keyword and value, but both of these can be any clojure value (```(gen/map gen/any gen/any)```). 

We could write a separate property-based test for each of these collection types, but it would be great if we could randomly select one each time. We can do this with the combinator ```gen/one-of```. This generates elements from a vector of given generators, picking generators at random. Let's pull this generator into a separate function in its own namespace so that it will be easily reusable, and call it from our test. The tests will take longer to run now that they can have large collections with any type of colujre values, so for now let's reduce the number of tests that we run. 

<pre><code>(def colls
  (gen/one-of [
    (gen/vector gen/any) 
    (gen/list gen/any) 
    (gen/set gen/any) 
    (gen/map gen/any gen/any) 
    gen/bytes 
    gen/string]))

(defspec my-count-property-test 50
  (prop/for-all [c colls]
    (= (count c) (my-count c))))</code></pre>   

We can run this and it still passes - our implementation of count gives the same result as clojure count across the different collection types. This gives us vastly more test coverage than our unit tests had done.

Let's now move on to ```reduce```, and again we want to test that our implementation behaves the same as the core clojure function. 

```reduce``` takes a function, a collection, and an optional initial value. We can generate collections as before, an the initial value could be any clojure value, ```gen/any```. We now need a function that will work with any two arguments. Just returning one of those arguments should do the trick - if we always return the second argument then reduce should just give us the last element of the collection. This would give us ``` However, as we found out when we were implementing our version of reduce, the function that is passed to ```reduce``` will be evaluated if there is no initial value and the collection is empty. This means we need the function to also take no arguments and return something - we could just make it return ```true```.

 <pre><code>(defn red-fn 
    ([] true)
    ([a b] b))</code></pre>

We can run our tests in an analogous way as for count, and assert that both ```reduce``` and ```my-reduce``` give the same result for the generated collections both with and without an initial value.

<pre><code>(defspec my-reduce-property-test 50
  (prop/for-all [c colls v gen/any]
    (and 
      (= (reduce red-fn c) (my-reduce red-fn c))
      (= (reduce red-fn v c) (my-reduce red-fn v c)))))</code></pre>

These tests also all pass.

We can test our filter implementation in a very similar way. ```filter``` requires a function and a collection. The function takes in each element of the collection in turn and must return a boolean. We can randomly generate booleans using ```gen/boolean```, and pass a generated boolean into the function together with the element from the collection. The function can then simply return the boolean, regardless of what was passed from the collection. We can then test that both implementations of filter return the same filtered collections.

<pre><code>(defn bool-fn [b _] b)

(defspec my-filter-property-test 50
    (prop/for-all [c colls b gen/boolean]
        (= (filter #(bool-fn b %) c) (my-filter #(bool-fn b %) c))))</code></pre>

No we have tests passing for filter as well. 

```map``` takes as arguments a function and any number of collections. Again, the function that we pass can be simple - we just need to test that our implementation can accept it. Let's use ```list``` as this can be applied to any Clojure collections.

This time we need to generate any number of collections to pass as arguments. We saw earlier how to generate vectors, and we can fill the vectors with the collection generators that we have already used. The vector cannot be empty, as ```map``` needs to take at least one collection, so we can use the combinator ```gen/not-empty```. We can then bind this generator of a non-empty vector of collection generators to ```cs```. Now within the assertion we can ```apply map``` to ```list``` and the collections (```(concat cs)```).

<pre><code>(defspec my-map-property-test 50
    (prop/for-all [cs (gen/not-empty (gen/vector colls))]
        (= (apply map list (concat cs)) (apply my-map list (concat cs)))))</code></pre>

We are successfully testing our implemention of ```map``` with random numbers of different types of collections all within a few lines of code!

Lastly is pmap. We can test that our implementation returns the same result as ```pmap``` in the same way that we tested for ```map```:

<pre><code>(defspec my-pmap-property-test 50
    (prop/for-all [c (gen/not-empty (gen/vector colls))]
        (= (apply pmap list (concat c)) (apply my-pmap list (concat c)))))</code></pre>

The key behaviour we have to test for, though, is that for process intensive / long-running functions it processes collections with at least two items quicker than ```map``` does, i.e. it works in parallel.

To ensure that will only generate collections with more than one element, we can use the combinator ```gen/such-that``` with the predicate that the count of the collection is greater than one.

<pre><code>(def colls-more-one-element
    (gen/one-of
    [(gen/such-that #(< 1 (count %))(gen/vector gen/any)) 
    (gen/such-that #(< 1 (count %))(gen/list gen/any)) 
    (gen/such-that #(< 1 (count %))(gen/set gen/any)) 
    (gen/such-that #(< 1 (count %))(gen/map gen/any gen/any)) 
    (gen/such-that #(< 1 (count %))gen/bytes) 
    (gen/such-that #(< 1 (count %))gen/string)]))</code></pre>

We can mimic a process intensive function by adding a thread sleep:

<pre><code>(defn long-running-job 
    ([& args]
        (Thread/sleep 50)
        (apply list args)))</code></pre>

And we can time how long our map functions, which was explained in the previous blog post. 

<pre><code>(defn realize-lazy-seq 
    ([map-type f & args]
        (loop [res (apply map-type f args)]
            (when res
                (recur (next res))))))

(defn test-time 
    ([map-type f & coll]
        (let [st (System/nanoTime)]
            (apply realize-lazy-seq map-type f coll)
            (/ (- (System/nanoTime) st) 1e9))))</code></pre>

We can now assert, for all collections that have more than one item, that our pmap implemention will return quicker than Clojure map. 

<pre><code>(defspec my-pmap-time-property-test 50
    (prop/for-all [c (gen/not-empty (gen/vector colls-more-one-element))]
        (> (apply test-time map long-running-job (concat c)) (apply test-time my-pmap long-running-job (concat c)))))</code></pre>

Our final tests all pass!

<b>Conclusions</b>

We now have property-based tests covering all of our implementations of the Clojure functions, and we can have much more confidence that our functions work as we want. The tests look at the expected behaviour of the code, rather than specific outputs, and there is considerably less test code to maintain compared to the unit tests.

Does this all mean that the unit tests were not required? Not at all. The unit tests helped me to design out my code, and to understand how the functions worked. They just didn't give me the confidence that I needed that I had covered the scope of inputs that are possible, and this is really where the property-based tests shine.

The source code for the implementations of the Clojure functions and associated tests are on <a href="https://github.com/jonathangraham/clojure_functions">github</a>.