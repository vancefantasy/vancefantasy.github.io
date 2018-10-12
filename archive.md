---
layout: page
title: Archive
---

{% for post in site.posts %}{{ post.date | date: "%Y-%m-%d" }} &raquo; [ {{ post.title }} ]({{ post.url }})  
{% endfor %}
