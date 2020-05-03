---
layout: page
title: Featured
permalink: /featured/
---

Here you can find all featured posts (the ones I personally value most).

<ul>
{% for post in site.posts %}
  {% if post.tags contains 'featured' %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
  {% endif %}
{% endfor %}
</ul>
