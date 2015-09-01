---
layout: post
title: Understanding core Clojure Functions
tagline: Implementing versions of <a href="#reduce">reduce</a>, <a href="#count">count</a>, <a href="#filter">filter</a>, <a href="#map">map</a> and <a href="#pmap">pmap</a>   
include_social: true
image: Clojure-image
---
{% include JB/setup %}

Every programming language has a bunch of core functions already defined for us. This is great - it means we can quickly get on building what we want, without having to worry about writing everything from scratch.

This power and ease of use does come with a cost, though. Do we really understand how the functions are working, and if not, how do we really know the power of the magic that we are wielding? Are we in danger of breaking our code through the misuse of functionality that we thought, but didn't truly, understand? And are we utilizing the functions to their full effect, or missing out on some of the benefits of the language?

Lets consider, for a moment, <a href="https://en.wikipedia.org/wiki/BASE_jumping">BASE jumping</a>, and lets assume that you are quite the thrill-seeker and an avid skydiver. You have jumped from a plane many times. You feel in full control from the initial leap, through free-fall and chute deployment, and onto landing. You understand your equipment; you are confident in how to maintain and prepare it, how to operate it, and how to ensure safety as you use it.

But, do you really understand your equipment? Do you know the opportunities that it provides, and the limitations of its use? Are you missing out on the ultimate thrill of BASE jumping because you don't realise that the parachute you already have will do the job? Or are you in severe danger because you think your equipment will work the same no matter the altitude of the jump, and you don't fully understand all of the constraints?

Before we take our first BASE jump, let's look at some clojure code. If clojure is new to you, then take a look at <a href="http://jonathangraham.github.io/2015/07/28/Book%20Review%20for%20Living%20Clojure/">my book review of Living Clojure</a> and get up to speed. In this post we are going to look at some core clojure functions, and implement versions of them ourselves. The aim is not to exactly replicate how the functions are already written, but to get an understanding of what they can and cannot do. We will build things up slowly, starting with <a href="#reduce">reduce</a>, before moving to <a href="#count">count</a>, <a href="#filter">filter</a>, <a href="#map">map</a> and <a href="#pmap">pmap</a>. 

Let's start up a new Leiningen project. I like the testing framework of <a href="http://speclj.com">speclj</a>, so <code>lein new specjls clojure_functions</code> got me started...

<a name="reduce"></a>
<b>REDUCE</b> 

First up we will implement our own version of ```reduce```, because it is the backbone of many clojure seq functions. Looking at the requirements for <a href="https://clojuredocs.org/clojure.core/reduce">reduce</a> there is quite a lot that we need it to do. It needs a function, which we'll call ```f```, which takes 2 arguments, but it can also just accept one. 

Let's start by taking the case with two arguments. The first argument is the starting value, ```val``` and the second is a collection, ```coll```. ```reduce``` will apply ```f``` to the ```val``` and the first element of ```coll```, and then will apply ```f``` to that result and the second item of ```coll```, etc. This sounds easily solved with a recursive function, but we'll build it up slowly. Firstly, the documentation tells us that if ```coll``` contains no items it should just return ```val```, and not apply ```f```. Let's start by writing a test - I'm using <a href="http://speclj.com">speclj</a>, so if you are using lein test or a different testing framework then the syntax will be different. We're going to write a test with the function ```+```, an initial ```val``` of 1, and an empty list. We expect this to just return ```val```, so should output 1.

