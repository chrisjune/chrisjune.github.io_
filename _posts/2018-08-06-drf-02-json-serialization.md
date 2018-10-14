---
layout: post-sidebar
date: 2018-08-06
title: "DRF 02 - JSON 직렬화"
categories: django
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 15
feature_image: feature-wolf
show_related_posts: true
square_related: recommend-fire
---
**데이터 직렬화(Serialization)의 개념을 이해하고, Serializer를 구현할 수 있다.**

## 데이터 직렬화
* 모든 프로그램통신에서 데이터는 반드시 문자열로 표현되어야함.
* 보내는쪽: 객체데이터를 문자로 변환하여 전송
    * 이를 직렬화 serializaion이라는 어려운 용어를 사용한다
* 받는쪽: 받은 문자열을 다시 객체로 변환하여 활용
    * 이를 역직렬화(Deserialization)이라고 한다
* 모든 언어에서 XML, JSON을 지원하며, 최근 대부분API서버에서는 Json포멧을 활용

## Json
* 웹어플리케이션에서는 HTML포맷으로 통신
* API서버는 주로 JSON포맷으로 통신
* 장고 API뷰에서도 JSON포맷을 사용
* JSON: 타 언어, 플랫폼과 통신할 때 사용
* PICKLE: 파이썬 전용 포맷, 파이썬 버전의 특성의 영향을 받음
* 문자열 직렬화 맛보기
    ```python
    import json
    list = {"key":"value"}

    json_str = json.dumps(list)
    > '{"key": "value"}'
    ```
* 문자열 역직렬화 맛보기
    ```python
    import json
    str = '{"key": "value"}'
    list = json.loads(str)
    > {"key": "value"}
    ```
---

## Django에서 JSON 직렬화
* 장고에서는 표준라이브러리 json.JSONEncoder를 상속받아 `DjangoJSONEncoder`를 제공
* QuerySet, Model인스턴스에 대한 직렬화는 불가능, for loop로 list를 직접 구현해줘야함

## Django Rest-framework에서 JSON 직렬화
* rest_framework.renderer.JSONRender는 json.JSONEncoder를 상속받아 직렬화 구현. 
* `DjangoJSONEncoder`보다 많은 데이터 타입을 직렬화하지만 역시 Model, QueyrSet인스턴스의 직렬화 불가능

## ModelSerializer
* QuerySet, Model을 JSONRenderer에서 변환 가능한 가능한 형태로 변환 장고의 ModelForm과 유사한 형태
* Django Form/ModelForm 와 Serializer/ModelSerializer
* 모델에서 데이터를 읽는 다는 것과 입력된 데이터에 대한 유효성 검사하는 것이 동일하다
* Form은 Form HTML을 생성, Serializer는 Json 문자열을 생성하는 것에서 차이가 있다.
* ModelSerializer 맛보기
    ```python
    from rest_framework.serializers import ModelSerializer

    class PostSerializer(ModelSerializer):
        class Meta:
            model = Post
            fields = '__all__'
    ```
* 직렬화된 데이터 활용 맛보기
    ```python
    post_obj = Post.objects.first() # 모델 인스턴스
    post = PostSerializer(post_obj)
    post.data # Ordered dict type
    ```
* 쿼리셋직렬화 맛보기
    * QuerySet은 ModelInstance의 집합이기 때문에, 직렬화시 may=True인자로 호출해야함
    ```python
    post_objs = Post.objects.all()
    post = PostSerializer(post_objs, many=True) # QuerySet의 경우 many=True
    ```
---
## View에서 직렬화된 데이터 활용
* JSON포멧을 직렬화된 데이터는 View를 통해서 응답이 이루어져야 한다
* JsonResponse 맛보기
    ```python
    from django.http import JsonResonse

    response = JsonResponse(
        data = data,
        encoder = DjangoJSONEncoder,
        safe = False # dict type체크를 목적(Dictionary: True, 이외 False)
        json_dumps_params = {'ensure_ascii': False},
        **kwargs
    )
    ```
* Rest-framework Response 맛보기
    ```python
    from rest_framework.response import Resonse
    obj = Post.objects.all()
    serializer = PostModelSerializer(obj, many=True)
    response = Response(serializer.data)
    ```

> 본 내용은 [`askDjango`](https://www.askcompany.kr/)강좌를 참고하였습니다