---
layout: home   # with minima, this shows a post list below your content
title: Home
---

{% capture readme %}{% include_relative README.md %}{% endcapture %}
{{ readme | markdownify }}
