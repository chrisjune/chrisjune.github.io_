---
layout: post-sidebar
date: 2018-10-13
title: "DRF 07 - Pagination"
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
API호출시 내용 길이를 관리하는 `Pagination`에 대해 알아본다

* 내용이 많을 경우 하나의 API요청처리시 서버와 클라이언트 양쪽모두 부하를 유발할 수 있다.
* 따라서, Pagination을 통해 출력하고자하는 데이터를 적절하게 관리해야 한다.

## Default Pagination

```python
from rest_framework.pagination import PageNumberPagination

class GenericAPIView(ApiView):
    pagination_class = PageNumberPagination

# 프로젝트/settings.py
REST_FRAMEWORK = {
    'PAGE_SIZE': 20,  # 디폴트 값은 None으로서 페이징 비활성화
}
```

## Custom pagination

```python
class PostNumberPagination(PageNumberPagination):
	page_size =5
	
class PostViewSet(ModelViewSet):
	pagination_class = PostNumberPagination
```

* `rest_framework 3.7.0 이상버전`에서는 디폴트 페이징 클래스 설정이 None이기 때문에 추가 전역설정필요
    ```python
    REST_FRAMEWORK = {
        'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    }
    ```