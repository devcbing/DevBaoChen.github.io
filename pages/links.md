---
layout: page
title: Links
description: 没有收藏的博客是孤独的
keywords: 收藏
comments: true
menu: 收藏
permalink: /links/
---

> 我的收藏

{% for link in site.data.links %}
* [{{ link.name }}]({{ link.url }})
{% endfor %}