<pre><code>(describe "test my-reduce function"

	(it "result 1 for addition function, with val of 1 and empty collection"
		(should= 1 (my-reduce + 1 '()))))</code></pre>

We can run our test, and get it to rerun automatically when we make changes to our program, by usings ```lein spec -a```. Our test fails, so let's write a function that will make this test pass. We can call our function ```my-reduce``` and it will need parameters ```f```, ```val``` and ```coll```. To make this first test pass we just need to return ```val```.
<pre><code>(defn my-reduce [f val coll]
	val)</code></pre>

Not very exciting, but it works. So let's now add a test where we have a single element, 1, in the collection. We need to apply the function ```+``` to ```val``` and the first element of ```coll```, so this should give the result 2.

<pre><code>(describe "test my-reduce function"

	(it "result 1 for addition function, with val of 1 and empty collection"
		(should= 1 (my-reduce + 1 '())))

	(it "result 2 for addition function, with val of 1 and collection containing element 1"
		(should= 2 (my-reduce + 1 '(1)))))</code></pre>

Our new test fails, so let's modify our function. It now just needs to return ```val``` if ```coll``` is empty, and otherwise needs to apply ```f``` to ```val``` and the first element of ```coll```.

<pre><code>(defn my-reduce [f val coll]
	(if (empty? coll)
		val
		(f val (first coll))))</code></pre>

It passes! We can easily extend our tests to other clojure collections, such as vectors and sets and check that our ```my-reduce``` function also works with these.

What if we have more than one element in the collection? We need to apply ```f``` to the result of applying ```f``` to val and ```(first coll)``` to the second element. Let's add another test:

<pre><code>(it "result 6 for addition function, with val of 1 and collection containing elements 2 and 3"
		(should= 6 (my-reduce + 1 [2 3])))
</code></pre> 
This now fails, so to get it to pass we need to recursively call ```my-reduce```, with the previous result being passed to ```val``` and ```(rest coll)``` being passed to ```coll```, and this will continue until ```coll``` is empty, at which point ```val``` is returned.

<pre><code>(defn my-reduce [f val coll]
	(if (empty? coll)
		val
		(my-reduce f (f val (first coll)) (rest coll))))
</code></pre>
The tests are all back working, so let's add a few more examples of reduce and check that our implementation handles them:

<pre><code>	(it "convert a vector to a set"
		(should= #{:a :b :c} (my-reduce conj #{} [:a :b :c])))

	(it "convert tuples to a map"
		(should= {:b 2, :a 1} (my-reduce conj {} [[:a 1] [:b 2]])))

	(it "add one collection to another"
		(should= [1 2 3 4 5 6] (my-reduce conj [1 2 3][4 5 6])))

	(it "combine sequences with cons"
		(should= '(6 5 4 1 2 3) (my-reduce #(cons %2 %1) [1 2 3] [4 5 6])))</code></pre>

All good, so what else does ```reduce``` need to do? Looking at clojure docs we need our implementation of ```reduce``` to also work if ```val``` is not supplied. In this case it should return the result of applying ```f``` to the first 2 items in ```coll```, then apply ```f``` to that result and the 3rd item, etc.

Let's write a new test:
<pre><code>	(it "result 1 for addition function with no val and list containing element 1"
		(should= 1 (my-reduce + [1])))</code></pre>

This fails becuase the wrong number of arguments are passed to ```my-reduce```. To fix this, we can add in an arity for ```my-reduce``` to just take in ```f``` and ```coll```, and then for it to call ```my-reduce``` passing ```(first coll)``` as ```val```, and ```(rest coll)``` as ```coll```. We have already written our ```my-reduce``` in a way to cope with an empty ```coll```, so this should work as long as we have at least one item in our collection.

<pre><code>(defn my-reduce
	([f coll]
		 (my-reduce f (first coll) (rest coll)))
	([f val coll]
		(if (empty? coll)
			val
			(my-reduce f (f val (first coll)) (rest coll)))))</code></pre>

This works as we wanted, and the test passes. We can add some additional tests to confirm that ```my-reduce``` handles multiple items in a collection in the way that we expect.

<pre><code>	(it "result 6 for multiplication on collection containing elements 1, 2 and 3"
		(should= 6 (my-reduce * [1 2 3])))

	(it "Combine a vector of collections into a single collection of the type of the first collection in the vector"
		(should= [1 2 3 :a :b :c [4 5] 6] (my-reduce into [[1 2 3] [:a :b :c] '([4 5] 6)])))</code></pre>

Finally, what happens if we pass in no ```val``` and an empty ```coll```? We need to return the result of calling ```f``` with no arguments. Evaluating ```+``` with no arguments returns 0, and evaluating ```*``` with no arguments returns 1. Let's add these tests.

<pre><code>	(it "result 0 for addition on empty list"
		(should= 0 (my-reduce + [])))

	(it "result 1 for multiplication on empty list"
		(should= 1 (my-reduce * [])))</code></pre>

To make these tests pass we simply need to make our version of ```my-reduce``` that takes ```f``` and ```coll``` to evaluate ```f``` if the ```coll``` is empty.

<pre><code>(defn my-reduce
	([f coll]
		(if (empty? coll)
			(f)
		 	(my-reduce f (first coll) (rest coll))))
	([f val coll]
		(if (empty? coll)
			val
			(my-reduce f (f val (first coll)) (rest coll)))))</code></pre>

So, now we've written our own implementation of ```reduce```, do we still want to take that BASE jump?

If we previously thought that we needed to pass an initial value as well as a collection to ```reduce```, we now have extended the power and utility we wield over this fundamental function. Also, even if we have a function that requires two arguments, we now know that we can successfully pass this to ```reduce```. Let's say we define a simple function ```f``` as ```(defn f [x y] (+ x y))```. If we call ```f``` with just one argument, say ```(f [1])``` we will get an error. However, we now know that calling ```(reduce f [1])``` will return ```1```, because the first item of the collection will be defined as ```val```, and given the rest of the collection is empty, ```reduce``` will just return ```val```.  

But we also now know some of the limitations of ```reduce```. Consider the situation where we define our ```coll``` as being a vector that contains a random number of elements, between 0 and 2, with the elements being consecutive integer values starting at 0: ```(def coll (into [] (range (rand-int 3))))```. We will not pass a ```val``` to ```reduce```, and we're going to pass in ```+``` as ```f```. So, what happens when we call ```reduce```? If we pass ```[0 1]``` as ```coll```, we will return the value 1, having applied ```+``` to the elements. If we pass ```[0]``` we will return 0, given there is only one element and so ```f``` is not called. What if we pass in ```[]```? We are passing an empty collection, so we just evaluate ```f```, which returns 0 in this case. All good. But what if we set ```-``` as our function? This will return -1 if we pass ```[0 1]```, and 0 if we pass ```[0]```, but will cause an error if we try to pass ```[]```, given it is not possible to evaluate ```-``` with no arguments. So, if we want to use ```reduce``` where there is a possiblity that we will be passing in an empty collection and no initial value then we need to make sure that we define a function that can be evaluated with no arguments. Alternatively, we now know that we can write our own ```reduce``` function, and we can modify it so that it defines differently what to do with an empty collection.

<a name="count"></a>
<b>COUNT</b>

Now we have written our implementation of ```reduce```, let's move on to ```count```. ```count``` takes a collection and returns the number of items that it contains. It will return 0 for nil. We will build our ```my-count``` function using a TDD approach again, so we start by writing a test.

<pre><code>(it "result 0 for an empty list"
		(should= 0 (my-count '())))</code></pre>

We can make this test pass by just returning 0: ```(defn my-count [coll] 0)```. We can also add a test where we pass ```nil``` in place of a collection, and our code will again suffice. Let's quickly add a third test, where we pass a collection containing a single element.

<pre><code>(describe "test count function"

	(it "result 0 for an empty list"
		(should= 0 (my-count '())))

	(it "result 0 for nil"
		(should= 0 (my-count nil)))

	(it "result 1 for a list of one item"
		(should= 1 (my-count '(1)))))</code></pre> 

To make this test pass, we can return 0 if the collection is empty (the clojure function ```empty?``` will return true for a collection with no items, and will also return true for nil), and otherwise we will return 1.

<pre><code>(defn my-count [coll]
	(if (empty? coll)
		0
		1))</code></pre>

Great, so now let's get this working if we have more than one item in the collection: ```(should= 2 (my-count '(1 2)))```. We can do this recursively by starting ```result``` at 0 and adding 1 every time we move a position along the collection until the collection is empty, at which point we simply return ```result```:

<pre><code>(defn my-count [coll]
	(loop [coll coll result 0]
		(if (empty? coll)
			result
			(recur (rest coll) (inc result)))))</code></pre>

This works, so let's add some more tests to make sure our ```my-count``` works as we expect:

<pre><code>(describe "test count function"

	(it "result 0 for an empty list"
		(should= 0 (my-count '())))

	(it "result 0 for nil"
		(should= 0 (my-count nil)))

	(it "result 1 for a list of one item"
		(should= 1 (my-count '(1))))

	(it "result 2 for a list of two items"
		(should= 2 (my-count '(1 2))))

	(it "result 10 for a list of ten items"
		(should= 10 (my-count '(:a :b :c :d :e :f :g :h :i :j))))

	(it "result 0 for an empty vector"
		(should= 0 (my-count [])))

	(it "result 10 for a vector of ten elements"
		(should= 10 (my-count [1 2 [3 4] 5 6 7 8 9 10 11])))

	(it "result 0 for an empty string"
		(should= 0 (my-count "")))

	(it "result 10 for a ten character string"
		(should= 10 (my-count "helloworld")))

	(it "result 0 for an empty map"
		(should= 0 (my-count {})))

	(it "result 1 for a map with a single key value pair"
		(should= 1 (my-count {:a 1})))

	(it "result 5 for a map with five key value pairs"
		(should= 5 (my-count {:a 1, "b" [1 2], 3 4, :d 5, "e" {:s 10}}))))</code></pre>

All tests pass! Now we have a test suite, let's refactor. We have already written ```my-reduce```, so let's see if we can use this to define ```my-count```. We would want to set ```val``` to 0 so that ```my-reduce``` returns 0 for an empty collection. If the collection is not empty, we need a function that will increment the count result each time that the function is called. The function will take two arguments, the count result which is initially 0 and the collection, but it is only concerned with the result, which it will increment by 1 every time that it is called. Since the function passed to ```my-reduce``` doesn't care about the collection, we can use an ```_``` rather than name it. If we put all this together then we get:

<pre><code>(defn my-count [coll]
	(my-reduce (fn [result _] (inc result)) 0 coll))</code></pre>

This successfully passes the test suite, and is now much cleaner than the code we had before.

<a name="filter"></a>
<b>FILTER</b>

```filter``` returns a lazy sequence of the items in a ```coll``` for which a predicate (```pred```) returns true. The predicate needs to be side-effect free. If a collection is not passed then ```filter``` will return a transducer, however, we will keep this outside of the scope of this post.

We can build up a recursive function in a similar way that we did for ```my-count```. If there are no items in our ```coll``` then intuitively we might expect that we could just return the ```coll```, or even ```(empty coll)``` (this returns an empty collection of the same type as the input ```coll```). However, ```filter``` does not necessarily return the same data type as the input, and instead we need to return a lazy sequence.

We can write a first test, where we use the predicate ```zero?``` (will return true if the item is 0) and pass an empty vector. The output should be of class clojure.lang.LazySeq, and have zero items. Since we have already written ```my-count``` let's use this to count the nuber of items in the output collection.

<pre><code>	(it "result empty lazy sequence when filtering for zero on an empty vector"
		(should= clojure.lang.LazySeq (class (my-filter zero? [])))
		(should= 0 (my-count (my-filter zero? []))))</code></pre> 

We can get these tests to pass just by returning ```(lazy-seq coll)```. The clojure function <a href="https://clojuredocs.org/clojure.core/lazy-seq">lazy-seq</a> <i>takes a body of expressions that returns an ISeq or nil, and yields a Seqable object that will invoke the body only the first time seq
is called, and will cache the result and return it on all subsequent seq calls.</i> We can modify or extend our tests to confirm that this works with all clojure collections, including lists, maps, sets and strings.

Let's now add a test that is going to filter a collection that contains items:

<pre><code>(it "result even numbers when filtering for even numbers"
		(should= '(0 2 4 6 8) (my-filter even? (range 10))))</code></pre>

As we move to consider collections that contain items we can start our recursive call by setting ```input``` to the ```coll``` that we passed in and ```result``` to ```[]```. We could have ```result``` initialised with any collection, or indeed an empty collection of the same type as the input, but by using a vector we can use ```conj``` to add items to the end, and not worry that different data sets use ```conj``` in different ways. 

We will then consider each item in turn, recursing with ```(rest input)```, until the collection is empty, at which point we want to return the ```result``` as a lazy sequence. So, what do we do with each item? We want to check if ```pred``` on the item returns true, and if it does ```conj``` the item to our ```result``` and recurse this, and if not just recurse the unaltered ```result```. 

<pre><code>(defn my-filter [pred coll]
	(loop [input coll result []]
		(if (empty? input)
			(lazy-seq result)
			(recur 	(rest input) 
				(if (pred (first input))
					(conj result (first input)) 
					result)))))</code></pre>

This passes, and we can add some further tests to check that our implementation behaves as ```filter``` should:

<pre><code>(it "filter strings"
		(should= 
			'("a" "b" "n" "f" "q") 
			(my-filter #(= 1 (my-count %)) ["a" "aa" "b" "n" "f" "lisp" "clojure" "q" ""])))

	(it "filter maps"
		(should= 
			'([:c 101] [:d 102]) 
			(my-filter #(> (second %) 100) {:a 1 :b 2 :c 101 :d 102 :e -1})))</code></pre>


The tests pass, but we are just converting the ```result``` to a lazy sequence at the end, whereas the real purpose of using a lazy seq is so that the computation can be <a href="http://noobtuts.com/clojure/being-lazy-in-clojure">lazy</a>.

We could move our lazy-seq to the front of the recur loop, but we would still not be behaving lazily. Every time we loop through we evaluate if the predicate is true, and we update our ```result``` accordingly. If we were lazy we would not evaluate until we reached the tail of the recursion.

In terms of laziness, there is a <a href="http://stackoverflow.com/questions/12389303/clojure-cons-vs-conj-with-lazy-seq">difference between ```conj``` and ```cons```</a>. The behaviour of ```conj``` depends on the collection type, so will need to be realised immediately. However, ```cons``` adds an item to the start of a collection, with everything else coming after, and so can be evaluated lazily. 

Let's re-write our ```my-filter``` function so that it is lazy. We can start with ```lazy-seq``` and pass it a ```when``` block, using the predicate ```(seq coll)```, as it's body of expression. The clojure function <a href="https://clojuredocs.org/clojure.core/seq">seq</a> <i>returns a seq on the collection. If the collection is empty it returns nil, and (seq nil) returns nil.</i> So, the ``when``` block will exit with an empty collection. If we do have a sequence we want to apply the filter predicate to the first item of the collection. If the predicate returns true we want to ```cons``` the ```(first coll)``` onto the filtered collection to come, ```(my-filter pred (rest coll))```. If the predicate returns false then we simply call ```my-filter``` with the ```(rest coll)```. Nothing will get evaluated until we reach the tail of the recursion, when the collection is empty. If we put this altogether we get:

<pre><code>(defn my-filter [pred coll]
	(lazy-seq 
		(when (seq coll)
			(if (pred (first coll))
				(cons (first coll) (my-filter pred (rest coll))) 
				(my-filter pred (rest coll))))))</code></pre>

and our test suite still passes. We are now evaluating lazily, and we have our basic implementation of ```filter```. So, what have we learnt from writing our own version? Well, ```filter``` is evaluated lazily, which can give us efficiency benefits, but it also means that the collection type we return with our filtered data set is different from the input type. If we want to continue processing with the same collection type that we had then we will have to pass the output from filter into the required collection type, e.g. ```(into [] (my-filter even? [0 1 2 3 4 5]))``` will return ```[0 2 4]```.

<a name="map"></a>
<b>MAP</b>

<i>Returns a lazy sequence consisting of the result of applying f to
the set of first items of each coll, followed by applying f to the
set of second items in each coll, until any one of the colls is
exhausted.  Any remaining items in other colls are ignored. Function
f should accept number-of-colls arguments. Returns a transducer when
no collection is provided.</i>

Again, for the purpose of this blog post we are not going to consider the situation where no collection is passed. If we first look at the situation where a single collection is passed to our map function then we can build things up in a similar way as ```my-filter```, and again test that we return an empty lazy sequence if we pass an empty collection.

<pre><code>(describe "test my-map function"

	(it "result empty lazy sequence when mapping with zero? on an empty vector"
		(should= clojure.lang.LazySeq (class (my-map zero? [])))
		(should= 0 (my-count (my-map zero? [])))))</code></pre>

We can use the same function structure as we had in ```my-filter```, but in the ```when``` block we simply need to ```cons``` the result from applying the function ```f``` to the first item to everything that will follow. This will then all evaluate when we reach the tail position of the recursion, which is when we have an empty collection.

<pre><code>(defn my-map [f coll]
		(lazy-seq (when (seq coll)
				(cons (f (first coll)) (my-map f (rest coll))))))</code></pre>

We can add tests for collections containing items, and these still pass:

<pre><code>	(it "result 1 for my-map with identity on vector of 1"
		(should= '(1) (my-map identity [1])))

	(it "maps across a range"
		(should= '(1 2 3 4 5 6 7 8 9 10) (my-map inc (range 0 10))))

	(it "maps with strings"
		(should= '("Hi A!" "Hi B!" "Hi C!") (my-map #(str "Hi " % "!" ) ["A" "B" "C"])))

	(it "maps with hash-maps"
		(should= '("three" "two") (filter identity (my-map {2 "two" 3 "three"} [5 3 2]))))</code></pre>

Now we can consider the situation where two collections are passed to ```my-map```. We need to apply the function to the first items of each collection, and then to the second, etc, and continue whilst both collections are sequences (non-empty). Let's add a couple of tests:

<pre><code>	(it "maps two vectors"
		(should= '(4 6) (my-map + [1 2] [3 4])))

	(it "stops when one list completed"
		(should= '(6 16) (my-map #(* 2 %1 %2) [1 2] [3 4 5 6])))</code></pre>

We can extend ```my-map``` to also take ```[f c1 c2]```, and we want to continue ```when``` both ```c1``` and ```c2``` are sequences. We need to apply ```f``` to the first item of both ```c1``` and ```c2``` and then recur with the rest of both collections.

<pre><code>(defn my-map 
	([f coll]
		(lazy-seq (when (seq coll)
				(cons (f (first coll)) (my-map f (rest coll))))))
	([f c1 c2]
		(lazy-seq (when (and (seq c1) (seq c2))
				(cons (f (first c1) (first c2)) (my-map f (rest c1) (rest c2)))))))</code></pre>

This works! So, how about when we have more than two collections. Let's add more tests:

<pre><code>	(it "maps three lists"
		(should= '(9 12) (my-map + '(1 2) '(3 4) '(5 6))))

	(it "maps four vectors"
		(should= '(16 20) (my-map + [1 2] [3 4] [5 6] [7 8])))</code></pre>

Now we can again extend ```my-map```, and this time take in any number of arguments ```[f c1 c2 & more]```. To handle an unknown number of arguments we can start a ```recur``` loop, setting ```c1``` to the first collection passed in, ```c2``` to the second collection, and ```r``` to the rest. We will then ```recur```, passing ```(my-map f c1 c2)``` back in as ```c1```, ```(first r)``` as ```c2``` and ```(rest r)``` as ```r```, and continue this until we only have two collections left. At this point ```(seq r)``` will return nil and we can escape from our recursive loop. We can then evaluate the final result by calling ```(my-map f c1 c2)```.

<pre><code>(defn my-map 
	([f coll]
		(lazy-seq (when (seq coll)
				(cons (f (first coll)) (my-map f (rest coll))))))
	([f c1 c2]
		(lazy-seq (when (and (seq c1) (seq c2))
				(cons (f (first c1) (first c2)) (my-map f (rest c1) (rest c2))))))
	([f c1 c2 & more]
		(loop [c1 c1 c2 c2 r more]
			(if (empty? r)
				(my-map f c1 c2)
				(recur (my-map f c1 c2) (first r) (rest r))))))</code></pre>

This now passes our test suite, but what about non-commutative functions? 

<pre><code>	(it "maps with non-commutative functions"
		(should= '([:a :d :g] [:b :e :h] [:c :f :i]) 
			(apply my-map vector [[:a :b :c][:d :e :f][:g :h :i]])))</code></pre>

This test fails, because we get a lazy sequence of vectors by calling ```(my-map f c1 c2)```, and this then becomes ```c1``` for the next iteration, resulting in a multi-dimensional vector. So, how do we solve this?

What we want is to convert the input collections to a single sequence, where the first item is a collection of all the first elements, the second item is a collection of all the second elements, etc. If we have this collection of reordered collections we can just map the result of applying the function to each collection in turn. We can create a single collection by adding the first collection, ```c1```, to the other collections, ```colls```. Let's then pass this concatenated collection to a function named reorder. 

We need ```reorder``` to take the first item from each collection and map it to a new collection, and add this to all the other reordered collections to come until one of the collections is empty. If we put this all together we get:

<pre><code>(declare my-map)

(defn reorder [c]
        (when (every? seq c)
           	(cons (my-map first c) (reorder (my-map rest c)))))

(defn my-map 
	([f coll]
		(lazy-seq (when (seq coll)
				(cons (f (first coll)) (my-map f (rest coll))))))
	([f c1 & colls]
     		(my-map #(apply f %) (reorder (cons c1 colls)))))</code></pre>

Note that becuase the functions ```my-map``` and ```reorder``` refer to each other we need to ```declare my-map``` at the top of the file.

We now have much cleaner code, and it works for non-commutative functions!

<a name="pmap"></a>
<b>PMAP</b>

Finally we are going to look at ```pmap```, which is <i>like ```map```, except ```f``` is applied in parallel</i>. This function is <i>only useful for computationally intensive functions where the time of f dominates the coordination overhead</i>.

We need ```my-pmap``` to return the same result as ```map```, so we can write a first test,

<pre><code>	(it "result as map"
		(should= (map inc [1 2]) (my-pmap inc [1 2])))</code></pre>

and write the function to just call ```my-map```.

<pre><code>(defn my-pmap
	([f coll]
		(my-map f coll)))</code></pre>

To mimic a computationally intense function, we can write a long-running function by inserting a sleep (```Thread/sleep 1000``` will cause the thread to block for 1000 ms), and then adding 10 to the input:

<pre><code>(defn long-running-job [n]
    (Thread/sleep 1000)
    (+ n 10))</code></pre>

We can test to check that the long-running-function gives the correct result, and also that it takes more that 1 s to run for each number (the 1 s sleep time plus some execution time) by using the system time, and checking it before and after calling the function:

<pre><code>	(it "test long-running-job function"
		(should= '(11 12) (my-pmap long-running-job [1 2])))

	(it "test long-running-job time"
		(should (< 1.0 (let [st (System/nanoTime)]
					(long-running-job 1)
					(/ (- (System/nanoTime) st) 1e9)))))</code></pre>

If we try to do the same thing for ```(map long-running-fuction [1 2 3 4])``` the test will fail. We might expect this function to take a little over 4 s to run (the ```long-running-function``` is called four times by ```map```), but it actually completes in a fraction of a second. Why? Remember that ```map``` is lazy, and the time we are measuring is the time to make a new lazy-seq, not the time to actually execute it. Let's write a test where we call a function named ```test-time``` and pass it a map type, function and collection:

<pre><code>(it "test time long-running job with map"
		(should (< 4.0 (test-time map long-running-job [1 2 3 4]))))</code></pre>

We can write ```test-time```, and measure the time to evaluate ```realize-lazy-seq```. ```realize-lazy-seq``` will recursively loop through the lazy sequence after each part is evaluated. 

<pre><code>(defn realize-lazy-seq [map-type f c]
	(loop [res (map-type f c)]
    		(when res (recur (next res)))))

(defn test-time [map-type f c]
	(let [st (System/nanoTime)]
		(realize-lazy-seq map-type f c)
			(/ (- (System/nanoTime) st) 1e9)))</code></pre>

Now when we run our tests they pass, so let's put in a new test where we will ```test-time``` for ```my-pmap```, using the same function and collection as before. If our implementation of pmap works then it should complete in less than 4 s. Indeed, if it completes each function in parallel then it should take just over 1 s.

<pre><code>(it "test time long-running job with my-pmap"
		(should (> 4.0 (test-time my-pmap long-running-job [1 2 3 4]))))</code></pre>

This fails, because currently ```my-pmap``` is just calling ```my-map```, so it will apply ```f``` to each item in the collection in turn. What we want is for ```f``` to be applied in parallel, and we can achieve this using <a href="https://clojuredocs.org/clojure.core/future">clojure futures</a>. If we invoke ```future``` every time we call ```f``` on each item, they will be evaluated in an available thread from the thread pool and the results cached until we call them with ```deref```, and so in this way we can utilise multiple threads.

We can set a local variable name ```results``` to be ```(my-map #(future (f %)) coll)```. This will generate a lazy sequence of future results, where ```f``` has been applied to each item in ```coll``` as a future. We might then imagine that we could map the futures to a new lazy sequence by applying the function ```deref``` to ```results``` to return the evaluations.

<pre><code>(defn my-pmap
	([f coll]
		(let [results (my-map #(future (f %)) coll)]
				(my-map deref results))))</code></pre>

However, when we run our tests the last test fails. We can modify the test by wrapping the function that we are calling in ```time```, and this will print the execution time:

<pre><code>(it "test time long-running job with my-pmap"
		(should (> 1.1 (time (test-time my-pmap long-running-job [1 2 3 4])))))</code></pre>

Now when we run the test it tells us that it takes just over 4000 ms to execute - the same time that it takes ```map```. So, what is wrong? Again, it is all down to lazy sequences. When we set ```results``` we are just generating the lazy sequence, and not evaluating it. We only evaluate ```results``` when we call ```deref```, so each future is generated and then immediately dereferenced, with the thread blocked until the result is available. In this way, we generate the result of applying the function to each item in the array in sequence, and not in parallel.

To make the function work in parallel we need to generate all of the futures before we start to dereference. We can do this by recursively iterating through the collection, applying ```f``` as a future to each item and adding this to an accumulator, ```acc```. Because we are not generating a lazy sequence, we will initiate each future as we iterate through. By starting the accumulator as an empty vector we can ```conj``` the futures into ```acc```, and then return ```acc``` as the escape from the recursion after we have iterated through all items (```(seq remaining)``` will return nil for empty collections). Now we have a collection of futures, which we can call ```results```, and so we can map this collection by applying ```deref``` to return the futures.

<pre><code>(defn my-pmap
	([f coll]
		(let [results
			(loop [remaining coll acc []]
				(if (seq remaining)
					(recur (rest remaining) (conj acc (future (f (first remaining)))))
					acc))]
			(my-map deref results))))</code></pre>


Our tests now pass! Indeed, our implementation ```my-pmap``` completes the test in just over 1 s, compared to 4 s for ```map```, so we could change our test requirement to return in less than 1.1 s.

Does what we are doing in ```results``` look familiar? We are taking an input collection, applying a function to each element in turn, and returning a single item - a vector. We know that we can perform this type of data manipulation with ```my-reduce```, so let's refactor. Our initial ```val``` is the empty vector, and we want to ```conj``` the ```future``` of applying ```f``` to each element of the ```coll``` into ```val```.

<pre><code>(defn my-pmap [f coll]
		(let [results (my-reduce #(conj %1 (future (f %2))) [] coll)]
			(my-map deref results)))</code></pre> 

Our refactored code using ```my-reduce``` looks cleaner, and the tests still pass. Let's now make sure that ```my-pmap``` can work with multiple collections. We can write tests that will map a function over two collections and check that they return the correct result and in a time that confirms the functions were applied in parallel.

<pre><code>	(it "maps two vectors"
		(should= '(14 16) (my-pmap long-running-job [1 2] [3 4])))

	(it "test time long-running job with my-pmap and two collections"
		(should (> 1.1 (time (test-time my-pmap long-running-job [1 2 3 4][1 2 3 4])))))</code></pre>

We could get these tests to pass simply by extending ```my-pmap``` to take two collections, and using the same logic that we did for a single collection. However, this approach will suffer the same limitations as we saw for the first implementation of ```my-map```. So let's instead use the same approach that we did for ```my-map```, and convert out collections into a single sequence of reordered collections. Indeed, we can do exactly the same as we did for <a href="#map">```my-map```</a> and also reuse the ```reorder``` function.

<pre><code>(defn my-pmap
	([f coll]
		(let [results (my-reduce #(conj %1 (future (f %2))) [] coll)]
			(my-map deref results)))
	([f c1 & colls]
    		(my-pmap #(apply f %) (reorder (cons c1 colls)))))</code></pre>

This works, and we can extend our test suite to ensure that it continues to work with more collections.

<br>
<b>CONCLUSIONS</b>

As an experienced skydiver you may feel confident about taking a first <a href="https://en.wikipedia.org/wiki/BASE_jumping">BASE jump</a>. However, there are a lot of considerations, a lot that you need to understand, before you can take the leap in relative safety. The higher altitude of a skydive means you can reach faster speeds before deploying your parachute, whereas it is normally not possible to reach terminal velocity during a BASE jump. The lower airspeeds during a BASE jump mean jumpers have less aerodynamic control of their bodies, so a clean parachute deployment is more difficult. The lower control and jumping close to a structure also vastly increases the risk of a collision, both before and after parachute deployment. As such, the inputs to the jump - the position of the jumper as they leap and the equipment that they use - are critical to a successful flight.

The same requirement to really understand how everything is working is equally as relevent to software development, or any other activity for that matter, although the consequences might not be as immediately life-impacting.

Every day when we write code, in whichever language we chose, our life is made easier and more efficient by the use of the core functions of the language. However, reliance on the hidden code comes at a risk, and if you don't fully understand what is happening then you risk undesired outcomes, and you may also be missing out on additional functionality. We should all strive to unveil the magic at the surface, and really dig deep so as to better know our craft.

The source code for my implementations of <a href="#reduce">reduce</a>, <a href="#count">count</a>, <a href="#filter">filter</a>, <a href="#map">map</a> and <a href="#pmap">pmap</a> are all available on <a href="https://github.com/jonathangraham/clojure_functions">github</a>.
