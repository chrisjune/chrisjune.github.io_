---
layout: post-sidebar
date: 2018-08-06
title: "DRF 04 - Viewset과 Router"
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
**Viewset은 한번에 하나의 URL만 매핑한다. Router를 활용하여 2개 URL에 매핑하는 방법을 알아본다.**

## ViewSet
* 일반적으로 CBV는 하나의 View(URL)만 처리한다
* Viewset은 CBV의 확장 형태로 목록단위와 Record단위 View를 처리한다.
* `list`/`create`/`retrieve`/`update`/`partial_update`/`destory` 함수를 모두 지원한다.
* 하지만, 하나의 URL에서 모두 처리하는 것은 불가능하다. 왜냐하면, 일반적인 REST API설계에서 벗어나기 때문이다.
* Viewset은 `.as_view({'http_method':'member_fuction'})`를 호출하여 하나의 뷰함수를 하나의 URL에 매핑함
* Viewset URL 매핑 맛보기
    ```python
    # list route
    post_list = PostViewSet.as_view({
    'get': 'list',
        'post': 'create',
    })

    # detail route
    post_detail = PostViewSet.as_view({
        'get': 'retrieve',
        'put': 'update',
        'patch': 'partial_update',
        'delete': 'destroy',
    })
    ```

## Router를 활용하여 여러개의 View 하나의 URL 매핑
* Router는 Viewset에 두개 URL에 매핑해준다.
* viewset이 지원하는 메소드에 대해서 URL매핑을 수행한다.
* list_route: 데이터 리스트 출력, 데이터 생성
* detail_route: 상세 데이터 출력, 데이터 수정, 데이터 부분수정, 데이터 삭제
    * `list router` -> `/prefix/`
    * `detail router` -> `/prefix/pk/`
* Router 맛보기
    ```python
    from rest_framework.routers import DefaultRouter

    router = DefaultRouter()
    router.register(r'prefix', PostViewSet)

    urlpatterns = [
        url(r'', include(router.urls)),
    ]
    ```
---

## 데코레이터를 활용하여 Viewset에 Custom View 추가
* `list route`와 `detail route`에 API추가
* Viewset내에서 `@list_route`와 `@detail_route` 데코레이터로 구현
* URL매핑은 데코레이터에 따라 Router에서 자동 매핑됨
* Custom Vivew 맛보기
    ```python
    from rest_framework.decorators import list_route, detail_route
    from rest_framework.response import Response


    class PostViewSet(ModelViewSet):
        queryset = PostViewSet.objects.all()
        serializer_class = PostSerializer

        @list_route()  # 목록 단위로 적용할 API이기에, list_route 장식자 사용
        def public_list(self, request):
            qs = self.queryset.filter(is_public=True)  # Post모델에 is_public 필드가 있을 경우
            serializer = self.get_serializer(qs, many=True)
            return Response(serializer.data)
    #URL 매핑은 `/prefix/함수명/`으로서 `/post/public_list/`


        @detail_route(methods=['patch'])  # Record 단위로 적용할 API이기에, detail_route 장식자 사용
        def set_public(self, request, pk):
            instance = self.get_object()
            instance.is_public = True
            instance.save()
            serializer = self.get_serializer(instance)
            return Response(serializer.data)
    # URL 매핑은 `/prefix/{pk}/함수명/`으로서 `/post/{pk}/set_public/`
    ```

> 본 내용은 [`askDjango`](https://www.askcompany.kr/)강좌를 참고하였습니다