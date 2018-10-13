---
layout: post-sidebar
date: 2018-10-13
title: "DRF 05 - Format인자와 다양한 Renderer"
categories: django
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 5
feature_image: feature-wolf
show_related_posts: true
square_related: recommend-fire
---
rest_framework.response.Response에서는 `api`, `json` 두가지 타입의 응답 가능함

* API: 웹브라우져 요청시
* JSON: API요청시

```python

# HTML 응답요청
http :8000/ep04/ Accept:text/html
http :8000/ep04/?format=api
http :8000/ep04/.api

# JSON 응답요청
http :8000/ep04/ Accept:application/json
http :8000/ep04/?format=json
http :8000/ep04.json
```
