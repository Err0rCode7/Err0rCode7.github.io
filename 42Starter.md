---
bg: "42main.jpg"
layout: page
permalink: /42Starter/
category: 42Starter
title: "42Starter"
crawlertitle: "42Starter from Err0rCode7"
active: 42Starter
---
{% for post in site.posts %}
  {% if post.category == "42Starter" %}
  <article class="index-page">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </article>
  {% endif %}
{% endfor %}
