---
layout: post-sidebar
date: 2018-07-07
title: "지옥에서온 Git"
categories: coding
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : true
read_time : 30
feature_image: feature-water
show_related_posts: false
square_related: recommend-spain
---
깃의 사용방법과 여러 명령어의 활용방법을 알아본다   
>해당 내용은 [`생활코딩`](https://opentutorials.org/course/2708)을 참고하였습니다


## 프로젝트 만들기 
* 현재의 디렉토리 git의 저장소로 만들기
```
mkdir gitfth
cd gitfth ; git init
```

## 관리대상등록
* git이 파일을 추적할 수 있도록 명령 
```
git add [file name]
```
* 폴더의 상태를 확인 
```
git status
```

## 정보설정
* 닉네이설정 
```
git config --global user.name "자신의 닉네임"
```
* 이메일설정 
```
git config --global user.email "자신의 이메일"
```

## Stage area
* 스테이지: 커밋 대기할 파일들이 가는 곳
* 레파지토리: 스테이지에서 커밋된 파일들이 있는 곳, 커밋을 하면 스테이지 위에있는 파일만 처리됨 

## 로그비교
* 로그에서 출력되는 버전간 차이점을 출력하고 싶을 때
```
git log -p
```
* 버전간의 차이점을 비교할 때
```
git diff 'ver1' 'ver2'
git add 전/후의 파일 내용을 비교할 때
git diff
```

## Reset vs Revert
* 과거시간으로 가버리기(타임머신)
```
git reset [version]
```
* 단, push후 reset은 절대 하면 안됨!!
* 수정하여 커밋하기
* git revert [version]

## Git Add 할때
* index에 주소, 파일명 정보가 들어있고
* 실제 파일내용은 objects/하위에 담겨있다
* git은 파일내용이 동일하면 내부적으로 동일한 객체로 취급한다
* 그 이유는 파일내용을 -> sha1 해쉬알고리즘 encoding한 값이 동일할 때, 같은 것으로 취급하기 때문이다

## Git commit 할때
* commit도 객체로 내부처리된다
* 커밋을 하면 스테이지 위에있는 파일만 처리됨 
* object의 종류
* blob -> tree -> commit

## Git Branch
* 고객에게 특별한 custom 기능을 추가할때
* 나중에 쉽게 빼기 좋은 기능을 추가할때
* 여러가지 개발된 내용을 통합하여 테스트할때
* git log --branches --decorate --graph
* git log --branches --decorate --graph --oneline
* master 엔 있고 exp엔 없는 것들 git log exp..master
* master엔 없고 exp엔 있는 것들 git log master..exp

## Git Merge
### 기본개념
* A로 B브랜치를 병합할때, 다른 branch를 내쪽으로 당겨온다고 생각하면 됨
* A <- B
* git checkout A
* git merge B
### Fast-forward vs Merge recursive
* fast-forward: 빨리감기
* 브랜치 이후로 별도의 마스터 커밋이 없을때 이루어짐.
* master의 head가 바뀌고 별도의 머지커밋이 생성되지 않음
### Merge Recursive: 자동머지
* 여러 브랜치에서 하나의 파일을 수정하더라도 수정된 위치가 겹치지 않으면
* 하나의 파일로 자동머지됨. (3-way)