---
layout: post-sidebar
date: 2018-10-14
title: "DRF 11 - DRF Default Settings"
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
**DRF 기본 설정에 대해 알아본다**
> [**django-rest-framework sttings**](https://github.com/encode/django-rest-framework/blob/3.8.2/rest_framework/settings.py#L31)

## Default Settings

#### DEFAULT_RENDERER_CLASSES
* HTTP 최종응답 생성
* `JSONRenderer`: JSON 포맷
* `BrowsableAPIRenderer` : Browsable API 포맷 응답 (HTML)

#### DEFAULT_PARSER_CLASSES
* HTTP 요청 내역 처리
* `JSONParser`: JSON 포맷 요청 처리
* `FormParser`: enctype application/x-www-form-urlencoded 요청 처리
* `MultiPartParser` : encytpe multipart/form-data 요청 처리 (파일 업로드 지원)

#### DEFAULT_AUTHENTICATION_CLASSES
* HTTP 요청 인증
* `SessionAuthentication` : 세션으로 유저 인증
* `BasicAuthentication`: base64로 user_id:password 값 인증
  * 보안적으로 매우 취약하므로 못할짓

#### DEFAULT_PERMISSION_CLASSES
* API호출 권한
* Permission 내용 참조

#### DEFAULT_THROTTLE_CLASSES, DEFAULT_THROTTLE_RATES
* 특정 시간당 최대 요청 횟수 제한
* `DEFAULT_THROTTLE_CLASSES` : 최대 호출를 제한할 로직 (클래스)
* `DEFAULT_THROTTLE_RATES` : 최대 호출 횟수 지정

#### DEFAULT_PAGINATION_CLASS
* 페이징처리
* PAGE_SIZE: 1페이지 최대 갯수

#### 인코딩
* `UNICODE_JSON` = True
    * json.dumps시 ensure_ascii=False 적용. `UTF-8 인코딩`
* `COMPACT_JSON` = True
    * json.dumps시 ', ' 아닌 ',' 적용
* `STRICT_JSON` = True
    * NaN, Inf 값 존재시 ValueError raise
* `COERCE_DECIMAL_TO_STRING` = True
    * Decimal을 문자열 강제 변환여부
* `UPLOAD_FILES_USE_URL` = True
    * 파일명 대신 URL제공

#### Browseable API
* `HTML_SELECT_CUTOFF`: Choice 옵션 최대 허용수
* `HTML_SELECT_CUTOFF_TEXT`: 초과시 안내 메시지