---
layout: page
title: Categories
permalink: /categories/
---

Here you can find all posts grouped by categories.

{% assign sorted_categories = site.categories | sort %}
{% for category in sorted_categories %}
  {% assign t = category | first %}
  {% assign posts = category | last %}

#### {{ t | downcase }}
<ul>
{% for post in posts %}
  {% if post.category contains t %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
  {% endif %}
{% endfor %}
</ul>
{% endfor %}
