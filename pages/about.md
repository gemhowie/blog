---
layout: page
title: About
description: 无意续余年
keywords: gemhowie, 豪伊
comments: true
menu: 关于
permalink: /about/
---

毕竟，人，总要仰望些什么。不然，此生就太无趣了

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
