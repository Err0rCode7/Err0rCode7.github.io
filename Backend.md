---
bg: "BACKEND.jpg"
layout: page
permalink: /Backend/
category: Backend
title: "Backend"
crawlertitle: "Backend"
active: Backend
---
{% for post in site.posts %}
  {% if post.category == "Backend" %}
  <article class="index-page">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </article>
  {% endif %}
{% endfor %}
