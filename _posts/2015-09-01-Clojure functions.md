---
layout: post
title: Understanding core Clojure Functions
tagline: Learning a language by implementing the core features   
include_social: true
---
{% include JB/setup %}

###Introduction

There are lots of approaches to learning a new language, and [Clojure](https://clojure.org/) has many great resources. For those that are active learners, the [Clojure koans](http://clojurekoans.com/) and [4Clojure](https://www.4clojure.com/) exercises provide a wonderful opportunity to explore the language. There are also plenty of books to get you started, including [Carin Meier's](https://twitter.com/gigasquid) [Living Clojure](http://shop.oreilly.com/product/0636920034292.do) that takes a very hands-on approach to learning (read a [review](http://jonathangraham.github.io/2015/07/28/Book%20Review%20for%20Living%20Clojure/) for more details).  

Another approach when trying to gain a fundamental understanding of how a language works is to implement your own versions of some of the core features. In this blog post we will be focusing on Clojure, but the approach is equally valid for all languages.

We will implement models of `reduce`, `count`, `filter`, `map`, and `pmap`, and in doing so will explore, amongst other things, recursion, lazy sequences, and futures. The goal for the models of the core functions that we implement is to generate the same output as those produced by the core language, given any valid input. For the purpose of this blog, we will not be considering the processing efficiency of the functions that we write.


###REDUCE

`reduce` is the backbone of many of the Clojure *sequence* functions. Before we start implementing it, what exactly is a sequence?

Alex Miller wrote a very clear [introduction to sequences](http://insideclojure.org/2015/01/02/sequences/). They are essentially *the key abstraction that connects immutable persistent collections and the sequence library*. The key sequence abstraction functions are `first`, `rest`, and `cons`. `(first coll)` will return the first item of the collection; `(rest coll)` returns everything but the first item, and an empty collection if there are no more items; `(cons item coll)` constructs a new sequence with the item prepended to coll. 

Calling `sequence` on a seqable - a collection, or other thing, that can produce a sequence - returns a sequence. Calling `seq` has the same effect, except that empty collections will return `nil`, rather than an empty sequence. Seqable collections include lists, vectors, sets, and maps. The sequence abstraction enables sequence functions - those that implement the sequence abstraction, including `reduce` and `map` - to handle any seqable data structure. This means that we can use the same function with any seqable collection. The sequence functions implicitly convert the input collection to a sequence.  

Now we know what a sequence is, we can look at `reduce`, and implement our own version, `my-reduce`.

From the [Clojure docs](https://clojuredocs.org/clojure.core/reduce):

`(reduce f coll)` `(reduce f val coll)`
*f should be a function of 2 arguments. If val is not supplied, returns the result of applying f to the first 2 items in coll, then applying f to that result and the 3rd item, etc. If coll contains no items, f must accept no arguments as well, and reduce returns the result of calling f with no arguments.  If coll has only 1 item, it is returned and f is not called.  If val is supplied, returns the result of applying f to val and the first item in coll, then applying f to that result and the 2nd item, etc. If coll contains no items, returns val and f is not called.*

We can start with the case where `f` has two arguments: `val` and `coll`. We need to apply `f` to `val` and `(first coll)`, and then apply `f` to this result and the second item, etc. We can do this by recursively calling `my-reduce` with `f`, `(f val (first coll))`, and `(rest coll)`, and continuing until the collection is empty. To escape the recursion, we can utilize our knowledge that calling `seq` on an empty seqable returns `nil`, which is falsy, whereas calling `seq` on a seqable with at least one item returns a sequence, which is truthy. At this point we return `val`, which is our reduced value from iterating through the collection. If we initially passed an empty collection we immediately return `val`, without `f` being called.

	(defn my-reduce [f val coll]
		(if (seq coll)
			(my-reduce f (f val (first coll)) (rest coll))
			val))   

This will work fine unless the collection gets too big. The recursion consumes stack space, and so eventually we will generate a stack overflow. For example, we generate a StackOverflowError if we run `(my-reduce + 0 (range 10000))`.

We can avoid this by using the `recur` special operator, which does constant-space recursive looping by rebinding and jumping to the nearest enclosing loop or function frame, and so giving us tail-call optimization. To gain this optimization, all we need to do is replace the `my-reduce` call with `recur`. With this change made, we can safely run `(my-reduce + 0 (range 1000000000))` and beyond.

Let's move to the situation where `val` is not supplied. In this case `reduce` should return the result of applying `f` to the first 2 items in `coll`, then apply `f` to that result and the 3rd item, etc.

To allow for this, we can add an arity taking in just `f` and `coll`, which then calls `my-reduce`, passing in `(first coll)` to `val`, and `(rest coll)` to `coll`. Our `my-reduce` function already works when there is an empty `coll`, so this should work as long as we have at least one item in our collection.

Finally, if we pass in an empty collection and no initial value, we need to return the result of evaluating `f` with no arguments. Again, we can use `(seq coll)` to direct the path of execution.

	(defn my-reduce 
		([f coll]
			(if (seq coll)
		 		(my-reduce f (first coll) (rest coll))
		 		(f)))
		([f val coll]
			(if (seq coll)
				(recur f (f val (first coll)) (rest coll))
				val)))

A consequence of `reduce` evaluating `f` with no arguments is that functions like `-` need to be passed with care. If you want to use `reduce` and have a function that cannot be evaluated with zero arguments, then either ensure the situation with no initial value and an empty collection cannot happen, or modify the function so that it can be evaluated with no arguments. Alternatively, we now know that we can write our own `reduce` function, and could modify it so that it defines differently what to do when passed a function with just an empty collection.

Another thing we may have learned about `reduce` is that even if we have a function that requires two arguments, we know that we can successfully pass this to `reduce` with only a single collection. We can demonstrate this with `add`: `(defn add [x y] (+ x y))`. If we just pass a single argument to `add` we will get an argument error. However, calling `(reduce add [1])` will return 1, because the first item of the collection will be defined as `val`, and given the rest of the collection is empty, `reduce` will just return `val` with the function not called.

###COUNT

Many other functions become easy to write now that we have written a version of `reduce`. We can demonstrate this with `count`:

`(count coll)`
*Returns the number of items in the collection. (count nil) returns 0. Also works on strings, arrays, and Java Collections and Maps.*

So, we need to iterate through a collection, increasing a counter for each item, and return the single count value. We have already seen how to reduce a collection to a single value, so let's use `my-reduce`to define `my-count`. 

We need to set an initial `val` to 0, so that `my-reduce` returns 0 for an empty collection. If the collection is not empty, we need a function that will increment the count result each time that the function is called. This anonymous function needs two arguments: the count result and the collection. Since the function is only concerned with the result, which will be incremented by 1 every time that it is called (`(inc result)`, we can use an `_`, rather than a name, for the collection. If we put this all this together we get:

	(defn my-count [coll]
		(my-reduce (fn [result _](inc result)) 0 coll))

###FILTER

`(filter pred)` `(filter pred coll)`
*Returns a lazy sequence of the items in coll for which (pred item) returns true. pred must be free of side-effects. Returns a transducer when no collection is provided.*

A lazy sequence can be created by the macro `lazy-seq`. This takes a body of expressions that returns an ISeq or nil, and yields a Seqable object that will invoke the body only the first time seq is called, and will cache the result and return it on all subsequent seq calls.

To demonstrate how lazy sequences work, let's first build a function that doesn't implement them correctly. 

We can build a recursive function, and start our recursive call by setting `input` to the `coll` that we pass in, and `result` to an empty vector. We could have `result` initialised with any collection, or indeed an empty collection of the same type as the input, but by using a vector we can use `conj` to add items to the end, and not worry that different data sets use `conj` in different ways. 

We will then consider each item in turn, recursing with `(rest input)`, until the collection is empty, at which point we want to return the `result` as a lazy sequence: `(lazy-seq result)`. So, what do we do with each item? We want to check if `pred` on the item returns true. If it does we `conj` the item to our `result` and recurse this, and if not we just recurse the unaltered `result`. 

	(defn my-filter [pred coll]
		(loop [input coll result []]
			(if (seq input)
				(recur 	(rest input) 
					(if (pred (first input))
						(conj result (first input)) 
						result))
				(lazy-seq result))))

This generates the correct results, but we are just converting the `result` to a lazy sequence at the end. What we want is for the computation to be lazy.

We could move `lazy-seq` to the front of the recur loop, but we would still not be behaving lazily. Every time we loop through we evaluate if the predicate is true, and we update our `result` accordingly. If we were lazy we would not evaluate until we reached the tail of the recursion.

Let's look at the consequence of not being lazy. `range` will generate a lazy-seq of nums. If we pass it an end number it will produce a range from 0 to end (exclusively). So, if we run `(my-filter even? (range 10))` we return a lazy sequence `(0 2 4 6 8)`. We can just return a collection containing the first two elements with `(take 2 (my-filter even? (range 10)))`, which returns `(0 2)`. What if we don't pass an end number to `range`: `(take 2 (my-filter even? (range)))`? We will produce an infinite lazy-seq, and our `my-filter` will try to evaluate it all before taking the first two elements. Evaluating an infite sequece takes a long time! If we were lazy we would just need to evaluate as needed, so just enough to generate two filtered items, rather than the entire infinite sequence.

In terms of laziness, there is a difference between `conj` and `cons`. The behaviour of `conj` depends on the collection type, so will need to be realised immediately. However, `cons` adds an item to the start of a collection, with everything else coming after, and so can be evaluated lazily. 

Let's re-write our `my-filter` function so that it is lazy. We can start with `lazy-seq` and pass it a `when` block, using the predicate `(seq coll)`, as it's body of expression. As before, the `when` block will exit with an empty collection. If we do have a sequence, we want to apply the filter predicate to the first item of the collection. If the predicate returns true we can `cons` the `(first coll)` onto the filtered collection to come: `(cons (first coll) (my-filter pred (rest coll)))`. If the predicate returns false then we simply call `my-filter` with the `(rest coll)`. Nothing will get evaluated until we reach the tail of the recursion, when the collection is empty. If we put this altogether we get:

	(defn my-filter [pred coll]
		(lazy-seq 
			(when (seq coll)
				(if (pred (first coll))
					(cons (first coll) (my-filter pred (rest coll))) 
					(my-filter pred (rest coll))))))

We are now evaluating lazily, and we have our basic implementation of `filter`. Now if we run `(take 2 (my-filter even? (range)))` we get our desired answer returned almost immediately.

For the purposes of this blog we will not cover the scenario where no collection is provided to filter, which would return a transducer. Look out for a future post on the subject of transducers.

One thing to note about `filter` is that it will return a lazy seq, rather than the input collection type. If we want to continue processing with the same collection type as the input, we will have to pass the output from filter `into` the required collection type, e.g. `(into [] (my-filter even? [0 1 2 3 4 5]))` will return `[0 2 4]`.

###MAP

`(map f)` `(map f coll)` `(map f c1 c2)` `(map f c1 c2 c3)` `(map f c1 c2 c3 & colls)`
*Returns a lazy sequence consisting of the result of applying f to the set of first items of each coll, followed by applying f to the set of second items in each coll, until any one of the colls is exhausted.  Any remaining items in other colls are ignored. Function f should accept number-of-colls arguments. Returns a transducer when no collection is provided.*

Again, for the purpose of this blog post we are not going to consider the situation where no collection is passed. 

For the case where we have a single collection, we can use the same function structure that we had in `my-filter`. In the `when` block we simply need to `cons` the result from applying the function `f` to the first item to everything that will follow. This will then all evaluate when we reach the tail position of the recursion, which is when we have an empty collection.

	(defn my-map [f coll]
			(lazy-seq (when (seq coll)
					(cons (f (first coll)) (my-map f (rest coll))))))

We could extend `my-map` to also take `[f c1 c2]`. We would need to apply `f` to the first item of both `c1` and `c2` and then recur with the rest of both collections until one of them is empty.

	(defn my-map 
		([f coll]
			(lazy-seq (when (seq coll)
					(cons (f (first coll)) (my-map f (rest coll))))))
		([f c1 c2]
			(lazy-seq (when (and (seq c1) (seq c2))
					(cons (f (first c1) (first c2)) (my-map f (rest c1) (rest c2)))))))

Given we cannot write arities for all possible numbers of collections, we instead need to write one that can take any number of arguments.

If we could convert the input collections to a single sequence, where the first item is a collection of all the first elements, the second item is a collection of all the second elements, etc, we could just map the result of applying the function to each collection in turn. 

To do this, we can first create a single collection by adding the first collection, `c1`, to the other collections, `colls`. We can then pass this concatenated collection to a function named reorder. 

We need `reorder` to take the first item from each collection and map it to a new collection, and add this to all the other reordered collections to come until one of the collections is empty. Since the two functions, `my-map` and `reorder`, are mutually recursive, we need to delare them. If we put this all together we get:

	(declare my-map)

	(defn reorder [c]
	        (when (every? seq c)
	           	(cons (my-map first c) (reorder (my-map rest c)))))

	(defn my-map 
		([f coll]
			(lazy-seq (when (seq coll)
					(cons (f (first coll)) (my-map f (rest coll))))))
		([f c1 & colls]
	     		(my-map #(apply f %) (reorder (cons c1 colls)))))

###PMAP

Finally we are going to look at `pmap`.

`(pmap f coll)` `(pmap f coll & colls)`
*Like map, except f is applied in parallel. Semi-lazy in that the parallel computation stays ahead of the consumption, but doesn't realize the entire result unless required. Only useful for computationally intensive functions where the time of f dominates the coordination overhead.*

We can apply `f` in parallel using Clojure [futures](https://clojuredocs.org/clojure.core/future). If we invoke `future` every time we call `f` on each item, they will be evaluated in an available thread from the thread pool, and the results cached until we call them with `deref`. In this way we can utilise multiple threads.

We can set a local variable name `results` to be `(my-map #(future (f %)) coll)`. This will generate a lazy sequence of future results, where `f` has been applied to each item in `coll` as a future. We might then imagine that we could map the futures to a new lazy sequence by applying the function `deref` to `results` to return the evaluations.

	(defn my-pmap
		([f coll]
			(let [results (my-map #(future (f %)) coll)]
					(my-map deref results))))

However, when we set `results` we are just generating the lazy sequence, and not evaluating it. We only evaluate `results` when we call `deref`, so each future is generated and then immediately dereferenced, with the thread blocked until the result is available. In this way, we generate the result of applying the function to each item in the array in sequence, and not in parallel.

To make the function work in parallel we need to generate all of the futures before we start to dereference, hence why `pmap` is semi-lazy. To do this we can make use of the fact that when we use `conj` the resut will be realised immediately. We can recursively iterate through the collection, applying `f` as a future to each item and `conj`ing this to an accumulator, `acc`, initialised as an empty vector. In this manner we initiate each future as we iterate through. We can then return `acc` as the escape from the recursion after we have iterated through all items. Now we have a collection of futures, which we can bind to `results`, we can map this collection by applying `deref` to return the futures.

	(defn my-pmap
		([f coll]
			(let [results
				(loop [remaining coll acc []]
					(if (seq remaining)
						(recur (rest remaining) (conj acc (future (f (first remaining)))))
						acc))]
				(my-map deref results))))

Does what we are doing in `results` look familiar? We are taking an input collection, applying a function to each element in turn, and returning a single item - a vector. We know that we can perform this type of data manipulation with `my-reduce`, so let's refactor. Our initial `val` is the empty vector, and we want to `conj` into `val` the `future` of applying `f` to each element of the `coll`.

	(defn my-pmap [f coll]
			(let [results (my-reduce #(conj %1 (future (f %2))) [] coll)]
				(my-map deref results))) 

Let's now make sure that `my-pmap` can work with multiple collections. Using the same approach that we did for `my-map`, we can convert our collections into a single sequence of reordered collections, and reuse the `reorder` function.

	(defn my-pmap
		([f coll]
			(let [results (my-reduce #(conj %1 (future (f %2))) [] coll)]
				(my-map deref results)))
		([f c1 & colls]
	    		(my-pmap #(apply f %) (reorder (cons c1 colls)))))

##Conclusions

We can learn a lot about a language by writing our own implementations of some of the core functions. In writing versions of `reduce`, `count`, `filter`, `map`, and `pmap`, not only have we seen how these work, but we have also explored a lot more of the functionality of Clojure, including the use of `recur`, `lazy-seq`, and `futures`.

How do we know the code all works as it should?

The implementations were built-up in a TDD fashion, with the unit tests written using the [speclj](http://speclj.com) framework (run with `lein spec`). Additionally, property-based tests were writen for all the functions, using [test.check](https://github.com/clojure/test.check) (run with `lein test`). You can read about the property-based tests that demonstrate that the functions behave as the core functions in the blog [Property-Based Testing in Clojure](http://jonathangraham.github.io/2016/01/07/property_based_testing_clojure_functions/).  

The source code for the above implementations, together with the unit and property-based tests, are all available on [github](https://github.com/jonathangraham/clojure_functions).


<b>EDIT May 12th, 2016:</b> Re-write of the original post to give extra clarity. 