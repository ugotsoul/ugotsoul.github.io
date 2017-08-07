---
layout: default
title: Home
---

<h3>I am so smart, I am so smart, S-M-R-T! ... D'oh!</h3>
{% for post in site.posts %}
  {{ post.date | date: "%b, %d, %Y" }}
  <a href="{{ post.url }}">
    {{ post.title }}
  </a>
{% endfor %}
