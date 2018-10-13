---
layout: post-sidebar
date: 2018-10-13
title: "DRF 08 - Serializer 유효성검사"
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
Serializer의 중요한 기능인 `Validate`에 대해 알아본다

## Serializer 생성자

* Serializer 첫번째인자:instance, 두번째인자:data(dictionary type)
    * form = MySerializer(my_instance)
    * form = MySerializer(my_instance, request.POST)
    * form = MySerializer(data=request.POST)
* instance가 없으면 .create(), instance가 주어지면 .update()
    ```python
    # Create, Update Example
    # .save() will create a new instance.
    serializer = CommentSerializer(data=data)

    # .save() will update the existing `comment` instance.
    serializer = CommentSerializer(comment, data=data)
    ```
    ```python
    ## BaseSerializer 
    if self.instance is not None:
        self.instance = self.update(self.instance, validated_data)
        assert self.instance is not None, (
            '`update()` did not return an object instance.'
        )
    else:
        self.instance = self.create(validated_data)
        assert self.instance is not None, (
            '`create()` did not return an object instance.'
        )
    ```
* serializer에 data가 주입된 후
    1. `.is_valid()`: 유효성 검
    2. `.initial_data`: 주입된 data
    3. `.validated_data`: 유효성 검증에 통과한 값
    4. `.save()`: .validated_data를 모델에 저장
    5. `.errors()`: 유효성 검사 에러내역
    6. `.data`: 유효성 검사 후에, 갱신된 인스턴스에 대한 필드값 사전
* `required=False` 인 필드의 경우, 필드가 포함되지 않을 경우  Validation 유효성검사를 하지않는다.

## Validators

> 데이터를 `Deserializer`할때, `.is_valid()`호출되거나, 객체를 저장할때 `validated_data`에 접근이 가능함.
장고 기본 validators과 더불어, `django-rest-framework`에서는 유일성 여부 체크를 도와주는 `Validator`를 제공해주며, queryset 범위를 제한하여 지정 범위 내에서의 유일성 여부를 체크할 수 있습니다.

* `UniqueValidator` : 지정 필드가 지정 QuerySet범위에서 Unique한지 체크
	* 모델 필드 `unique=True` 설정에 대응하여, 자동 추가
* `UniqueTogetherValidator`
	* 모델 클래스.Meta.unique_together 속성에 대응하여, 자동 추가
* `UniqueForDateValidator`
	* 모델 필드 unique_for_date=True 설정에 대응하여, 자동 추가
* `UniqueForMonthValidator` :
	* 모델 필드 unique_for_month=True 설정에 대응하여, 자동 추가
* `UniqueForYearValidator`
	* 모델 필드 unique_for_year=True 설정에 대응하여, 자동 추가

#### Serializer에서 유효성검사 함수지정

* Validator는 모델측에서 관리하는 것이 좋다.
* 필드 정의 시에 validators 인자를 지정하는 것이 DRF구조적으로 이상적임

## 필드 검사

#### 특정필드 검사

* validate_필드명으로 구현해줌
* 함수 인자로 해당값이 value인자로 전달됨
* 리턴값을 통해서 값 변환 가능
    ```python
    class PostSerializer(serializers.Serializer):
        title = serializers.CharField(max_length=100)

        def validate_title(self, value):
            if 'django' not in value:
                raise ValidationError('제목에 필히 django가 포함되어야합니다.')
            return value
    ```
  
#### 다수 필드 검사

* validate()로 구현
    ```python
    class PostSerializer(serializers.Serializer):
        title = serializers.CharField(max_length=100)

        def validate(self, data):
            if 'django' not in data['title']:
                raise ValidationError('제목에 필히 django가 포함되어야합니다.')
            return data
    ```
## Mixin Perform-Function
* API결과를 DB에 반영하고자 할 때, create, update, destroy 커스텀하고싶다면
perform_함수명으로 추가구현가능
    ```python
    # 예) ip 저장, 유저정보저장
    def perform_create(self, serializer):
        serializer.save(ip=self.request.META['REMOTE_ADDR'])
    ```