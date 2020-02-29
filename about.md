---
layout: default
title: About
---

# {{ page.title }}

Welcome to Tender Is The Byte, a blog about programming, research, ecology, and more!

## Authors

{% for author in site.data.authors.all %}
{% include bio.html author=author.id %}
{% endfor %}