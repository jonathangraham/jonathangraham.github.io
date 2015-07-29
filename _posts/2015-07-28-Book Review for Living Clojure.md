---
layout: post
title: Living Clojure
tagline: A Book Review of Living Clojure, by Carin Meier   
include_social: true
image: living-clojure-image
---
{% include JB/setup %}

<b>Living Clojure</b> is writen by <a href="https://twitter.com/gigasquid">Carin Meier</a>, and was published in April 2015 by <a href= "http://shop.oreilly.com/product/0636920034292.do">O'Reilly</a>. The book is available in print and ebook format, and I read the print version, which <a href="https://8thlight.com/">8th Light</a> bought for me to study.
<br><br>
The book's tagline is <i>An Introduction and Training Plan for Developers</i>, and its target audience is experienced programmers who have not worked in Clojure before. Would I consider myself an experienced programmer? Not yet, but I have some experience in several languages, and on reading the <a href="http://cdn.oreillystatic.com/oreilly/booksamplers/9781491909041_sampler.pdf">free sample chapter</a> I decided that the book would be gentle enough to allow me to follow. And I have worked in Clojure before. In fact, Clojure was the first programming language that I experienced, and I coded it live on stage as <a href="http://meta-ex.com/">Meta-eX</a>. At the time, however, I only learnt how to manipulate existing code, and not how to create well structured programs. Having now switched careers to one of a software developer, I was eager to learn what <i>Living Clojure</i> was really about.
<br><br>
A quick disclaimer: I do know Carin, and I had the privilege of sharing the stage with her at <a href="http://www.thestrangeloop.com/">Strange Loop</a> last year, as we live-coded music and robots together. Ask yourself this though: why would you not buy a book writen by someone who live-codes robots?! Robots aside, these are just my thoughts and experiences from reading <i>Living Clojure</i>, and if it sounds like it might be of interest to you then I'd urge you to read on. 
<br><br>
So, the book: what do you need to know before you start? Well, you might want to take a look at <a href="https://en.wikipedia.org/wiki/Alice%27s_Adventures_in_Wonderland">Alice in Wonderland</a>, as this is what all the examples are based on. It might be an unusual choice to theme a programming book on, and some of the examples seem a little contrived, but go with it and you'll see the logic in the literary nonsense. Other than that, if you haven't written any Clojure before just make sure you take a look at the Preface to ensure that you have your environment all setup.
<br><br>
<b>The Structure of Clojure:</b> Chapter 1 gives a gentle introduction to how Clojure is structured, from simple values like intergers and strings, to Clojure collections, like maps and vectors. It then covers creating local (<code>let</code>) and global (<code>def</code>, <code>defn</code>) bindings, and organising these within namespaces. There are limited exercises and examples in the book, so it is well worth taking the time to practice and reinforce what you have learnt by completing the <a href="http://clojurekoans.com/">Clojure Koans</a>. I had actually completed some of koans prior to starting <i>Living Clojure</i>, but I'd recommend reading Chapter 1 first, and then setting aside a few hours to complete the problems in Koans 1 - 6.
<br><br>
<b>Flow and Functional Transformations:</b> The pace begins to pick-up in Chapter 2, and starts by covering flow control with <code>if</code>, <code>when</code> and <code>cond</code>. It then moves on to currying, through the use of <code>partial</code>, and combining multiple functions with <code>comp</code>. There is a big emphasis throughout the book on writing clear and understandable code, and the section on destructuring starts to tackle this. The chapter then moves on to working with infinite lists through the use of laziness, and then to the fundamentals of recursion. Recursion is at the heart of the Clojure language, so it's worth spending the time now to make sure that you're fully onboard with the concept. The rest of the chapter looks at common abstractions that are built using recursive calls, including map, reduce and filter. Given how fundamental these expressions are for shaping and transforming collections, it is worth spending the time to write your own implementations of them, and I'll cover this in a future blog.
<br><br>
A lot is covered quickly in this chapter, and at the end Carin advises not to worry if it hasn't all clicked into place and to forge ahead anyway. This is exactly what I did, but I'd actually recommend that you stop and practice a while first. Again, the <a href="http://clojurekoans.com/">Clojure Koans</a> are great for reinforcing what you have learnt, and you should now be able to tackle Koans 7 - 14.
<br><br>
<b>State and Concurrency:</b> Things get exciting in Chapter 3, where we look at how we can manipulate state safely whilst working concurrently. The chapter begins with the <code>atom</code> for independent and synchronous state changes, and introduces multi-threading through the use of the <code>future</code> form. It quickly moves on to coordinated synchronous changes using refs, before covering the use of agents for those times when you don't need the results right away. <a href="http://clojurekoans.com/">Clojure Koans</a> 15 and 16 give exercises for some of these concepts, and I look forward to tackling more involved examples with the safe use of concurrency in Clojure.
<br><br>
<b>Java Interop and Polymorphism:</b> There is a lot of power derived from being built on the JVM, and Chapter 4 briefly explains how to interact with Java classes and libraries. It then moves on to polymorphism, which allows functions to respond differently to alternative types of input. There are multimethods and protocols in Clojure, and I chose to explore the use of multimethods when writing a <a href="https://github.com/jonathangraham/clojure_bencoder">Clojure implementation of Bencode</a>.
<br><br>
<b>How to Use Clojure Projects and Libraries:</b> I really like how easy it is to create new projects with Leiningen and to include libraries within them. The best way to learn is to do, and Chapter 5 holds your hand as you start working with the project structure. One thing that I found, though, is that it is easy to make mistakes even when following simple examples, and also that the error logs for Clojure are not straight-forward to read and understand. When you are starting out and see an incomprehensible log for the first time, take a look at <a href="https://blog.8thlight.com/connor-mendenhall/2014/09/12/clojure-stacktraces.html">Clojure Stack Traces for the Uninitiated</a>, which will help you make sense of them.
<br><br>
<b>Communication with core.async:</b> The ability to easily do asynchronous programming is really quite cool, and Chapter 6 shows you the power of <a href="https://github.com/clojure/core.async">async.core</a>, a Clojure library. Sticking with the Alice in Wonderland theme, in this chapter you walk through how to create multiple channels and execute from them asynchronously. It also explains how to create JAR files for easy sharing of your Clojure project.
<br><br>
<b>Creating Web Applications with Clojure:</b> It may not be the most exciting of examples, but Chapter 7 gives you everything that you need to build your own Web App. Follow through the example and you will have the basic project structure and also a knowledge of how it all works together, which will serve you well as a starting point for whatever you want to build. It starts off by creating a compojure project, which gives us a minimal web server, before including Ring-JSON middleware to enable us to handle JSON responses. Next, we look at ClojureScript, enabling us to use Clojure in the browser as well as on the server, and then we include cljs-http so that we can use the power of <i>core.async</i> to handle HTTP calls asynchronously.
<br><br>
I found that in order to make the example work I had to make some changes to the default middleware settings in the <i>handler.clj</i> file, so that the keys were returned as keywords rather than strings. To do this I removed <code>[ring.middleware.defaults :refer [wrap-defaults site-defaults]]</code> from the <code>:require</code> list and added <code>{:keywords? true}</code> to <code>ring-json/wrap-json-response</code> in <code>app</code>. This then gives the following for the <i>handler.clj</i> file:
<pre><code>(ns cheshire-cat.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.json :as ring-json]
            [ring.util.response :as rr]))

