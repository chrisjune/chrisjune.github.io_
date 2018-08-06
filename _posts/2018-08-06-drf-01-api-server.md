---
layout: post-sidebar
date: 2018-08-06
title: "DRF 01 - API서버"
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
REST API서버란 무엇이고, FBV, CBV를 만들며 Django Rest-framework에 대해 알아본다.

## API서버란
* 앱/웹을 위한 `데이터 중심`의 서비스이며 버전개념이 존재한다.

## REST API
* REST API서버는 데이터의 Create, Read, Update, Delete의 액션을 제공한다.
* 하지만, 대부분의 REST API라는 API들은 REST 아키텍쳐가 아니며, 단순히 HTTP프로토콜을 통한 HTTP API라고 해야한다.
* django rest-framework는 django서버에서 데이터 핸들링 작업을 보다 쉽게 만들어주는 제공하는 파이썬 라이브러리이다.

## API호출
1. JS를 통한 호출
2. 브라우져, Selenium을 통한 호출
3. 웹요청개발 프로그램을 통한 호출
	* GUI: Postman, Insomnia
	* CLI: cURL, HTTPie
	* library: requests

## HTTPie
* GET
```shell
> http GET 요청할주소 GET인자명==값 GET인자명==값
```
* POST
```shell
> http --json POST 요청할주소 GET인자명==값 GET인자명==값 POST인자명=값 POST인자명=값
> http --form POST 요청할주소 GET인자명==값 GET인자명==값 POST인자명=값 POST인자명=값
# form 옵션 지정 시 : multipart/form-data 요청, HTML Form과 동일
# json 옵션을 지정하거나 생략 시 : application/json 요청, 요청 데이터를 JSON포맷으로 직렬화해서 전달
```
* PUT
```shell
> http PUT 요청할주소 GET인자명==값 GET인자명==값 PUT인자명=값 PUT인자명값
```
* DELETE
```shell
> http DELETE 요청할주소 GET인자명==값 GET인자명==값
```
* [`httpbin.org`](http://httpbin.org)/GET, POST, DELETE  사이트로 호출 테스트가능
    * Request
```shell
> http GET httpbin.org/get
```
    * Response
```shell
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 256
Content-Type: application/json
Date: Mon, 06 Aug 2018 12:07:53 GMT
Server: gunicorn/19.9.0
Via: 1.1 vegur
{
    "args": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Connection": "close",
        "Host": "httpbin.org",
        "User-Agent": "HTTPie/0.9.9"
    },
    "origin": "61.97.139.163",
    "url": "http://httpbin.org/get"
}
```

## Django를 활용한 REST API 구현 맛보기 Tutorial
### FBV(Function Based View)
* 구현 순서
    1. model
    2. form ModelForm
    3. views request function return JsonResponse
    4. urls
* Model
    ```python
    # 모델 구현 : myapp/models.py
    from django.db import models

    class Post(models.Model):
        message = models.TextField()
    ```
* Form
    ```python
    # 폼 구현 : myapp/forms.py
    from django import forms
    from .models import Post

    class PostForm(forms.ModelForm):
        class Meta:
            model = Post
            fields = '__all__'
    ```
* View
    ```python
    # 뷰 구현 : myapp/views.py
    from django.http import HttpResponse, JsonResponse
    from django.http import QueryDict
    from django.shortcuts import get_object_or_404
    from django.views.decorators.csrf import csrf_exempt
    from .models import Post
    from .forms import PostForm

    @csrf_exempt
    def post_list(request):
        if request.method == 'GET':
            qs = Post.objects.all()
            data = [{'pk': post.pk, 'message': post.message} for post in qs]  # 수동 JSON 직렬화
            return JsonResponse(data, safe=False)
        elif request.method == 'POST':
            form = PostForm(request.POST)
            if form.is_valid():
                post = form.save()
                return HttpResponse(status=201)
            data = form.errors
            return JsonResponse(data, status=400)

    @csrf_exempt
    def post_detail(request, pk):
        post = get_object_or_404(Post, pk=pk)

        if request.method == 'GET':
            return JsonResponse({'pk': post.pk, 'message': post.message})
        elif request.method == 'PUT':
            put = QueryDict(request.body)
            form = PostForm(put, instance=post)
            if form.is_valid():
                post = form.save()
                data = {'pk': post.pk, 'message': post.message}
                return JsonResponse(data=data, status=201)
            return JsonResponse(form.errors)
        elif request.method == 'DELETE':
            post.delete()
            return HttpResponse('', status=204)
    ```
* Urls 구현
    ```python
    # URLConf 구현 : myapp/urls.py
    from django.conf.urls import url
    from .views import post_list, post_detail

    urlpatterns = [
        url(r'^post/$', post_list, name='post-list'),
        url(r'^post/(?P<pk>\d+)/$', post_detail, name='post-detail'),
    ]
    ```

### CBV(Class Based View)
* 구현순서
    1. model
    2. serializer ModelSerializer
    3. views ModelViewSet
    4. urls DefaultRouter
* Model
    ```python
    모델 : myapp/models.py
    from django.db import models

    class Post(models.Model):
        title = models.CharField(max_length=100)
    Serializer : myapp/serializers.py (Form과 유사)
    from rest_framework import serializers
    from .models import Post
    ```
* Serializer
    ```python
    # Serializer: myapp/serializers.py
    class PostSerializer(serializers.ModelSerializer):
        class Meta:
            model = Post
            fields = '__all__'
    ```
* View
    ```python
    # 뷰 : myapp/views.py
    from rest_framework import viewsets
    from .models import Post
    from .serializers import PostSerializer

    class PostViewSet(viewsets.ModelViewSet):
        queryset = Post.objects.all()
        serializer_class = PostSerializer
    ```
* URL
    ```python
    # URL : myapp/urls.py
    from django.conf.urls import include, url
    from rest_framework.routers import DefaultRouter
    from .views import PostViewSet

    router = DefaultRouter()
    router.register(r'post', PostViewSet)

    urlpatterns = [
        url(r'', include(router.urls)),
    ]
    ```
> 본 내용은 [`askDjango`](https://www.askcompany.kr/)강좌를 참고하였습니다