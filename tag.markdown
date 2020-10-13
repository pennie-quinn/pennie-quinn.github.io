---
layout: page
title: Tags
permalink: /tag/
---

{% assign sortedtags = site.tags | sort %}
{% for tag in sortedtags %}
<a href="/tag/{{ tag[0] }}">{{ tag[0] }} ({{ tag[1].size }})</a>
{% endfor %}
