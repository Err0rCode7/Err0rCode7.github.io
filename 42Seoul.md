---
bg: "42seoul.jpg"
layout: page
permalink: /42Seoul/
category: 42Seoul
title: "42Seoul"
crawlertitle: "42Seoul from Err0rCode7"
active: 42Seoul
---
{% for post in site.posts %}
  {% if post.category == "42Seoul" %}
  <article class="index-page">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </article>
  {% endif %}
{% endfor %}
