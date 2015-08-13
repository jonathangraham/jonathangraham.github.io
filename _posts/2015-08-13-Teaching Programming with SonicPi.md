---
layout: post
title: Learning to Code with Audible Programming
tagline: Writing Structured Code with SonicPi and then Playing it Live.   
include_social: true
image: SonicPi-image
---
{% include JB/setup %}

If you have never programmed before, how do you know if it is something that you could do? If you are trying to teach someone else to code, where do you start? And if we are learning new concepts, how can we create fast feedback loops?
<br><br>
In this blog post, we are going to look at how we can use <a href="http://sonic-pi.net">SonicPi</a>, an audible computing program, to explain some core programming concepts. We will use the sounds generated to give us immediate feedback on what is happening, but the focus will not be on making music, although we will touch on this at the end. To follow along, simply open up Sonic Pi on your <a href="https://www.raspberrypi.org/">Raspberry Pi</a> (it comes pre-installed on all new machines), or <a href="http://sonic-pi.net">download</a> the free app for Mac or Windows. To create this post I used Sonic Pi v2.6, and the results may look, sound and behave differently if you are using a different version. Sonic Pi uses the Ruby programming language, but it also has many domain specific features. So that you can transfer your knowledge easily from Sonic Pi to a generic Ruby environment, I have deliberately focussed on the language common to both.  
<br><br>
Before we write any code, let's spend a moment thinking about this day-to-day activity: laundry. It's not the most exciting of topics, but if you can explain how you break down the process of getting the laundry done then you can also write code. This may sound ridiculous, especially if you see programming as intimidating and alien, but being able to break down a complex system into logical, discrete steps is the key. The actual code you can learn with time and practice, but first you need to be able to think in the way that a computer thinks.
<br><br>
We could have used pretty much any everyday process, but we'll go ahead with the laundry example. What's the first thing you do in the laundry cycle? You wear some clothes.
<br><br>
Let's write some code in Sonic Pi to represent wearing clothes. We write our code in a Buffer in the top-left window of Sonic Pi (these are called Workspaces in earlier versions). The top-right window gives us a log of what is happening when we run our code, and we'll use this later, and the bottom window, which we can hide and reveal by pressing the Help button, gives us tutorials, examples, and explanations about all the samples, synths, effects and language that we will use in Sonic Pi.<br><br>

<a href="http://sonic-pi.net"><img src="/assets/images/SonicPiScreen.png" alt="Sonic Pi Screen" style="width:100%;height:100%;"></a><br><br>

