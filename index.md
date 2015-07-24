---
layout: page
title: Jonathan Graham
tagline: Code and Music; Science and Art
include_social: true
---
{% include JB/setup %}

<section id="research" class="centered">
  <p class="section-title"><span>Blogs</span></p>
  <p>Life is short and opportunities are many.<br><br> My interests cover a melting pot of code, music, science and art, and these blogs will cover all of them. Sometimes the topics will be discrete, but often, as in life, they will bring together a mixture of them all. Hopefully there will be something that interests you along the way!</p>
</section>
<section>
  <ul class="research">
    Recent blogs:<br> 
    {% for post in site.posts limit: 5%}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
    <br><br>See the full list of blogs <a href="../blog.html">here</a>
  </ul>
</section>
<section id="research" class="centered">
  <p class="section-title"><span>Background</span></p>
  <p>A little about my current state:</p>
  <section>
    <article class="research-item">
      <h2>Code</h2>
      <p>It was through music that I was first exposed to code, and where I saw just how powerful and flexible a tool programming was. The more I spoke with top-notch software developers, the more I realised the limitless opportunities that there were to do new and awesome things. And so I have taken the plunge and switched my career to one of code. A career that has many overlaps with science, music and art, and a career that I am excitedly getting fully immersed in. <br><br> Currently, I am expanding my programming abilities from procedural hobby code to one of structured professional quality through an apprenticeship position at <a href="http://8thlight.com">8th Light</a>. I am also co-founder of <a href="http://minedminds.github.io/">Mined Minds</a>, bringing free computer programming training to Greene County, PA. </p>
      <div class="more">
        <a href="code.html" class="button">programming</a>
      </div>
    </article>
    <article class="research-item">
      <h2>Music</h2>
      <p>Having a club-full of revellers dancing to the music you are creating live through the writing of code is a truly amazing experience, and one that I would recommend to everyone! <br><br>As <a href="http://meta-ex.com">Meta-eX</a>, we harnessed the power of SuperCollider through the Overtone platform to generate music, be it gently evolving soundscapes or pulsating beats, by writing Clojure code in the Emacs Live text editor. <br><br>I have also played live solo gigs using <a href="http://sonic-pi.net">SonicPi</a>, and have used this tool to teach programming concepts to non-programmers of all ages.</p>
      <div class="more">
        <a href="music.html/" class="button">music</a>
      </div>
    </article>
  </section>
</section>
<section>
  <section>
    <article class="research-item">
      <h2>Science</h2>
      <p>I am passionate about designing and developing robust and scalable systems, which are well understood and can be modified to react to changes in production environment or customer requirements. To many people this passion sounds very similar to software development, but mine actually stems from the process of manufacturing drugs for GlaxoSmithKline.<br><br>Following the completion of my PhD in Organic Chemistry, I spent over a decade in drug development. During that time I led multiple projects from early discovery through to manufacture and commercialisation, authored several papers and presented at many international conferences.   </p>
      <div class="more">
        <a href="science.html" class="button">science</a>
      </div>
    </article>
    <article class="research-item">
      <h2>Art</h2>
      <p>What is life without art? Art is everything that isn't purely functional, and it's an art to design and develop something that functions beautifully.<br><br> I've been lucky enough to be involved in <a href="http://www.breathproject.net">The Breath Project</a> as a chemistry consultanat, and we've filmed some incredible footage of chemical reactions and explosions. Part of my goal on the project is to explain the science behind the art, and I have used hands-on workshops in schools to inspire children from 5 years old upwards with the art of science.</p>
      <div class="more">
        <a href="art.html" class="button">art</a>
      </div>
    </article>
  </section>
</section>