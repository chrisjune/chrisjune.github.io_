---
layout: post-sidebar
date: 2018-07-21
title: "인코딩과 디코딩 한번에 정리!"
categories: coding
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 10
feature_image: feature-book
show_related_posts: true
square_related: recommend-sunset
---
* 인코딩: 문자열을 바이트로 변환함
	* '한글' -> b'\xc7\xd1\xb1\xdb'
* 디코딩: 바이트를 문자열로 변환
	* b'\xc7\xd1\xb1\xdb' -> '한글' 
* 파이썬에서는 문자열을 유니코드로 처리 즉, 문자열(String) = Unicode

## 인코딩
* string -> bytes
* 유니코드 -> utf8, euc-kr, ascii

```python
str = '한글'
encoded = str.encode('utf-8')
b'\xed\x95\x9c\xea\xb8\x80'

encoded = str.encode('euc-kr')
b'\xc7\xd1\xb1\xdb'
```

## 디코딩
* bytes -> string
* utf8, euc-kr, ascii -> unicode

```python
str = b'\xc7\xd1\xb1\xdb'
decoded = str.decode('euc-kr')
'한글'
```

## 인코딩 바이트문자열간 변환하려면?
* bytes -> str -> bytes
* utf8 -> unicode -> euc-kr

```python
str = b'\xc7\xd1\xb1\xdb'
decoded = str.decode('euc-kr')
'한글'
encoded = decoded.encode('utf-8')
b'\xc7\xd1\xb1\xdb'
```

## 개발자로서 숙지해야 할 것
1. 입력으로 받은 바이트 문자열을 가능한한 가장 빨리 유니코드 문자열로 디코딩 할 것.
2. 변환된 유니코드 문자열로만 함수나 클래스등에서 사용할 것.
3. 입력에 대한 결과를 전송하는 마지막 부분에서만 유니코드 문자열을 인코딩해서 리턴할 것.