For now, just focus on the window where we write the code. We can write <code>sample</code> follwed by one of the included sound samples (you can also use your own samples, but we'll cover that another time). After you have typed <code>sample</code> you get a drop-down list to choose from, or you can go to the Help window and select Samples and choose from there. For this example lets pick ```:drum_tom_hi_hard```, so we need to write <code>sample :drum_tom_hi_hard</code>. Note how all sample names start with a colon - this is because they are keywords that represent a path to the sample, and it is important to include. We can then play the program, which is currently a single line, by hitting Run. You should hear the sound of a drum playing. Press Run again, and listen carefully to the sound we are using to represent the <i>wear</i> activity in our laundry cycle.  
<br><br>
Great, so we've worn our clothes - what's next? Lets wash them. We can use a different sound to represent this. This time, rather than chosing a sample, lets write <code>play</code>, which will play the synth that is defined. The default synth with Sonic Pi is beep, so given we haven't defined anything else we will use this. <code>play</code> on its own doesn't do anything, and will result in an error if we hit Run with this. We need to give it an argument so it knows what to play. The easiest thing to do is to provide it with a MIDI note, which will tell Sonic Pi the pitch to play. Lets choose 60 (this is equivalent to "middle C", and we could get the same result by writing ```play :c4``` instead), so on a new line write <code>play 60</code> and then hit Run. We now hear both the drum sample and the synth play at the same time. The computer is actually playing them sequentially - it reads from the top - but as soon as the first sample is triggered it moves to the next line, and this all happens quicker than our ears can hear. To get around this we can tell the program to pause by inserting a <code>sleep</code> of a defined amount of time. By default, Sonic Pi is set to run at 60 bpm (beats per minute), and so every beat comes 1 second apart. If we write <code>sleep 1</code> this will pause the program for 1 beat, which with the default bpm is 1 second. Let's try this and add <code>sleep 1</code> on a line between our code representing wear and wash. Now we hear the sounds as two separate events.
<br><br>
Now we have worn and washed our clothes, we need to dry them. Lets write <code>sample :elec_beep</code> to represet the drying. Lets assume we have a pause of 1 between each operation, so we will tell the program to <code>sleep 1</code> after each event. Our code should now look like this:
<pre><code class="ruby">sample :drum_tom_hi_hard
sleep 1
play 60
sleep 1
sample :elec_beep
sleep 1
</code></pre>
and as we keep hitting Run we will hear it play out our representation of wear, wash, dry. From now on, we will refer to this as our laundry cycle.
<br><br>
What if we want to repeat this laundry cycle without having to hit Run again? Well, we could type it out all again, but that would begin to get tedious the more we wanted to do, so instead let's wrap all our code in a block and ask it to repeat 5 times. To do this write <code>5.times do</code> at the top of the file, on a line before everything else, and <code>end</code> on the line after the last <code>sleep 1</code>. Now, everything between the ```do``` and the ```end``` will get repeated 5 times. Try it out!
<br><br> 
If we hit the Align button (on the top header bar) then all the code between the ```do``` and the ```end``` will get indented. This doesn't change what is happening with the code, but it starts to make it easier to read. Everything that is indented is a block that is being acted on by the instruction above, and in this case it is the <code>5.times do</code>
<pre><code class="ruby">5.times do
  sample :drum_tom_hi_hard
  sleep 1
  play 60
  sleep 1
  sample :elec_beep
  sleep 1
end
</code></pre>
Talking of being easy to read, what do you think about what we have written so far? Would someone else be able to understand what you are trying to do? How would they even know that you were coding a laundry cycle? And if you wanted to change how you did the wash, for example, you'd have to find the relevent code within the entire laundry cycle that you've written. So, let's refactor - change what we have written so that it works the same but is cleaner code. To do this, we're going to write functions to represent our wear, wash and dry operations. In Ruby, the programming language that we are using to write our code in Sonic Pi, we write ```def``` followed by a name of the function, then the body of the function, and lastly ```end``` to complete the function. First, lets write a function to represent wearing clothes. We can call the function anything, but it should be something that simply explains what the function is doing. Lets call this one 'wear'. At the top of the file insert a line and write ```def wear```. On the next lines we need to put our wear function - this was the drum sample followed by a sleep of 1. And then beneath that we write ```end``` to close off the function. We can now call this function from anywhere else in the file just by writing the function name, ```wear```, so lets remove the relevent code from our laundry cycle and simply replace it with ```wear```.
<pre><code class="ruby">def wear
  sample :drum_tom_hi_hard
  sleep 1
end

5.times do
  wear
  play 60
  sleep 1
  sample :elec_beep
  sleep 1
end 
</code></pre>
Hit Run again, and it should sound exactly the same as before. If we were writing professional code we would want to have a test suite that demonstrated that all of our functions performed as we intended, but we can't do this within Sonic Pi. Instead, we need to rely on our ears, and to listen as we read the code to check that it is performing as we expect. With practice you can start isolating individual sounds from within the music you have created, but if you struggle it is always worth pulling out just one function at a time and listening to what it is doing. Now that we have our ```wear``` function working, lets do the same for wash and dry. 
<pre><code class="ruby">def wear
  sample :drum_tom_hi_hard
  sleep 1
end

def wash
  play 60
  sleep 1
end

def dry
  sample :elec_beep
  sleep 1
end

5.times do
  wear
  wash
  dry
end
</code></pre>
We now have more lines of code than before to do the same work, but it is a lot easier to read what is happening. What's more, we can reuse the functions wherever we like without having to re-write the code, and if we wanted to change how ```wash```, for example, worked then we would know exactly where to go. Remember, we are using Sonic Pi so that we get audible feedback about how the program is working, but we could equally be writing code to control the appliances or to order cleaning services online, etc. So, within Sonic Pi, let's change our ```wash``` function. Instead of ```play 60```, we can change it to ```play 70```. Now we hear the ```wash``` function play at a higher pitch, but for our example we could think of this as washing at a higher temperature.
<br><br>
So, we can control how ```wash``` works, but we have to go into the function and change the code every time we want it to use a different value. What if we could just tell the program the temperature we wanted ```wash``` to run at whenever we called the function? As it happens, we can simply do this by passing an <i>argument</i> - the value that we want - into the function that we want to call. Within the laundry cycle we can provide the 'temperature' that we want, putting it within parentheses after our call to ```wash```, so to keep things sounding the same as they currently are we would write ```wash(70)```. Before this will work, we need to change our ```wash``` function so that it will accept the value as a <i>parameter</i>. We write the name for the parameter in parentheses after the function name, and we can name the parameter anything we like. Lets choose 'temperature', so that it makes sense within the program that we are writing. Within the ```wash``` function we can now access this variable, the value that we passed in, just by writing the parameter name: ```temperature```. The words <i>argument</i> and <i>parameter</i> are often used interchangeably, and can appear a little confusing at first. The argument is the data that is passed to the function when it is called; the parameter is a variable within the function. After making these changes, if we call ```wash(70)``` our program should sound exactly the same as before.
<pre><code class="ruby">def wear
  sample :drum_tom_hi_hard
  sleep 1
end

def wash(temperature)
  play temperature
  sleep 1
end

def dry
  sample :elec_beep
  sleep 1
end

5.times do
  wear
  wash(70)
  dry
end
</code></pre>
Let's do something similar for ```dry``` and ```wear```. The sample chosen to play in ```dry``` could be passed in when the function is called. We could use this in our example to represent selecting the type of dryer (tumble dryer or line dry, for example). And for ```wear``` we could pass in a value to represent smell, and use this to control the rate at which the sample is played. When samples are played at a higher rate they are effectively squashed, which means that they are played faster and are also heard at a higher pitch. The default rate in Sonic Pi is 1, so we need to use this value if we are to keep things sounding the same as before.
<pre><code class="ruby">def wear(smell)
  sample :drum_tom_hi_hard, rate: smell
  sleep 1
end

def wash(temperature)
  play temperature
  sleep 1
end

def dry(dryer)
  sample dryer
  sleep 1
end

5.times do
  wear(1)
  wash(70)
  dry(:elec_beep)
end
</code></pre>
When we run this code it should sound exactly the same, but now we have the power to change the data passed to the functions so as to create different laundry cycles. Go ahead and change what is passed to the function to anything that you want, and then listen to the effects.
<br><br>
Of course, laundry cycles aren't always as straight forward as just wear, wash, dry. What if you have a stain on your clothes? You may want to <code>treat_stain if dirty?</code>. Hopefully the intent of this code is clear, even if you are new to Ruby. It will call a function called ```treat_stain``` (the use of the underscore to separate words in the name of functions is idiomatic in Ruby) if the function ```dirty?``` returns true (again, the use of a ? at the end of a function name is idioatic in Ruby when the result of the function is a boolean - true or false). So, let's write these functions, starting with ```dirty?```. In this case, we are going to assume that you are pretty messy and half the time you stain your clothes. We'll ask the computer to randomly choose the value 1 or 2 from an array by writing ```[1,2].choose``` and then test if the result is equal to 1 by using the equality sign, which is ```==```. The function will return true if the result is equal to 1, and false if not, so on average it should return true half of the time. If the result is false then ```treat_stain``` won't be called, but if it is true then it will be called. Now let's write a function for ```treat_stain```. We do this in the same way as before, and this time let's choose to call ```sample :ambi_choir``` and to then ```sleep 2```. If we put this all together our code should now look like this:
<pre><code class="ruby">def wear(smell)
  sample :drum_tom_hi_hard, rate: smell
  sleep 1
end

def wash(temperature)
  play temperature
  sleep 1
end

def dry(dryer)
  sample dryer
  sleep 1
end

def dirty?
  1 == [1,2].choose
end

def treat_stain
  sample :ambi_choir
  sleep 2
end

5.times do
  wear(1)
  treat_stain if dirty?
  wash(70)
  dry(:elec_beep)
end
</code></pre>
Great, we've now used a conditional to control the path of our program! The program will do different things depending on whether a condition is met or not. In this case we just told the program to do something (call the function ```treat_stain```) if a conditional (```dirty?```) returned true, but we could also write it so that it did different things depending on the outcome. For this we would just need to write
<pre><code class="ruby">if {write the conditional in here} 
  {do something}
else
  {do something different}
end
</code></pre>
Now, let's add a second laundry cycle after the first, but this time we will pass different arguments. We can also change the cycles to just repeat 2 times so that the code takes less time to run, but feel free to make them repeat as many times as you would like. The functions that we defined above will all remain unchanged, so I won't write them here again, but they should stay at the top of your file.
<pre><code class="ruby">2.times do
  wear(1)
  treat_stain if dirty?
  wash(70)
  dry(:elec_beep)
end

2.times do
  wear(5)
  treat_stain if dirty?
  wash(90)
  dry(:elec_cymbal)
end
</code></pre>
So, with the same functions we can now run different laundry cycles just by passing in different arguments. And now if we want to change any of our functions we still only need to do it in one place, rather than in each of the laundry cycles. Let's try this with our ```wear``` function. We're going to make a recurrsive function, that is one that calls itself. We are going to keep wearing our clothes, and each time we do their smell level will increase. We can do this by adding the line <code>wear(smell + 1)</code> after ```sleep 1```. This will call the ```wear``` function again, but this time the argument that is passed in will be incremented by 1 from the previous value, so if ```smell``` has a value of 1, ```smell + 1``` will return the value of 2 (note that in Sonic Pi you can write ```inc smell``` to give the same result). Try and see what happens if we run this now.
<br><br>
We have to be careful with recurrsive functions otherwise they will continue forever. Hit Stop to force our program to stop. We can avoid the recurrsion from continuing forever by ensuring that it has an escape path. Let's do this by adding a conditional. If the ```smell``` is greater or equal to 10 (<code>if smell >= 10</code>) we play a snare drum sample (<code>sample :drum_snare_hard</code>), ```else``` we play our previous sample, sleep, and then recall the ```wear``` function with the ```smell``` value increased by 1, followed by ```end``` on a new line to close off the conditional. Don't forget to align your code so that it is easy to read. We now have our escape, and we will only contiue to wear our clothes until they reach a certain level of smell. Make these changes and then hit Run, and as it plays follow the code so that you can see what is happening. This is the beauty of using Sonic Pi - you can hear what your program is doing, and follow along with its logic. Your code should now look like this:
<pre><code class="ruby">def wear(smell)
  if smell >= 10
    sample :drum_snare_hard
  else
    sample :drum_tom_hi_hard, rate: smell
    sleep 1
    wear(smell + 1)
  end
end

def wash(temperature)
  play temperature
  sleep 1
end

def dry(dryer)
  sample dryer
  sleep 1
end

def dirty?
  1 == [1,2].choose
end

def treat_stain
  sample :ambi_choir
  sleep 2
end

2.times do
  wear(1)
  treat_stain if dirty?
  wash(70)
  dry(:elec_beep)
end

2.times do
  wear(5)
  treat_stain if dirty?
  wash(90)
  dry(:elec_cymbal)
end
</code></pre>
So, we've refactored our initial code and written functions with parameters. We're using conditionals and have also safely written a recurrsive function. What about running different things at the same time? This is called multi-threading, and can allow our programs to run more efficiently. It's also of vital importance when making music - to make a good dance track, for example, we'll want beats, bass, hooks and vocals all going on at the same time. Luckily, we can do this in Sonic Pi by creating an ```in_thread``` block. If we put this around the first of our laundry cycles by writing ```in_thread do``` at the start and ```end``` at the end, it will play at the same time as the second of the laundry cycles.
<pre><code class="ruby">in_thread do
  2.times do
    wear(1)
    treat_stain if dirty?
    wash(70)
    dry(:elec_beep)
  end
end

2.times do
  wear(5)
  treat_stain if dirty?
  wash(90)
  dry(:elec_cymbal)
end</code></pre> 
If you are struggling to hear what is happening, look at the log on the right hand side of Sonic Pi. It tells you what is playing at each point in time, and you should be able to see that both of the laundry cycles are happening concurrently.
<br><br>

<b>Next steps to becoming a computer programmer</b><br><br>
We've now taken an everyday problem and structured it in a way that a computer program can understand. If you followed along and it all made some sense, then you certainly have the potential to make it as a software developer! Try expanding the exercise further - this is what happens in software developemnt: further requirements will be added and you will have to respond to them. What about sorting clothes by colour or fabric? What detergent choices are there? Do you want to fold your clothes or hang them in a closet? What about prioritisation over what is washed? And after that why not break down another day-to-day activity and code it in Sonic Pi? It could be anything you like. Maybe making a cup of coffee or baking a cake; driving to work; doing school homework. It could be anything you like. I'd love to hear what you come up with.
<br><br>
If you found breaking down a problem into a program that the computer understands fun, then why not learn more about how to code? There are an enormous amount of resources online, as well as local groups and training camps. You've been writing Ruby code in Sonic Pi, so a good place to start might be to continue to learn this language. Amongst other things, there is the free online book <a href="http://learnrubythehardway.org/book/">Learn Ruby The Hard Way</a>, that will walk you through setting up your Ruby environment and taking your next steps to becoming a programmer.
<br><br>
<b>Extending the laundry example to make music</b><br><br>
The inspiration for creating a simple interface to live-code music came during the times that <a href="https://twitter.com/samaaron">Sam Aaron</a> and myself were up on stage as <a href="http://meta-ex.com">Meta-eX</a> in front of audiences of non-programmers. We had a constant dilemma of how to convey what we were actually doing. We projected our code but to most people it was alien, and rather than understanding that the code was our instrument, they just commented on how interesting and beautiful our visuals were. They couldn't see that we were building and manipulating synths, writing the riffs, and composing the beats right in the instance in which we played. We might as well have just pressed play on a pre-recorded track. So, how could we demonstrate the virtuosity of our performance? The answer: get more people doing it, so that code becomes as understood an instrument as a guitar or piano. As you have seen, with <a href="http://sonic-pi.net">SonicPi</a>, there is a low barrier of entry into making music with code, so let's go ahead with out laundry example and see how we can extend it into a live-coding performance.  
<br>
If you want the laundry cycles to continue forever, you can simple replace the ```2.times do``` with ```loop do```. This is great, unless you want to change anything, and no one wants to listen to the same thing happening over and over forever. You can Stop the program, make a change, and then hit Run again, but if you are making music live you don't want to stop everything whenever you want to make a change. Let's see what happens if you do change the ```temperature``` of the first laundry cycle to ```80```, but you don't stop the program first. When you make the change nothing happens until you hit Run again, and when you do a new run will start, but the previous one will still be continuing. Now you have both versions running, and unless you were extremely lucky they won't be in time. This may not be an issue for doing your laundry, but it is fundamental for making music that you can control.
<br><br>
Luckily, Sonic Pi can handle the multi-threading through the use of ```live_loop```. This will handle the threading behind the scenes, and if the thread already exists it will redefine the functions but not start a new thread, and if it doesn't exist it will create a new thread. Let's go ahead and make the changes to our code. We'll keep all our functions the same as before, and just change the laundry cycles. Because ```live_loop``` takes care of the threading for us, we can remove the ```in_thread``` block around the first laundry cycle. We then need to change ```loop``` to ```live_loop``` and give each one a unique name. Let's call them ```:laundry_1``` and ```:laundry_2```. Like with samples, again note that our names in ```live_loop``` must start with a colon. The last part of our program should now look like this:

<pre><code class="ruby">live_loop :laundry_1 do
  wear(1)
  treat_stain if dirty?
  wash(80)
  dry(:elec_beep)
end


live_loop :laundry_2 do
  wear(5)
  treat_stain if dirty?
  wash(90)
  dry(:elec_cymbal)
end</code></pre> 

We are now free to hot-swap the code, and the changes will be made without our music stopping. Lets make some changes to ```:laundry_1``` by changing the ```wash``` parameter to ```60``` and the ```dry``` parameter to ```:guit_harmonics```. Hit Run again, and you'll hear that the music continues, but that the changes have occured (the changes come into effect the next time that the live_loop function is called, so it may take a little while before you hear the change that you made). If you look at the log on the right hand side as you hit Run you will see something like this

<pre><code>=> Starting run 20

=> Redefining fn :live_loop_laundry_1

=> Redefining fn :live_loop_laundry_2

=> Thread :live_loop_laundry_1 exists: skipping creation

=> Thread :live_loop_laundry_2 exists: skipping creation

=> Completed run 20</code></pre>

although your run number may well be different. So, Sonic Pi has redefined the functions, seen that the threads already exist, and so it doesn't create new threads but continues with the original run, now with the redefined live_loops. Equally well, we can change the other functions that we use on-the-fly as well. Let's go into the function ```wear``` and change the value of ```sleep``` from ```1``` to ```0.1```. Again, when we hit Run the music continues, but the next time that ```wear``` is called within the program it will run with the modification to the function. Awesome!
<br><br>
What if we want to add more functions? Easy - we just do it in the same way, all whilst the program is running. Let's define a function called ```iron```, and because I don't like ironing we'll write ```use_synth :growl``` to define the synth we'll use, and then ```play 60```. We can now call ```iron``` anywhere in our code, and we'll place it after ```dry``` in ```:laundry_1```.

<pre><code class="ruby">def wear(smell)
  if smell >= 10
    sample :drum_snare_hard
  else
    sample :drum_tom_hi_hard, rate: smell
    sleep 0.1
    wear(smell + 1)
  end
end

def wash(temperature)
  play temperature
  sleep 1
end

def dry(dryer)
  sample dryer
  sleep 1
end

def iron
  use_synth :growl
  play 60
  sleep 1
end

def dirty?
  1 == [1,2].choose
end

def treat_stain
  sample :ambi_choir
  sleep 2
end

live_loop :laundry_1 do
  wear(1)
  treat_stain if dirty?
  wash(60)
  dry(:guit_harmonics)
  iron
end

live_loop :laundry_2 do
  wear(5)
  treat_stain if dirty?
  wash(90)
  dry(:elec_cymbal)
end</code></pre>

Things get slightly more complicated when we want to add in more live_loops. What if we want time to be indicated by the sounding of a bass_drum at the start of each time period and a snare_drum in the middle? We can do this by adding a live_loop called :time, and adding our samples in.

<pre><code class="ruby">live_loop :time do
  sample :drum_bass_hard
  sleep 0.5
  sample :drum_snare_hard
  sleep 0.5
end </code></pre>

This time a new thread has been created that didn't already exist, so a new run is started. The problem is the same as before: our new run is not necessarily in time with the existing run. We can get around this, though, by syncing our threads.
<br><br>
Ideally, we will sync to an existing thread. What this means is that the new thread will block until the thread that it is synced to starts another cycle. Our problem here is that both ```:laundry_1``` and ```:laundry_2``` are longer than our new thread ```:time```, and they are also of variable length, due to the ```treat_stain``` conditional. This means that ```:time``` will play once, and then will have to wait until one of the laundry cycles has finished before it plays again, so it will not keep the metronome beat that we wanted from it. I find it good practice to define at the start of a performance a live_loop that will stay constant throughout, and then you can sync everything to this. I will often define a ```:tick``` that just sleeps for 1, and then I can sync everything to this.
<pre><code class="ruby">live_loop :tick do
  sleep 1
end </code></pre> 
If, like now, we haven't done this at the start, we can add it in retrospectively as we play, and just suffer a momentary blip in our performance as all the running threads get in sync. Lets do this now by adding the ```live_loop :tick```, and then inserting the line ```sync :tick``` at the start of each of the other live_loops:

<pre><code class="ruby">def wear(smell)
  if smell >= 10
    sample :drum_snare_hard
  else
    sample :drum_tom_hi_hard, rate: smell
    sleep 0.1
    wear(smell + 1)
  end
end

def wash(temperature)
  play temperature
  sleep 1
end

def dry(dryer)
  sample dryer
  sleep 1
end

def iron
  use_synth :growl
  play 60
  sleep 1
end

def dirty?
  1 == [1,2].choose
end

def treat_stain
  sample :ambi_choir
  sleep 2
end

live_loop :laundry_1 do
  sync :tick
  wear(1)
  treat_stain if dirty?
  wash(60)
  dry(:guit_harmonics)
  iron
end

live_loop :laundry_2 do
  sync :tick
  wear(5)
  treat_stain if dirty?
  wash(90)
  dry(:elec_cymbal)
end

live_loop :time do
  sync :tick
  sample :drum_bass_hard
  sleep 0.5
  sample :drum_snare_hard
  sleep 0.5
end

live_loop :tick do
  sleep 1
end</code></pre>

We now have three runs all going (the first run had the ```:laundry_1``` and ```:laundry_2``` threads; the second has the ```:time``` thread; and the third has the ```:tick``` thread), but they are now all synced, and so running in time. We can now freely change all our functions and they will stay perfectly in time, as long as we keep ```:tick``` constant. If we add more threads, then we just need to sync them to ```:tick``` as well, and they will then start in time. As we get more advanced in our musical compositions we may want to sync to different threads, but to start with this appraoch is a safe bet. 
<br><br>
At the moment our example isn't very musically exciting, but you now have the basic skills you need to go out and rock! Explore the tutorials and examples that Sonic Pi provides, and take a look at what everyone else is doing. If you know Ruby already then use your skills to create powerful functions, and if coding is all new to you then why not go out and learn more?
<br><br>

<b>Conclusions</b><br><br>
<a href="http://sonic-pi.net">SonicPi</a> is a simple to use and yet powerful application that can start you on the path to live-coding music. With a bit of skill and a lot of practice you could be rocking out a venue, big or small, using your Raspberry Pi or laptop. But Sonic Pi, with it's immediate audible feedback and use of the Ruby programming language, is also a great place to discover if programming is something that you're interested in, and it can give you the basic skills to start you writing professional, structured code. 