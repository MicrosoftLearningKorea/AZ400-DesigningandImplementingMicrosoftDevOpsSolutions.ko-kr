---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
ms.openlocfilehash: b620a0c228333f5d50f55bd3e2238ccdd6dd8472
ms.sourcegitcommit: d8f0a6eb53689d119ce8cf3a4416ad2c80cf919d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/27/2022
ms.locfileid: "137894863"
---
# <a name="content-directory"></a>콘텐츠 디렉터리

아래에는 각 랩 연습의 하이퍼링크 목록이 나와 있습니다.

## <a name="labs"></a>랩

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 모듈 | 랩 |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}


