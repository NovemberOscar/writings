# Python으로 GraphQL API 서버 만들어 보기

## GraphQL 시스템 알아보기
GraphQL 서버를 만들려면 GraphQL API를 구성하는 요소들에 대해 알아볼 필요가 있다. GraphQL API는 다음의 요소로 이루어진다.

- 스키마: DB 스키마가 테이블과 릴레이션을 정의하는 것처럼 GraphQL 스키마는 뮤테이션, 쿼리, 타입과 Enum 등 GraphQL 시스템을 정의한다.
    - 쿼리: CRUD의 Read를 담당하는 부분이다
        - 필드: 쿼리의 속성, DB의 테이블 역할을 수행한다.
    - 뮤테이션: CRUD의 Create, Update, Delete를 담당한다
- 리솔버: 쿼리나 뮤테이션 요청이 들어올 때마다 요청에 맞게 리턴값을 만들어 내는 로직 부분이다.

즉, GraphQL 시스템은 스키마로 시스템의 구조를 정의하고, 리솔버에서 행동을 정의한다. 따라서 쿼리나 뮤테이션에 대해 요청을 하면 리솔버에서 요청을 처리해 응답하는 구조를 가진다.

## 스키마 언어
GraphQL 서비스는 어떤 언어로든 작성할 수 있다. 특정 프로그래밍 언어 에 의존하지 않기 위해 우리는 GraphQL 스키마 언어를 사용할 것이다.    이 스키마 언어는 쿼리 언어와 비슷하며 GraphQL 스키마를 언어에 구애받지 않는 방식으로 표현 할 수 있게 해준다. 

우리는 GraphQL 시스템을 만들 때 먼저 스키마 언어로 표현한 후 코드로 구현할 것이다.

## 프로젝트 셋업
이 포스팅 시리즈에서는 간단한 Hello World 예제로부터 시작해 책을 유저가 검색, 대출하고 반납할 수 있는 도서관 시스템을 만들어 볼 것이다.

우리의 서버를 만들기 위해 파이썬에서 MongoDB와 Flask를 사용할 것이고 몇가지의 라이브러리 설치가 필요하며 파이썬과 Flask에 대한 이해가 어느정도 있다고 가정한 후 진행할 것이다.

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
    - fields.py
    - query.py
    - mutaion.py
    - schema.py
    - server.py
```
이 글에서는 간단한 Hello World 예제를 만들어 보지만 시리즈를 진행해 나가면서 도서관 시스템을 완성할 것이다.

## 스키마 설계
우리가 만들어 볼 서비스의 요청과 응답을 쿼리 언어로 표현하면 다음과 같다.
```
#query
{
    Hello(name: 'Lewis'){
        answer
    }
}
```
```json
{
    "data":{
        "Hello":{
            "answer": "Hello Lewis!"
        }
    }
}
```

## 필드 만들기
쿼리 언어 표현을 실제 코드로 만들기에 앞서 스키마 언어로 나타내어 보자. 우리가 만든 쿼리 예제는 이름 인자를 받고 받은 이름에 대해 Hello를 하는 응답을 한다. 우리가 만들 스키마의 필드는 다음과 같이 표현된다.
```
type Hello {
    answer: String!
}
```

이제 이 필드를 fields.py 에 코드로 나타내어 보자.
```python
import graphene

class HelloField(graphene.ObjectType):
    answer = graphene.String()
```
`fields.py`


## 쿼리 만들기
우리가 만들 쿼리는 스키마 언어로 다음과 같이 표현된다.
```
type Query{
    Hello(name: String): Hello
}
```
GraphQL 시스템이 쿼리를 처리하기 위해서는 인자와 반환 타입을 스키마에 명시해야 한다. 우리가 작성한 스키마 언어대로라면 Hello 쿼리를 할 때 name 인자를 받고 Hello 타입의 결과를 반환 할 것이다.

이를 query.py 에 코드로 표현하면 다음과 같다.

```python
import graphene
from .fields import HelloField

class Query(graphene.ObjectType):
    post = graphene.List(of_type=HelloField,
                         name=graphene.String())
```

이때까지는 쿼리의 구조와 모양을 만들었다. 이제는 리솔버를 만들어 보자.

우리의 시스템은 인자로 이름을 받아서 "Hello + 받은 이름" 을 내놓는다. 이 로직을 리솔버로 만들어서 추가하자.

```python
import graphene
from .fields import HelloField

class Query(graphene.ObjectType):
    Hello = graphene.String(of_type=HelloField,
                            name=graphene.String())

    # 리솔버
    @staticmethod
    def resolve_hello(root, info, name):
        return "Hello " + name
```

## 뮤테이션 만들기
우리의 시스템에 기본값 기능을 추가해 보자.
```python
...
from .fields import HelloField

default_value = "Lewis"

class Query(graphene.ObjectType):
    Hello = graphene.String(of_type=HelloField,
                            name=graphene.String(default_value=default_value))
...
```
간단히 이렇게 바꿔주면 기본값을 "Lewis" 로 지정해 인자로 아무것도 넣지 않았을 경우 "Hello Lewis"를 출력하도록 할 수 있다.

이제 이 기본값을 바꿀 수 있는 뮤테이션을 추가해보자. 만들 뮤테이션을 스키마 언어로 표현하면 다음과 같다.

```
type Mutation {
    setDefault(name: String): String!
}
```
이 뮤테이션을 mutation.py에 코드로 나타내면 다음과 같다.

```python
import graphene
from .query.py import default_value

class setDefaultMutation(graphene.Mutation):
    class Arguments:
        # 인자
        name = graphene.String()

    # 반환값
    message = graphene.String()

    # 리솔버
    @staticmethod
    def mutate(root, info, name):
        default_value = name

        return "Success"

class Mutation(graphene.ObjectType):
    set_default = setDefaultMutation.Field()
        
```

## 스키마 만들기

지금까지 스키마를 구성하는 쿼리와 뮤테이션을 만들었으니 이제 최종 스키마에 이것들을 연결해 보자. 우리의 스키마는 스키마 언어로 다음과 같이 표현된다
```
schema {
    query: Query
    mutation: Mutation
}
```

schema.py에 코드로 나타내 보면 다음과 같다.
```python
import graphene
from flask_graphql import GraphQLView
from .mutation import Mutation
from .query import Query

schema = graphene.Schema(query=Query, mutation=Mutation)
```

이제 이 스키마를 Flask 서버로 서비스하기 위해 app.py에 다음의 코드를 작성한 후 실행하고 localhost:5000에 접속하여 GraphiQL로 우리의 첫 GraphQL API를 만나보자.
```python
from flask import Flask

app = Flask(__name__)

app.add_url_rule('/graphql', view_func=GraphQLView.as_view('graphql', schema=schema, graphiql=True))

if __name__ == "__main__"
    app.run(debug=True)
```

## 테스트하기

다음 포스트에서는 좀더 기능을 추가해서 MongoDB를 연동해볼 것이다.