---
layout: page
title: Dr. Jonathan Graham
tagline: Code and Music; Science and Art
include_social: true
---
{% include JB/setup %}

<section id="research" class="centered">
  <div class="col-lg-10 col-lg-offset-1 ">
  <p>
    I create, and enable others to create more.
    <br><br>
    My interests are diverse, and cover a melting pot of code, music, science and art, all blended together with business, social enterprise, and teaching. Collaborating with others, I have helped create new tech career opportunities in rural Appalachia, developed software to perform music with code, brought new drugs to the market, and created art with chemistry.<br><br>
    Find out more below, and in my <a href="../blog.html">blogs</a> and <a href="../talks.html">talks</a>.</p>
  </div>
</section>
<section>
<div class="col-lg-10 col-lg-offset-1 row centered">
  <div class="col-md-6">
  <p class="section-title"><span>Code</span></p>
      <p>As co-founder of <a href="http://minedminds.org/">Mined Minds</a>, I am helping to create tech hubs in rural communities by training unemployed and underemployed workers how to write code, and employing our graduates as software consultants. By teaching the fundamentals of full stack software development, with an emphasis on understanding and quality within an agile development environment, our graduates add real value from day one.</p>
      <p>As well as teaching the fundamentals, I also provide training across the globe in a range of subjects, from Clojure and Reactive Systems, to making music through code, and providing pragmatic approaches to agile development.</p>
      <p>I have delivered projects for clients ranging from multi-nationals to local businesses and start-ups, using many programming languages, frameworks, and paradigms.</p><br><br>
  </div>
  <div class="col-md-6">
  <p class="section-title"><span>Science</span></p>
      <p>I am passionate about designing and developing robust and scalable systems, which are well understood and can be modified to react to changes in production environment or customer requirements. To many people this passion sounds very similar to software development, but mine actually stems from the process of manufacturing drugs for GlaxoSmithKline.<br><br>Following the completion of my PhD in Organic Chemistry, I spent over a decade in drug development. During that time I led multiple projects from early discovery through to manufacture and commercialisation, authored several papers and presented at many international conferences. I now use my science background to help teach and present about software development concepts.</p>
  </div>
  </div>
  </section>
  <section>
  <div class="col-lg-10 col-lg-offset-1 row centered">
    <div class="col-md-6">
    <p class="section-title"><span>Music</span></p>
        <p>Having a club-full of revellers dancing to the music you are creating live through the writing of code is a truly amazing experience, and one that I would recommend to everyone! <br><br>As <a href="http://meta-ex.com">Meta-eX</a>, we harnessed the power of SuperCollider through the Overtone platform to generate music with Clojure code, be it gently evolving soundscapes or pulsating beats.<br><br>To bring the power of code to non-tech audiences, what we learned from the stage was transformed into <a href="http://sonic-pi.net">SonicPi</a>, which I have used to perform live solo gigs, run workshops, and teach programming concepts to non-programmers of all ages.</p><br><br>
    </div>
  <div class="col-md-6">
  <p class="section-title"><span>Art</span></p>
      <p>What is life without art? Art is everything that isn't purely functional, and it's an art to design and develop something that functions beautifully.<br><br> I've been lucky enough to be involved in <a href="http://www.breathproject.net">The Breath Project</a> as a chemistry consultanat, and we've filmed some incredible footage of chemical reactions and explosions. Part of my goal on the project is to explain the science behind the art, and I have used hands-on workshops in schools to inspire children from 5 years old upwards with the art of science.</p>
  </div>
  </div>
</section>
<section>
  <div class="col-lg-10 col-lg-offset-1 ">
  <ul class="research">
    Recent blogs:<br> 
    {% for post in site.posts limit: 10%}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}: {{ post.tagline }}</a></li>
    {% endfor %}
    <br><br>See the full list of blogs <a href="../blog.html">here</a>
  </ul>
  </div>
</section>