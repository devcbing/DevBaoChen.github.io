---
layout: page
title: 生活
description: 人越学越觉得自己无知
keywords: 生活, 
comments: false
menu: 生活
permalink: /wiki/
---

> 生活的诗与酒！

<ul class="listing">
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" %}
<li class="listing-item"><a href="{{ site.url }}{{ wiki.url }}">{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
</ul>
