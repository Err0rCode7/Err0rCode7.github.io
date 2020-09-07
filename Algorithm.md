---
bg: "Algorithm.jpg"
layout: page
permalink: /Algorithm/
category: Algorithm
title: "Algorithm"
crawlertitle: "Algorithm from Err0rCode7"
active: Algorithm
---
{% for post in site.posts %}
  {% if post.category == "Algorithm" %}
  <article class="index-page">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </article>
  {% endif %}
{% endfor %}
