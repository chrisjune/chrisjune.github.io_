---
layout: post-sidebar
date: 2018-07-07
title: "장고 - Regex 정규식"
categories: django
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 4
feature_image: feature-san-fran
show_related_posts: true
square_related: recommend-spain
---
`askdjango`를 참고하였습니다

### 정규식 패턴
* []: 한문자를 의미
* [0-9] : \d
* [0-9a-fA-F] : 16진수 한글자
* [ㄱ-힣] : 한글 한글자
* ord()함수: 문자열의 ascii값
* r'' :raw의 뜻으로, 이스케이프문자를 자동으로 변환해줌 \d -> \\d
* 횟수
	* 0 또는 1 : ?
	* 0 회 이상: *
	* 1 회 이상: +
	* m 회 : {m}
	
### 장고 URL패턴	
* r'^(?P<id>\d+)/$ : /10/ 
* (?P): 영역의 정규표현식을 적용할 것을 의미
* <x> : x라는 변수명으로 인자를 보냄
* 뷰 인자로 받는 모든 값은 문자열 타입
* url(r'^sum/(?P<x>\d+)/(?P<y>\d+)$',views.ts):
	* /sum/10/20