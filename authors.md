---
layout: post
title: Authors
permalink: /authors/
sitemap: true
---

{% for author in site.authors %}
[{{ author.name }}]({{ site.url }}/authors/{{ author.name }})
{% endfor %}