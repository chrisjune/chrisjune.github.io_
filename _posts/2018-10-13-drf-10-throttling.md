---
layout: post-sidebar
date: 2018-10-13
title: "DRF 10 - Throttling"
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
**API 호출 횟수 제한(Throttling)에 대해 알아본다**

## 용어

* Rates: 단위 기간내 최대 호출횟수
* Scope: Rate의 alias
* Throttle: 최대 호출 갯수 결정클래스

## 기본 제공 Throttle

* 제한 초과시 `429 Too Many Requests` 응답
* `AnonRateThrottle`
    * 인증요청에는 제한을 두지 않고, 비인증 요청에는 IP 별로 횟수 제한
    * Throttle 클래스별로 scope을 1개만 지정할 수 있습니다.
    * 디폴트 scope: `anon`
* `UserRateThrottle`
    * 인증요청에는 유저 별로 횟수를 제한하고, 비인증 요청에는 IP 별로 횟수 제한
    * Throttle 클래스별로 scope을 1개만 지정할 수 있습니다.
    * 디폴트 scope: `user`
* `ScopedRateThrottle`
    * 인증요청에는 유저 별로 횟수를 제한하고, 비인증 요청에는 IP 별로 횟수 제한
    * 여러 APIView내 `throttle\_scope`값을 읽어들여, APIView별로 다른 Scope을 적용해줍니다.
  
    ```python
    # Throttle for all APIview
    ## settings.py`
    'DEFAULT_THROTTLE_CLASSES':[
    'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES':{
    'user': '10/day',
    }

    # views.py
    from rest_framework.throttling import UserRateThrottle
    class PostViewSet(ViewSet):
    throttle_classes = UserRateThrottle
    ```
    ```python
    # Custom throttle for APIView
    ## settings.py
    REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
    'rest_framework.throttling.ScopedRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
    'contact': '1000/day',
    'upload': '20/day',
    },
    }
    ## views.py
    class ContactListView(APIView):
    throttle_scope = 'contact'
    class ContactDetailView(APIView):
    throttle_scope = 'contact'
    class UploadView(APIView):
    throttle_scope = 'upload'
    ```
#### Rates 포멧

* '숫자/간격'
    * 숫자: 지정시간내 최대 요청가능 횟수
    * 간격
        * s:초
        * m:분
        * h:시
        * d:일
    ```python
    1000/d # 하루에 1천번 호출 가능
    200/h # 시간당 200번 호출 가능
    10/s # 초당 10번 호출 가능
    ```
#### Client IP

X-Forwarded-For 헤더값과 REMOTE_ADDR WSGI 변수값을 참조하여 IP를 확정
X-Forwarded-For 헤더값을 우선시함
```python
xff = request.META.get('HTTP_X_FORWARDED_FOR')
remote_addr = request.META.get('REMOTE_ADDR')
if xff:
client_ip = ''.join(xff.split())
else:
client_ip = remote_addr
```

* settings.py에 제한 가능/ APIView 에 제한 가능
* 숫자/ 간격 :초/분/시/일
    ```python
        'DEFAULT_THROTTLE_CLASSES':[
            'rest_framework.throttling.UserRateThrottle',
        ],
        'DEFAULT_THROTTLE_RATES':{
            'user': '10/day',
        }
    ```
