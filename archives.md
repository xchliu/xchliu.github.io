---
layout: default
title: 归档
permalink: /archives/
---

<style>
.archive-year { margin: 1.5rem 0; }
.archive-year h2 { font-size: 1.3rem; border-bottom: 2px solid #eee; padding-bottom: 0.5rem; color: #555; }
.archive-year ul { list-style: none; padding: 0; margin: 0.8rem 0; }
.archive-year li { padding: 0.4rem 0; border-bottom: 1px solid #f0f0f0; }
.archive-year li:last-child { border-bottom: none; }
.archive-year .post-date { color: #999; font-size: 0.85rem; margin-right: 1rem; min-width: 4.5rem; display: inline-block; }
.archive-year .post-title { color: #333; text-decoration: none; }
.archive-year .post-title:hover { color: #007bff; }
.archive-year .post-cats { font-size: 0.8rem; color: #888; margin-left: 0.5rem; }
</style>

<p style="color:#888;">共 {{ site.posts | size }} 篇文章</p>

{% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year_group in posts_by_year %}
<div class="archive-year">
<h2>{{ year_group.name }}</h2>
<ul>
{% for post in year_group.items %}
<li>
    <span class="post-date">{{ post.date | date: "%m-%d" }}</span>
    <a class="post-title" href="{{ post.url }}">{{ post.title }}</a>
    {% if post.categories %}
    <span class="post-cats">· {{ post.categories | join: ", " }}</span>
    {% endif %}
</li>
{% endfor %}
</ul>
</div>
{% endfor %}
