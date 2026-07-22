---
layout: default
title: 标签
permalink: /tags/
---

<style>
.tag-cloud { margin: 1.5rem 0; }
.tag-group { margin: 1.5rem 0; }
.tag-group h3 { font-size: 1.1rem; border-bottom: 1px solid #eee; padding-bottom: 0.3rem; color: #555; }
.tag-group ul { list-style: none; padding: 0; margin: 0.5rem 0; }
.tag-group li { padding: 0.3rem 0; }
.tag-group .post-date { color: #999; font-size: 0.85rem; margin-right: 1rem; min-width: 4.5rem; display: inline-block; }
.tag-group .post-title { color: #333; text-decoration: none; }
.tag-group .post-title:hover { color: #007bff; }

.tag-badge { display: inline-block; background: #e8f4fd; color: #007bff; padding: 4px 12px; border-radius: 20px; font-size: 0.9rem; margin: 4px; text-decoration: none; }
.tag-badge:hover { background: #007bff; color: #fff; text-decoration: none; }
</style>

{% comment %}Collect all categories as tags{% endcomment %}
{% assign all_tags = "" | split: "" %}
{% for post in site.posts %}
  {% if post.categories %}
    {% for cat in post.categories %}
      {% assign all_tags = all_tags | push: cat %}
    {% endfor %}
  {% endif %}
{% endfor %}

{% assign unique_tags = all_tags | uniq | sort %}

<div class="tag-cloud" style="margin-bottom:2rem;">
{% for tag in unique_tags %}
{% assign count = all_tags | where_exp: "t", "t == tag" | size %}
<a class="tag-badge" href="#{{ tag | slugify }}">{{ tag }} ({{ count }})</a>
{% endfor %}
</div>

{% for tag in unique_tags %}
<div class="tag-group">
<h3 id="{{ tag | slugify }}">{{ tag }}</h3>
<ul>
{% for post in site.posts %}
  {% if post.categories contains tag %}
  <li>
    <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
    <a class="post-title" href="{{ post.url }}">{{ post.title }}</a>
  </li>
  {% endif %}
{% endfor %}
</ul>
</div>
{% endfor %}
