---
layout: page
title: 收藏的链接
description: 没有链接的博客是孤独的
keywords: 链接
comments: true
menu: 链接
permalink: /links/
---

> God made relatives. Thank God we can choose our friends.

{% for link in site.data.links %}
* [{{ link.name }}]({{ link.url }})
{% endfor %}
