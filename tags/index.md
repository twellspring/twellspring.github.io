---
layout: default
title: tags
---


<ul class="tags">
{% assign sortedtags = site.tags | sort %}
{% for tag in sortedtags %}
  {% assign t = tag | first %}
  {% assign posts = tag | last %}
  <li><a href="/tags/{{t}}">{{ t }} ({{ posts | size }})</a> </li>
{% endfor %}
</ul>