(defroutes app-routes
  (GET "/" [] "Hello World")
  (GET "/cheshire-cat" []
  	(rr/response {:name "Cheshire Cat" :status :grinning}))
  (route/not-found "Not Found"))

(def app
	(-> app-routes
		(ring-json/wrap-json-response {:keywords? true})
    (wrap-defaults site-defaults)
    ))</code></pre>

Finally, we use the Enfocus library to update our webpage with the data we want to display and to apply event handling and effects. The final version of our web app just displays a couple of words that then fade out at different rates, but the final few pages of the chapter point to other libraries and resources that will help you on your way to designing killer apps.
<br><br>
<b>The Power of Macros:</b> The final chapter in Part I covers macros. Some core Clojure expressions, such as <code>when</code>, are macros, and it's good going back to the source code to understand how they work. The take-home messages for me from this chapter are that macros are great for pattern encapsulation and for adding language features, but that they can easily be misused. For now, I have decided to keep this feature in my back-pocket, but it is there ready for me to pull out when the need arises.
<br><br>
<b>Part II: Living Clojure Training Program:</b> Part II of <i>Living Clojure</i> is all about practicing your skills as a Clojure developer. There is some useful information about where to go for help and resources, and how to get involved in the community, and then comes a seven-week plan of structured training. I've currently completed the first three weeks, which involve various <a href="https://www.4clojure.com/">4clojure</a> exercises to help reinforce and practice what you have learnt. You get to see other people's solutions when you have completed each exercise, and if you would like to see my solutions then you can search for <i>jpg1000</i>. For the more simple problems it is easy to just code your answer directly into the browser, but for more involved exercises I pulled the problems out into a new Leiningen project and developed my solution through a TDD approach. I look forward to completing the more project-based exercises in the rest of the training program over the following few weeks.
<br><br>
<b>CONCLUSIONS:</b> When I first started the book I was unsure about how helpful it would be. I found the pace at the start a little slow, and the examples didn't grab my attention. However, I found my learning curve quickly accelerated and I feel like the book has given me a really good foundation to move forward with Clojure development. I'd recommend anyone new to Clojure to give it a go, and make sure you give yourself plenty of time to complete the <a href="http://clojurekoans.com/">Clojure Koans</a> and <a href="https://www.4clojure.com/">4clojure</a> exercises along the way.
<br><br> 
If you'd like to buy <i>Living Clojure</i> then go to <a href= "http://shop.oreilly.com/product/0636920034292.do">O'Reilly</a>, and to connect with Carin Meier check her out on <a href="https://twitter.com/gigasquid">twitter</a>.  




