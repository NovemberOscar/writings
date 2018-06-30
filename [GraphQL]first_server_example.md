# Python으로 GraphQL API 서버 만들어 보기

## GraphQL 시스템 알아보기
GraphQL 서버를 만들려면 GraphQL API를 구성하는 요소들에 대해 알아볼 필요가 있다. GraphQL API는 다음의 요소로 이루어진다.

- 스키마: DB 스키마가 테이블과 릴레이션을 정의하는 것처럼 GraphQL 스키마는 뮤테이션, 쿼리, 타입과 Enum 등 GraphQL 시스템을 정의한다.
    - 쿼리: CRUD의 Read를 담당하는 부분이다
        - 필드: 쿼리의 속성, DB의 테이블 역할을 수행한다.
    - 뮤테이션: CRUD의 Create, Update, Delete를 담당한다
- 리솔버: 쿼리나 뮤테이션 요청이 들어올 때마다 요청에 맞게 리턴값을 만들어 내는 로직 부분이다.

즉, GraphQL 시스템은 스키마로 시스템의 구조를 정의하고, 리솔버에서 행동을 정의한다. 따라서 쿼리나 뮤테이션에 대해 요청을 하면 리솔버에서 요청을 처리해 응답하는 구조를 가진다.

## 프로젝트 셋업
이 포스팅 시리즈에서는 간단한 Hello World 예제로부터 시작해 책을 유저가 검색, 대출하고 반납할 수 있는 도서관 시스템을 만들어 볼 것이다.

우리의 서버를 만들기 위해 파이썬에서 MongoDB와 Flask를 사용할 것이고 몇가지의 라이브러리 설치가 필요하며 Flask에 대한 이해가 어느정도 있다고 가정하여 진행할 것이다.

먼저 Python과 가상환경 설치가 다 되었다면 다음의 텍스트를 `requirements.txt`에 복사한 다음 `pip install -r requirements.txt` 명령어를 수행하자.

```
#requirements.txt

aniso8601==3.0.0
click==6.7
Flask==1.0.1
Flask-GraphQL==1.4.1
Flask-GraphQL-Auth==0.3
graphene==2.1
graphql-core==2.0
graphql-relay==0.4.5
itsdangerous==0.24
Jinja2==2.10
MarkupSafe==1.0
mock==2.0.0
mongoengine==0.15.0
mongomock==3.10.0
pbr==4.0.2
promise==2.1
PyJWT==1.6.1
pymongo==3.6.1
Rx==1.6.1
sentinels==1.0.0
six==1.11.0
typing==3.6.4
Werkzeug==0.14.1
```

지금 만들어 볼 Hello World 서버의 구조는 다음과 같다.
```
/
    /app
        /graphql_view
            fields.py
            query.py
            mutaion.py
            __init__.py
        __init__.py
    server.py
```
이 글에서는 간단한 Hello World 예제를 만들어 보지만 시리즈를 진행해 나가면서 도서관 시스템을 완성할 것이다.

## 필드 만들기
쿼리와 뮤테이션을 만들기에 앞서 쿼리와 뮤테이션에서 사용할 필드를 만들어 보자.

## 쿼리 만들기

## 뮤테이션 만들기

## 스키마 만들기

## 테스트하기

다음 포스트에서는 좀더 기능을 추가해서 MongoDB를 연동해볼 것이다.