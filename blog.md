---
layout: page
title: Blogs
---
{% include JB/setup %}

<div id="home">
  <ul class="posts">
    {% for post in site.posts %}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}<!-- : {{ post.tagline }} --></a></li>
    {% endfor %}
  </ul>
</div>
