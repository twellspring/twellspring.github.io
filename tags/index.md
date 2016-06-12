---
layout: default
title: tags
---

{% capture tags %}
  {% for tag in site.tags %}
    {{ tag[0] }}
  {% endfor %}
{% endcapture %}
{% assign sortedtags = tags | upcase | split:' ' | sort %}

{% for tag in sortedtags %}
  <li><a href="/tags/{{ tag }}">{{ tag }} <span>({{ tag.size }})</span></a></li>
{% endfor %}
