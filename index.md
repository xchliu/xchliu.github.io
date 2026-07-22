---
layout: default
title: 首页
---

<div style="max-width:720px;margin:0 auto;padding:30px;">
<h1>{{ site.title }}</h1>
<p style="color:#888;">{{ site.description }}</p>
<hr style="border:none;border-top:1px solid #eee;margin:20px 0;">

{% for post in site.posts %}
<div style="margin-bottom:20px;padding:15px;background:#fff;border-radius:6px;box-shadow:0 1px 3px rgba(0,0,0,.06);">
  <h2 style="margin:0 0 5px;font-size:1.2rem;">
    <a href="{{ post.url }}" style="color:#333;text-decoration:none;">{{ post.title }}</a>
  </h2>
  <div style="color:#888;font-size:0.85rem;">
    {{ post.date | date: "%Y-%m-%d" }}
    {% if post.categories %}
    · {{ post.categories | join: ", " }}
    {% endif %}
  </div>
  {% if post.excerpt %}
  <p style="color:#555;font-size:0.95rem;margin:8px 0 0;">{{ post.excerpt | strip_html | truncate: 120 }}</p>
  {% endif %}
</div>
{% endfor %}
</div>