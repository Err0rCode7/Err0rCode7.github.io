---
bg: "Review.jpg"
layout: page
permalink: /Reviews/
category: Reviews
title: "Reviews"
crawlertitle: "Reviews from Err0rCode7"
active: Reviews
---
{% for post in site.posts %}
  {% if post.category == "Diary" %}
  <article class="index-page">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </article>
  {% endif %}
{% endfor %}
