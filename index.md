---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# 콘텐츠 디렉터리

각 랩 연습에 대한 하이퍼링크는 아래에 나열되어 있습니다.

## 랩

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 모듈 | 랩 |
| --- | --- |
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

## 데모

{% assign demos = site.pages | where_exp:"page", "page.url contains '/Instructions/Demos'" %}
| 모듈 | 데모 |
| --- | --- |
{% for activity in demos  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
