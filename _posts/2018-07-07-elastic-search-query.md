---
layout: post-sidebar
date: 2018-07-07
title: "Elastic search 쿼리"
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
elastic search의 기본적인 쿼리방법을 알아본다 

## ES에 있는 index들을 가져옴
GET _cat/indices

## match_all: 검색조건없이 모든 내용 검색
* "_source": select에서 보여줄 컬럼값들
```json
GET /dev-product/_search
{
    "query":{"match_all":{}},
    "_source":["sell_price","item_stock_status"]
}
```

## 특정 인덱스에서 검색
* match: where조건과 같이 특정 조건을 넣어 검색
* 검색된 결과중 Item_name만 출력
```json
GET /dev-product/_search
{
  "query": {
    "match": {
      "sell_price": 10000
    }
  },
  "_source": [
    "item_name"
  ]
}
```

## 특정 index에서 쿼리하기.
* bool은 조건을 여러게 넣을 수 있도록 해줌
* must: 모두 만족해야하는 and조건
* should: 조건중 하나만 만족하면 되는 or조건
* 판매가격이 만원또는 5만원인 상품중 카테고리가 가방이 아닌 상품검색
```json
GET dev-product/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "sell_price": 10000
          }
        },
        {
          "match": {
            "sell_price": 50000
          }
        }
      ],
      "must_not": {
        "match": {
          "category_keyword": "가방"
        }
      }
    }
  },
  "_source": [
    "item_name",
    "sell_price"
  ]
}
```

## filter조건에서는 range로 판매가격의 범위를 지정할 수 있음
* 판매가격이 2천원에서 2만원사이인 상품을 조회
```json
GET /dev-product/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "range": {
          "sell_price": {
            "gte": 2000,
            "lte": 20000
          }
        }
      }
    }
  }
}
```