---
layout: post-sidebar
date: 2018-10-13
title: "DRF 06 - Filtering"
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
장고 APIView의 `Request`와 `URL Capture`에 대해 알아본다

# API Request

* 목록조회 APIView에서는 queryset을 필터링할 필요가 있음
* 검색어, 유저에 따른 필터링의 조건이 필요함
* APIView 는 View 를 상속받은 CBV
    * self.request 로 HTTPRequest를 참조할 수 있음
    * self.request.user: 로그인된 유저정보
    * self.request.GET : GET요청인자
    * self.request.POST: POST 요청인자
    * self.request.query_params: GET인자와 동일한 값. 가독성이 높음
        ```python
            self.request.query_params.get('user_id')
        ```

# URL Capture

* `self.kwargs['username']`으로 URL capture 값을 획득할 수 있음
    ```python
    urlpatterns =[
        url(r'^(?P<username>\w+)$',views.SomeView.as_view()),
    ]
    ```