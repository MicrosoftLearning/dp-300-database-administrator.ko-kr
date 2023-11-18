---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# 데이터베이스 관리 연습

이 연습에서는 Microsoft 과정 [DP-300: Microsoft Azure SQL 솔루션 관리](https://docs.microsoft.com/training/courses/dp-300t00)를 지원합니다.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 모듈 | 연습 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

