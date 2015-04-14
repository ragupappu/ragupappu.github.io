---
layout: page
title: Archive
---

{% for post in site.posts %}
  {{ post.date | date_to_string }} &nbsp;&nbsp;&raquo;&nbsp;&nbsp; [ {{ post.title }} ]({{ post.url }})
{% endfor %}

{% include copyright.html %}

