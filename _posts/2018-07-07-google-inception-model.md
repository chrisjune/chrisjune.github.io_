---
layout: post-sidebar
date: 2018-07-07
title: "구글 이미지 학습(Inception)"
categories: coding
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : true
read_time : 30
feature_image: feature-san-fran
show_related_posts: false
square_related: recommend-spain
---
Google V3 Inception 모델을 활용한 이미지학습에 대한 소개

## Classify 하려는 대상별로 폴더생성
* root 폴더 / 모델명 폴더 / 카테고리별 폴더 별로 생성한다
* 예) /workspace/animal/cat, /workspace/animal/dog, workspace/animal/tiger
* 최소한 35개 이상의 이미지필요  

## Inception model 다운로드
```shell
curl -LO https://raw.githubusercontent.com/golbin/TensorFlow-Tutorials/master/11%20-%20Inception/retrain.py
```   

## 인셉션 모델을 활용한 이미지 트레이닝
* 모델 학습시키기 위한 명령어
```command
 python retrain.py \    
--bottleneck_dir=./[root폴더]/bottlenecks \
--model_dir=./[root폴더]/inception \
--output_graph=./[root폴더]/[모델폴더명].pb \
--output_labels=./[root폴더]/[모델폴더명].txt \
--image_dir .[root폴더]/[모델폴더명] \
--how_many_training_steps [학습횟수]
```
* example
```
python retrain.py \    
--bottleneck_dir=./workspace/bottlenecks \
--model_dir=./workspace/inception \
--output_graph=./workspace/animal.pb \
--output_labels=./workspace/animal.txt \
--image_dir ./workspace/animal \
--how_many_training_steps 1000
```   

## 모델평가
1. 모델 평가 프로그램 다운로드
```shell
curl -LO https://raw.githubusercontent.com/golbin/TensorFlow-Tutorials/master/11%20-%20Inception/predict.py
```
2. 파일내 경로 수정  
python predict.py [파일명]
3. matplotlib backend 에러 처리  
http://dukwon.tistory.com/30
