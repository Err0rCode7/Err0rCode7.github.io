---
bg: "owl.jpg"
layout: page
permalink: /Diary/
category: Diary
title: "Diary"
crawlertitle: "Diary from Err0rCode7"
summary: "Diary"
active: Diary
---
{% for post in site.posts limit: 5 %}
  {% if post.category == "Diary" %}
  <article class="index-page">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </article>
  {% endif %}
{% endfor %}
