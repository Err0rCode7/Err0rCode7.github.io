---
bg: "Projects.jpg"
layout: page
permalink: /Projects/
category: Projects
title: "Projects"
crawlertitle: "Projects from Err0rCode7"
active: Projects
---
{% for post in site.posts %}
  {% if post.category == "Projects" %}
  <article class="index-page">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    {{ post.excerpt }}
  </article>
  {% endif %}
{% endfor %}
