---
layout: post-sidebar
date: 2018-10-14
title: "DRF 12 - DRF Authentication"
categories: django
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 10
feature_image: feature-wolf
show_related_posts: true
square_related: recommend-fire
---
**장고의 인증서비스에 대해 알아보고 기본 예제를 활용해본다**

## DRF 제공 인증 서비스
* `SessionAuthentication`
    * 외부 서비스에서 사용불가
* `BasicAuthentication`
    * 보안상 매우 위험함
* `TokenAuthentication`
    * 초기에 username/password 토큰발급받아 매API호출시 사용
    * token을 저장할 storage 필요

## TokenAuthentication tutorial
**1. settings 추가 후 migrate하여 모델생성**
* Token 어트리뷰트: key, user
* 모델생성했다고 토큰이 자동생성되지 않음
    ```python
    INSTALLED_APPS = (
    'rest_framework.authtoken',
    )
    ```
**2. Token 생성**
* 토큰 생성법
    1. ObtainAUthToken
    2. Signal
    3. Management를 통한 생성
        * 에러날 경우 장고 update
        * 생성: `python manage.py drf_create_token <username>`
        * 강제 재생성: `python manage.py drf_create_token -r <username>`
    4. API Endpoint로 노출
        * `url(r'^api-token-auth/$', obtain_auth_token)`
        * 여기서는 API Endpoint로 token 생성
        * url로 endpoint를 지정
            ```python
            from rest_framework.authtoken.views import obtain_auth_token
            urlpatterns += [
            url(r'^api-token-auth/', obtain_auth_token),
            ]
            ```
        * API를 호출하여 token 발급
            ```shell
            > http POST http://server/api-token-auth/ username="id" password="pw"
            HTTP/1.0 200 OK
            Allow: POST, OPTIONS
            Content-Type: application/json
            Date: Fri, 01 Dec 2017 06:49:35 GMT
            Server: WSGIServer/0.2 CPython/3.6.1
            {
            "token": "9cdd705c0a0e5adb8671d22bd6c7a99bbacab227"
            }
            ```
**3. 발급된 토큰으로 API 호출**
* 터미널
    ```shell
    export TOKEN='9cdd705c0a0e5adb8671d22bd6c7a99bbacab227'
    > http GET http://localhost:8000 "Authroization: Token $TOKEN"
    ```
* 파이썬
    ```python
    import requests
    token='9cdd705c0a0e5adb8671d22bd6c7a99bbacab227'
    headers = {
    'Authroization': 'Token '+token, #띄어쓰기 주의!
    }
    response = requests.get('http://localhost:8000', headers=headers)
    ```
