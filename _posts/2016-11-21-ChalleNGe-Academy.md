---
layout: post
title: Basic Education Should Include Coding
tagline: A Weekend Teaching at the Mountaineer Challenge Academy   
include_social: true
---
{% include JB/setup %}

The <a href="http://www.wvchallenge.org/">Mountaineer Challenge Academy</a> "give[s] academically challenged teens a second chance at obtaining their basic education". To me, basic education should include, at a minimum, some exposure to code. The Cadets, who range in age from 16 - 18, attend the 22 week residential bootcamp with the goal of obtaining their High School Diploma, as well as finding a path forward for their future. 

About a month ago, I was invited by <a href="http://www.nationalguard.mil/portals/31/Features/ngbgomo/bio/1/1892.html">General Hoyer</a>, the Adjutant General of the National Guard in West Virginia, to visit the Mountaineer Challenge Academy. General Hoyer is passionate about developing sustainable futures for the students, and sees programming as a great opportunity to create growth and employment in West Virginia. He asked <a href="http://www.minedminds.org/">Mined Minds</a> to build a computer programming curriculum and to show the Cadets the power of code. How could we say no to giving at-risk children the chance to see that a career in the tech industry is an opportuity that is open to them?

With only a few weeks left before graduation for this cohort, we didn't have time to teach a full curriculum, so instead chose to lead an intensive weekend of coding to give them a taste of what it is all about. That is how <a href="https://twitter.com/pandamonial">Amanda</a> and I spent this last weekend.

We started the weekend by playing a game of <a href="https://en.wikipedia.org/wiki/Hangman_(game)">hangman</a> on a whiteboard, and then talked about how we could program that game. Together, we were able to quickly break the game down into the component steps, like storing a word in memory, updating blanks with letters guessed, keeping track of incorrect guesses, etc. We debated whether drawing a man hanging was a good thing, and discussed how we could build all the logic of the game without needing to worry about the user interface at all. In the end, the students decided to keep with the traditional visulisation of the game.

Once we had all the component parts of the program written on paper, we all coded them, one by one, in Ruby. How did we know the code was doing what we wanted? We built up our program using a <a href="https://en.wikipedia.org/wiki/Test-driven_development">TDD</a> approach. Because each method was only doing one specific thing, the students were able to explain in English what each one needed to do, and so writing the code only required some small assistence with the syntax. And becuase each method was at most only a few lines long, it was easy to debug - a must when there is a classroom full of new coders writing their first program!

Day one left the students exhausted, but all the backend work was worth it when it came together on Sunday. We used Sinatra to build the web app, calling in all the logic that we had built the previous day. As always with teaching, the "aha" moments are extremely rewarding, and it was exciting for everyone when the students saw in the browser the result of changes that they made in their code.

Due to access restrictions, we did not have the students push their code to github, but at the end of the day we committed a <a href="https://github.com/MinedMindsFoundation/wvchallenge">representative example</a>, and deployed the app to heroku so that everyone can <a href="https://wvchallenge.herokuapp.com">play the game</a>.  

The Cadets still had lots of features that they wanted to add - like displaying the word at the end of the game, making the choices case insensitive, storing game results, having different difficulty levels - and several were excited about exploring the potential of a future career in code.

We covered a lot in the weekend. The Cadets wouldn't be able to write the code from scratch on their own again, but that wasn't the point. They saw how to break down a problem, and that coding each individual part was not that complicated. Not all of them would do it again - for some spending full days at the computer would not be their choice - but others saw an opportunity that they never thought they could do or that would be fun.  

If you've never done any teaching before and you have some free time then I'd certainly recommend that you consider giving it a go. You learn a lot, it can be very rewarding, and is often a lot of fun. Feel free to get in touch with me if you'd like any advice on how to get started.
<hr>
Mined Minds is a 501c3 non-profit. If you'd like to support our work in creating software development career opportunities for those that would not traditionally get the chance, then you can do so <a href="http://www.minedminds.org/donate">here</a>.