---
layout: page
title: 关于
menu: About
---
{% assign current_year = site.time | date: '%Y' %}

SunnyDay
===
男 80后

## 概况

- 邮箱：yangbaojin2015#gmail.com
- 主页：[https://github.com/yangbaojin1988](https://github.com/yangbaojin1988)

计算机专业毕业，{{ current_year | minus: 2013 }} 年在职工作经验，{{ current_year | minus: 2013 }} 年 web 开发经验。


## keywords
<div class="btn-inline">
{% for keyword in site.skill_keywords %} <button class="btn btn-outline" type="button">{{ keyword }}</button> {% endfor %}
</div>

### 简言
无论你觉得自己多么的不幸，永远有人比你更加不幸；无论你觉得多么的了不起，也永远有人比你更强…