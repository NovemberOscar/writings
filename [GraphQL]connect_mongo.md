# Connect MongoDB

우리가 만든 GraphQL 서버에 DB를 연결해 보자. 기본적으로 웹 서버는 클라이언트의 요청을 따라 DB를 조회하거나 삭제, 수정, 추가를 한다. 우리의 GraphQL 서버도 쿼리가 들어오면 쿼리에 맞추어 DB를 조회하고, 뮤테이션 요청이 들어오면 DB에 항목을 추가, 수정하거나 삭제한다. 

graphene에 쿼리 요청이 들어온다면 graphene은 그 요청을 받아서 resolver에서 요청된 조건대로 DB에 조회를 하고 그 결과물을 스키마에서 반횐하기로 정의된 필드에 바인딩해 클라이언트가 요청한 방식대로 전달한다.

따라서 우리가 resolver를 작성하기 위해서는 MongoDB ORM인 Mongoengine의 도큐멘트를 GraphQL 필드에 바인딩하는 유틸리티 함수인 constructor와 DB 조회에 사용할 인자 필터링 함수 argument_filter가 필요하다. 물론 이것보다 더 나은 방법이 있을수도 있다.

먼저 constructor를 만들어보자. constructor는 Graphene 필드 클래스와 mongoengine 도큐먼트 타입을 받아서 Graphene 필드 클래스를 반환할 값으로 초기화 해 반환할 필드를 만들어 준다. 코드는 다음과 같다. 여기서 주의할 점은 만약 Mongoengine 도큐먼트에 `id`라는 칼럼이 있을 경우 MongoDB의 `_id`칼럼과 동일하기 때문에 Graphene 필드에 kwargs를 이용해 넣는 값을 `_id`가 아니라 `id`로 넣어주어야 한다는 것이다.

```python
def construct(graphene_field, mongo_obj):
    field_names = list(graphene_field._meta.fields)

    if 'id' in field_names:
        field_names.append('_id')  # id라는 칼럼이 Graphene 필드에 있을 경우 _id 란 항목 추가

    kwargs = {attr: val for attr, val in mongo_obj.to_mongo().items() if attr in field_names}  # id 대신 _id와 다른 칼럼들을 키로 가지는 딕셔너리에 Mongoengine 도큐먼트 바인딩

    if '_id' in kwargs:
        kwargs['id'] = str(kwargs.pop('_id'))  # kwargs에 _id 가 있을 경우 키 이름을 id로 변경

    return graphene_field(**kwargs)
```

다음으로는 argument_filter를 알아보자. argument filter를 쓰는 이유는 Graphene의 작동방식과 관련되어 있다. 간단한 query 클래스를 만들어 보면 다음과 같다.
```python
class Query(graphene.ObjectType):
    id = graphene.String(of_type=graphene.String(),
                            name=graphene.String(),
                            age=graphene.Int())

    # 리솔버
    @staticmethod
    def resolve_id(root, info, name, age):
        return User.objects(name=name,
                            age=age).first()["id"]
```
아래쪽의 리솔버에 집중해 보자. 파이썬은 함수의 인자를 딕셔너리로 전달할 수 있는 **kwargs란 문법을 제공한다. 따라서 위의 코드를 다음과 같이 바꿀 수 있다.
```python
    def resolve_id(root, info, **kwargs):
        return User.objects(**kwargs).first()["id"]
```
그런데 이 코드를 실행해 보면서 name이나 age 조건을 모두 주지 않으면 인자가 채워지지 않았다고 오류가 뜰 것이다. 이를 막기 위해 기본값을 None으로 채우고 kwargs로 전달된 값들을 체크해 None인 것은 제외하여 DB를 조회하는 형식을 사용할 수 있다.
이를 코드로 구현하면 다음과 같다.

```python
def argument_filter(kwargs):
    query = {}
    for key, value in kwargs.items():  # 전달받은 인자와 그 값
        if value is not None:  # None 이 아니여야한다.
            query.update({key: value})

    return query

class Query(graphene.ObjectType):
    id = graphene.String(of_type=graphene.String(),
                            name=graphene.String(),
                            age=graphene.Int())

    @staticmethod
    def resolve_id(root, info, **kwargs):
        return User.objects(**argument_filter(kwargs)).first()["id"]
```
이제 원하는 인자만 이용해서 쿼리가 가능하다. 

다음 포스트에서는 GraphQL에서의 인증에 대해서 알아보자.