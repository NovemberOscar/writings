# 왜 GraphQL을 사용해야 할까?

현재 수많은 웹 서비스와 API들은 REST API를 사용하여 클라이언트-서버 간 통신을 수행한다. 

물론 REST API는 좋은 방식이지만 다음과 같은 문제점이 생긴다. 
[그림 1]
첫번쨰, 수많은 양의 엔드포인트들
[그림 2]
두번째, 데이터가 원한것보다 너무 많음
[그림 3]
세번째, 여러가지 데이터를 사용하기 위해 요청을 여러번 해서 데이터를 받아와야 함 (N+1 문제)
[그림 4]
네번쨰, 응답 데이터의 구조를 API 도큐멘트 없이 알 수 없다.

그러나 ✨GraphQL✨이란게 있다!
GraphQL을 간단히 설명해 보자면 클라이언트-서버 간 통신을 위해 기존 REST에서 사용하는 엔드포인트 방식 대신 DB에 날리는 SQL 쿼리처럼 API 서버를 대상으로 하는 쿼리 언어(Application Query Language)를 사용하는 방식이다. 그리고  GraphQL은 위와 같은 REST API 문제를 해결할 수 있다.

[그림 1]
첫번째, 단일 엔드포인트 사용
[그림 2]
두번쨰, 쿼리를 만들어 필요한 데이터만 받아올 수 있음
[그림 3]
세번째, 쿼리를 사용해 단일 요청으로 필요한 데이터를 한번에 받아올 수 있다.
[그림 4]
네번째, 쿼리에 대응되는 데이터만을 보내주기 때문에 항상 응답이 예측 가능함

거기다가 REST 방식과는 다르게 프로토콜 의존적이지 않아 HTTP 프로토콜이 아닌 다른 프로토콜 위에서도 동작할 수 있기도 하다. 

다음 포스팅에서는 GraphQL API를 구성하는 쿼리, 뮤테이션, 스키마 등 GraphQL의 시스템에 대해 알아보고 파이썬으로 간단한 API 서버를 만들어 볼 것이다.
