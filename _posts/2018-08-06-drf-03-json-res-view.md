---
layout: post-sidebar
date: 2018-08-06
title: "DRF 03 - JSON 응답뷰(APIView, ViewSet)"
categories: django
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 25
feature_image: feature-wolf
show_related_posts: true
square_related: recommend-fire
---
> Django Rest-framework에서 제공하는 APIView와 ViewSet에 대해 알아보고 Serialized 된 데이터를 View에 표현해본다

## View
* Serializer의 데이터를 활용하여 HTTP요청을 처리한다
* 데이터 흐름: Model -> Serializer -> View
* Serializer와 View 맛보기
    * Serializer
        ```python
        class PostSerializer(serializers.ModelSerializer):
            class Meta:
                model = Post
                fields = ['title', 'content']
        ```
    * View
        ```python
        serializer = PostSerializer(data=request.POST)
        if serializer.is_valid():
            serializer.save()
            return JsonResonse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)
        ```

## APIView 활용하여 FBV, CBV 구현
* FBV는 구식, CBV는 신식이 아니라, 용도별로 구현방법이 다르다.
* FBV(Function Based View): 특별한 뷰를 구현할 때 CBV보다 간단하다.
    * `@api_view` 데코레이터를 활용하여 구현
* CBV: 여러 뷰를 통해 걸쳐 반복되는 루틴의 경우 구현시 유리하다.
    * `APIView`를 상속하여 구현
* APIView 맛보기
* APIView를 활용하여 CBV 맛보기
* `APIView.as_view()`에서 `csrf_exempt` 처리된 뷰를 만들어줌
    ```python
    from rest_framework.response import Response
    from rest_framework.views import APIView
    from .models import Post
    from .serializers import PostSerializer

    class PostListAPIView(APIView):
        def get(self, request):
            serializer = PostSerializer(Post.objects.all(), many=True)
            return Response(serializer.data)

        def post(self, request):
            serializer = PostSerializer(data=request.data)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data, status=201)
            return Response(serializer.errors, status=400)
    ```
* @api_view를 활용하여 FBV 맛보기
    ```python
    from django.http import get_object_or_404
    from rest_framework import status, Response
    from rest_framework.decorators import api_view
    from .models import Post
    from .serializers import PostSerializer

    @csrf_exempt
    @api_view(['GET', 'POST'])
    def post_list(request):
        if request.method == 'GET':
            serializer = PostSerializer(Post.objects.all(), many=True)
            return Response(serializer.data)
        else:
            serializer = PostSerializer(data=request.data)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data, status=201)
            return Response(serializer.errors, status=400)
    ```
---

## Mixin을 활용한 View 구현(APIView 상속)
* `APIView`를 직접 상속하여 구현시 중복로직이 발생한다.
* 따라서, 중복되는 부분을 미리 묶어놓은 `Mixin`(Create, List, Retrieve, Update, Destroy)을 지원해줌.
* Mixin은 말그대로 섞어놓은 클래스 다른 기능은 없음
* `Mixin`을 활용한 CBV 맛보기
    ```python
    from rest_framework import generics
    from rest_framework import mixins

    class PostListAPIView(mixins.ListModelMixin, mixins.CreateModelMixin,
                    generics.GenericAPIView):
        queryset = Post.objects.all()
        serializer_class = PostSerializer

        def get(self, request, *args, **kwargs):
            return self.list(request, *args, **kwargs)

        def post(self, request, *args, **kwargs):
            return self.create(request, *args, **kwargs)
    ```
* `Generics`를 활용한 CBV(여러 Mixin을 하나로 묶음)
    ```python
    from rest_framework import generics

    class PostListAPIView(generics.ListCreateAPIView):
        queryset = Post.objects.all()
        serializer_class = PostSerializer
    ```
---

## Viewset을 활용하여 View 구현
* `REST API`는 생성/목록조회, 상세조회/수정/삭제를 위해 각각의 URL(View)가 필요하다.
* 두개의 view는 동일한 serializer와 queryset을 지정하기 때문에 한번에 처리가능하도록
* rest-framework에서는 ViewSet을 제공함.
* ViewSet은 CBV가 아니라 2개의 뷰를 만들어주는 `헬퍼 클래스`
    * `viewsets.ReadOnlyModelViewSet`
        * 목록 조회, 특정 레코드 조회를 지원 => 2개의 URL 지원
    * `viewsets.ModelViewSet`
        * 목록 조회, 생성, 특정 레코드 조회/수정/삭제 지원 => 2개의 URL 지원
* Router를 통해 일괄적으로 `URLConf`에 등록가능함
* Viewset 맛보기
* Views
    ```python
    from rest_framework import viewsets

    class UserViewSet(viewsets.ReadOnlyModelViewSet):
        queryset = User.objects.all()
        serializer_class = UserSerializer
    ```
* Urls
    ```python
    from rest_framework.routers import DefaultRouter

    router = DefaultRouter()
    router.register(r'user', views.UserViewSet)

    urlpatterns = [
        url(r'', include(router.urls)),
    ]
    ```
---
## APIView 클래스와 api_view장식자(Default 값)
* 직렬화 클래스 지정 : `renderer_classes` 속성 (list)
    * `rest_framework.renderers.JSONRenderer` : JSON 직렬화
    * `rest_framework.renderers.TemplateHTMLRenderer` : HTML 페이지 직렬화
* 비직렬화 클래스 지정 : `parser_classes` 속성 (list)
    * `rest_framework.parsers.JSONParser` : JSON 포맷 처리
    * `rest_framework.parsers.FormParser`
    * `rest_framework.parsers.MultiPartParser`
* 인증 클래스 지정 : `authentication_classes` 속성 (list)
    * `rest_framework.authentication.SessionAuthentication` : 세션에 기반한 인증
    * `rest_framework.authentication.BasicAuthentication` : HTTP Basic 인증
* 사용량 제한 클래스 지정 : `throttle_classes` 속성 (list)
    * 디폴트 : 빈 튜플
* 권한 클래스 지정 : `permission_classes` 속성 (list)
    * `rest_framework.permissions.AllowAny` : 누구라도 접근 허용
* 요청에 따라 적절한 직렬화/비직렬화 클래스를 선택 : `content_negotiation_class` 속성 (문자열)
    * 같은 URL로의 요청이지만, JSON응답을 요구하는 것이냐 / HTML응답을 요구하는 것인지 판단
    * `rest_framework.negotiation.DefaultContentNegotiation`
* 요청 내역에서 API 버전 정보를 탐지할 클래스 지정 : `versioning_class` 속성
    * 디폴트 : `None` : API 버전 정보를 탐지하지 않겠다.
    * 요청 URL에서, GET인자에서, HEADER에서 버전정보를 탐지하여, 해당 버전의 API뷰를 호출토록 합니다.

> 본 내용은 [`askDjango`](https://www.askcompany.kr/)강좌를 참고하였습니다