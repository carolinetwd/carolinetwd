---
layout: page
title: Blog
permalink: /blog/
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>— {{ post.date | date: "%b %d, %Y" }}</small>
      {% if post.excerpt %}<div>{{ post.excerpt }}</div>{% endif %}
    </li>
  {% endfor %}
</ul>
