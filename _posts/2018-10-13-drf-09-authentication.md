---
layout: post-sidebar
date: 2018-10-13
title: "DRF 09 - Authentication"
categories: django
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 20
feature_image: feature-wolf
show_related_posts: true
square_related: recommend-fire
---
**Authentication & Permission에 대해 알아본다**

## 지원하는 인증의 종류

* `SessionAuthentication`
	* 세션을 통한 인증 여부 체크
	* APIView를 통해 디폴트 지정 (우선순위 1)
* `BasicAuthentication`
	* Basic 인증헤더를 통한 인증 수행 (ex: Authorization: Basic YWxsaWV1czE6MTAyOXNoYWtl)
	* APIView를 통해 디폴트 지정 (우선순위 2)
* `TokenAuthentication`
	* Token 헤더를 통한 인증 수행 (ex: Authorization: Token 401f7ac837da42b97f613d789819ff93537bee6a)
* `RemoteUserAuthentication`
	* User 정보가 다른 서비스에서 관리될 때, Remote 인증 (장고 공식문서)
	* Remote-User 헤더를 통한 인증 수행

#### HTTP Basic 인증 헤더

* httpie 이용시 --auth id:password로 인자 지정가능
    ```python
    #BasicAuth 인증
    # Request
    ❯ http -a username:password --form POST httpbin.org/post

    # Response
    HTTP/1.1 200 OK
    Access-Control-Allow-Credentials: true
    Access-Control-Allow-Origin: *
    Connection: keep-alive
    Content-Length: 477
    Content-Type: application/json
    Date: Tue, 31 Jul 2018 13:11:23 GMT
    Server: gunicorn/19.9.0
    Via: 1.1 vegur

    {
        "args": {},
        "data": "",
        "files": {},
        "form": {},
        "headers": {
            "Accept": "*/*",
            "Accept-Encoding": "gzip, deflate",
            "Authorization": "Basic dXNlcm5hbWU6cGFzc3dvcmQ=",
            "Connection": "close",
            "Content-Length": "0",
            "Content-Type": "application/x-www-form-urlencoded; charset=utf-8",
            "Host": "httpbin.org",
            "User-Agent": "HTTPie/0.9.9"
        },
        "json": null,
        "origin": "192.3.3.3",
        "url": "http://httpbin.org/post"
    }
    ```

    ```shell
    >>> from base64 import b64encode
    >>> str = '한글비밀번호'
    >>> encoded = str.encode()
    >>> encoded
    b'\xed\x95\x9c\xea\xb8\x80\xeb\xb9\x84\xeb\xb0\x80\xeb\xb2\x88\xed\x98\xb8'
    >>> based = b64encode(encoded)
    >>> based
    b'7ZWc6riA67mE67CA67KI7Zi4'
    >>> from base64 import b64decode
    >>> unbased = b64decode(based)
    >>> unbased
    b'\xed\x95\x9c\xea\xb8\x80\xeb\xb9\x84\xeb\xb0\x80\xeb\xb2\x88\xed\x98\xb8'
    >>> unbased.decode()
    '한글비밀번호'
    ```

## Permission
* `django-rest_framework`에서는 `Permission`을 기본적으로 제공해줌
    * `is_superuser`: 모든 권한 허용
    * `is_staff`: admin페이지 접속만 가능, 일반 user와 동일
    * `is_active`: false 인 경우, 권한지정과 상관없이 모두 불허

* `AllowAny` : 인증여부에 상관없이, 뷰 호출 허용 (디폴트 지정)
* `IsAuthenticated` : 인증된 요청에 한해서, 뷰 호출 허용
* `IsAdminUser` : Staff 인증 요청에 한해서, 뷰 호출 허용
* `IsAuthenticatedOrReadOnly` : 비인증 요청에게는, 읽기 권한만 허용
* `DjangoModelPermissions` : 인증된 요청에 한해서만 뷰 호출을 허용하고, 추가로 유저별 인증 권한체크를 수행
* `DjangoModelPermissionsOrAnonReadOnly` : DjangoModelPermissions과 유사하나, 비인증 요청에 대해서는 읽기 권한만 허용
* `DjangoObjectPermissions`
비인증된 요청은 거부
인증된 요청에 한, Record 접근에 대한 권한체크를 추가로 수행

#### 커스텀 Permission 만들기

* django-rest-framework에서 기본 제공해주는 Permission만으로도 대개 충분합니다만, 필요에 의해 커스텀 Permission을 만들고 싶을 수 있습니다.
* 모든 Permission 클래스는 다음 2가지 함수를 선택적으로 구현합니다.

* `has_permission`(request, view) : 뷰 호출 접근권한
APIView 접근 시, 체크
이를 구현한 Permission 클래스 : IsAuthenticated, IsAuthenticated, IsAdminUser, IsAuthenticatedOrReadOnly, DjangoModelPermissions, DjangoModelPermissionsOrAnonReadOnly
* `has_object_permission`(request, view, obj) : 개별 Record 접근권한
APIView의 get_object함수를 통해 object 획득 시, 체크 - 개별 GET/PUT/DELETE 요청
브라우저를 통한 API 접근에서 CREATE/UPDATE Form 노출 여부 확인 시에, 체크
이를 구현한 Permission 클래스 : DjangoObjectPermissions

##### 포스팅 작성자에 한해서, 수정 권한은 부여하되 삭제권한은 superuser에게만 부여

* 
    ```python

    from rest_framework import permissions

    class IsAuthorUpdateOrReadonly(permissions.BasePermission):
        # 인증된 유저에 한해, 목록조회/포스팅등록을 허용
        def has_permission(self, request, view):
            return request.user.is_authenticated

        # superuser에게는 삭제 권한만 부여하고
        # 작성자에게는 수정 권한만 부여해봅시다.
        def has_object_permission(self, request, view, obj):
            # 조회 요청(GET, HEAD, OPTIONS) 에 대해서는 인증여부에 상관없이 허용
            if request.method in permissions.SAFE_METHODS:
                return True

            # 삭제 요청의 경우, superuser에게만 허용
            if (request.method == 'DELETE'):
                return request.user.is_superuser  # request.user.is_staff

            # PUT 요청에 대해, 작성자일 경우에만 요청 허용
            return obj.author == request.user
    ```

##### 자동생성되는 값이 수정되지 않지만 보여져야 하는 경우 ReadOnlyField(source)로 처리가능

* 
    ```python
    serializers.ReadOnlyField(source='참조할필드명.속성명')
    ```
