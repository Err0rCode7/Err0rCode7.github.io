---
bg: "Review.jpg"
layout: page
permalink: /Java/
category: Java
title: "Java"
crawlertitle: "Java"
active: Java
---
{% for post in site.posts %}
  {% if post.category == "Java" %}
  <article class="index-page">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </article>
  {% endif %}
{% endfor %}
