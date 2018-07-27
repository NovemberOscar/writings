# GraphQL과 인증

GraphQL은 플랫폼 독립적이고 기존의 REST API와는 완전히 다르기 때문에 GraphQL에서의 인증은 큰 문제라고 생각할수도 있지만 인증은 그렇게 어렵지 않다.

토큰을 전달하는 위치가 헤더에서 쿼리나 뮤테이션의 인자로 바뀌었을 뿐이다. 우리가 GraphQL 서버를 만들떄 사용하는 Flask와 Graphene에서 인증을 처리하는 것도 기존 Flask에서 Flask-JWT-Extended나 Flask-JWT-Simple을 사용했던 것처럼 [Flask-GraphQL-Auth](https://github.com/devArtoria/flask-graphql-auth)를 사용하면 그저 쿼리 리솔버나 뮤테이션에 인증 데코레이터를 적용해 인증을 간편하게 처리할 수 있다.

물론 graphene과 Flask로 만든 GraphQL 서버를 HTTP에서만 동작시킨다면 Flask의 리퀘스트 객체를 그대로 활용할 수 있기 때문에 헤더로 토큰을 전달하여 인증을 처리하는것이 가능하다. 그러나 이것이 GraphQL 한 방법이 아니라는것을 염두하자.

그렇다면 Flask-GraphQL-Auth의 작동 방식을 조금 알아보며 인증을 어떻게 처리하는지 알아보자.

```python
def jwt_required(fn):
    """
    A decorator to protect a resolver and mutation.

    If you decorate an resolver or mutation with this, it will ensure that the requester
    has a valid access token before allowing the resolver or mutation to be called. This
    does not check the freshness of the access token.
    """
    @wraps(fn)
    def wrapper(*args, **kwargs):
        token = kwargs.pop(current_app.config['JWT_TOKEN_ARGUMENT_NAME'])  # token을 인자에서 pop해서 가져온다
        try:
            verify_jwt_in_argument(token)  # 토큰 검증 및 ctx_stack에 유저 정보 등록
        except jwt.ExpiredSignatureError:
            return GraphQLError(str(RevokedTokenError()))
        except Exception as e:
            return GraphQLError(str(e))

        return fn(*args, **kwargs)
    return wrapper
```

GraphQL 서버가 요청을 받으면 클라이언트의 요청에 따라 해당하는 쿼리 리솔버와 뮤테이션을 실행시킨다. 여기서 인자는 kwargs로 받을 수 있기 때문에 쿼리 리솔버와 뮤테이션에 데코레이터를 사용해 이 인자들을 사용할 수 있다.  
Flask-GraphQL-Auth의 jwt_required나 jwt_refresh_token_required 데코레이터는 이를 이용해 쿼리 리솔버와 뮤테이션이 실행되기 전에 kwargs로 인자를 받아 유효한 요청인지 검사하는 일종의 미들웨어로 동작한다.   
요청이 유효하다면 context에 요청한 사용자를 등록하여 쿼리 리솔버나 뮤테이션이 사용할 수 있도록 하고 쿼리 리솔버나 뮤테이션은 원본 토큰을 넘겨받지 않게 한다.  
따라서 쿼리 리솔버나 뮤테이션은 기존의 Flask에서 사용했던 것처럼 get_raw_jwt, get_jwt_identity와 같은 컨텍스트 메소드를 사용해 요청의 주체를 식별할 수 있게 된다.

다만 현재 Flask-GraphQL-Auth에서 에러가 발생할 시 나오는 GraphQLLocatedError가 Flask의 errorhandler로 핸들링이 되지 않는 문제가 있어 토큰에 문제가 있으면 exception이 그대로 노출되는 문제가 있다. Flask-GraphQL-Auth는 이 에러 핸들링 문제만 해결이 된다면 1.0 릴리즈를 할 예정이다. 

사용자 클레임을 비롯해 좀 더 Flask-GraphQL-Auth에 대해 알아보고 싶다면 온라인 도큐멘테이션을 찾아보자. 
https://flask-graphql-auth.readthedocs.io/en/latest/

PS) **혹시 해결할 수 있는 아이디어가 있으신 분은 이슈나 PR을 부탁드립니다!**