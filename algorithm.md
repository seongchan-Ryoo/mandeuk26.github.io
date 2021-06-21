---
layout: page
title: Algorithm
---

수정 필요

<div class="message">
  {% for tag in tags %}
  <a href="#{{ tag | slugify }}">{{ tag }}</a>&nbsp;&nbsp;
  {% endfor %}
</div>

{% for tag in tags %}
<p id="{{ tag | slugify }}"><b>{{ tag }}</b></p>
<ul>
  {% for post in site.posts %}
  {% if post.tags contains tag %}
  <li>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
  </li>
  {% endif %}
  {% endfor %}
</ul>
{% endfor %}
