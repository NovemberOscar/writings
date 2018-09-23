# 플라스크 이펙티브하게 사용하기

## Why Flask is cooooool?
우리가 사용하는 카카오톡, 페이스북, 구글 등의 앞면은 안드로이드 개발자, 프론트엔드 개발자가 만들어 간다. 
그리고 서비스를 사용하면서 우리가 입력한 데이터나 행동은 서버에 저장되고 이 서버는 백앤드 개발자가 만든다. 
백앤드 개발을 하며 로직을 만드는 것은 당연히 중요하다. 그러나 로직을 짜는 것 말고도 백앤드 개발은 비용과 효율 사이에서 최선의 성능을 내도록 하면서 트래픽과 사용할 프레임워크 등 다양한 요소들을 고려해야 한다. 그런데 백앤드 개발을 처음 하게 되면 수많은 선택지를 보고 고민하게 된다. 
장고와 같은 풀스택 프레임워크는 많은 부분을 자동으로 구성해 주기 때문에 풀스택 프레임워크를 추천하기도 하지만 오히려 이 요소가 초보자들에게는 이해할 수 없는 블랙박스와 같은 점이 될 수도 있다. 그렇다면 플라스크를 사용해 보는 것은 어떨까? 플라스크는 마이크로 프레임워크다. 정말 필수적인 기능만을 제공하면서 자유로운 커스터마이징이 가능해 개발자가 직접 시스템의 구조를 구성하면서 시스템의 구성에 대해 많은 궁금점을 가질 수 있게 하고 다양한 구조를 만들수 있게 한다. 

플라스크를 선택했다면 구조를 만들어 가며 많은 궁금점이 생길 것이다. 이 글은 플라스크 프로그래머들이 좀더 나은 시스템을 구성하는것에 대한 도움이 되기를 바라며 작성했다.

## BETTER WAY 01: 플러거블 뷰와 블루프린트를 사용하자
플라스크로 서버를 짠다고 하면 보통 다음의 코드를 생각하기 마련이다. 

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def main():
    return "Hello World!"
```

우리는 플라스크 어플리케이션을 만든다고 하면 route 데코레이터를 사용하는 함수 기반의 코드를 한 파일에 작성하는 것을 생각한다. 

플라스크 어플리케이션을 어느정도 작성해 보았다면 알겠지만 이런 식으로 어플리케이션을 작성하다 보면 한 파일에 모든 코드가 집중되며 끝없이 길어지는 것을 보게 될 것이다. 이렇게 어플리케이션을 설계한다면 유지보수 및 효율적인 개발이 힘들어져 대용량 어플리케이션으로 확장하기가 매우 힘들어진다.

이러한 문제를 해결하기 위해 플라스크에서는 블루프린트와 플러거블 뷰라는 것을 제공한다.
플라스크 앱 객체와 유사하게 사용되는 블루프린트는 어플리케이션을 완성하기 위한 일종의 "빵틀"로 미리 만들어 놓은 부분을 어플리케이션에 추가하는 방식으로 사용되어  대용량 어플리케이션의 개발 및 유지를 크게 단순화 할 수 있다.
플러거블 뷰는 `MethodView`라는 것을 제공하여  HTTP 메소드별로 로직을 작성하는 것을 편리하게 하여 주면서 함수 기반이 아닌 클래스 기반으로 코드를 작성하게 해 주기 때문에 코드의 재사용성을 높일 수 있다.
또한 블루프린트는 API를 버저닝할때 사용하여 API 버전 단위로 앱을 분할하면  API의 버전 관리를 효율적으로 할 수 있게 해 준다.

다음은 플러거블 뷰와 블루프린트를 조합한 단순한 예시이다.

먼저 플러거블 뷰를 사용한 블루프린트 모듈을 하나 만들자.

```py
from flask import Blueprint
from flask.views import MethodView

bp = Blueprint("test", __name__)  # 플라스크 앱 객체 대신 블루프린트 객체

class Test(MethodView):
    def get(self):
        return "Hello World!"

bp.add_url_rule('/', view_func=Test.as_view("TestAPI"))  # 뷰 클래스 등록

```

그 다음 모듈에 있는 블루프린트를 앱에 등록하여 앱을 실행하는 메인을 작성하여 보자.

```py
from flask import Flask
import .test as test

app = Flask(__name__)

app.register_blueprint(test.bp)  # 블루프린트 등록
```

지금은 오히려 거추장스러워 보이지만 좀 더 큰 어플리케이션에서 다음과 같은 구조를 사용해 모듈화한다면 개발과 유지에 큰 도움이 될 수 있다.
또한 뒤에서 설명할 애플리케이션 팩토리와 같이 사용할때 더 큰 효과를 발휘한다.

플러거블 뷰는 `MethodView`를 상속받는 형식으로 뷰를 구성하게 된다. 이를 이용하여 베이스 뷰 클래스를 만들어 두면 뷰를 만들때 코드를 재사용하기 쉬워진다. 예를 들어 기본 JSON 인코더 대신 유니코드를 지원하는 인코더 함수를 작성하는 등 공통적인 코드를 함수로 만들어 `MethodView`를 사용하여 만든 베이스 뷰 클래스에 만들어 두고 각각의 뷰를 만들때마다 베이스 뷰를 상속받아 작성하면 중복 코드를 크게 줄일 수 있다.

다음은 `MethodView`를 상속받아 유니코드 인코딩용 함수를 포함한 베이스 뷰 클래스를 만들어 활용하는 예시이다. 베이스 뷰에 공통적인 코드를 작성하여 각각의 뷰 클래스는 코드가 줄어드는것을 알 수 있다. 

```py
from flak import Flask, Response
from flask.view import MethodView
import ujson

app = Flask(__name__)

class BaseView(MethodView):  # 공통 코드를 담는 베이스 뷰
    @classmethod
    def jsonify_response(status, data): 
        data = ujson.dumps(data)

        response = Response(data, status, content_type="application/json; charset=utf8")

        return response

class Test(BaseView):  # 베이스 뷰를 상속받음
    def get(self):
        return self.jsonify_response(200, {"Hello": "World"})

app.add_url_rule('/', Test.as_view("test"))
```

**핵심 정리**
1. 플러거블 뷰를 사용하면 코드를 클래스화 하며 중복 코드 제거가 가능하다.
2. 블루프린트는 코드 모듈화 및 api 버전 관리를 위해 사용하자.

## BETTER WAY 02: 애플리케이션 팩토리와 설정 객체를 사용하자
앞서 블루프린트를 소개할 때 애플리케이션 팩토리란것을 잠깐 언급했다. 애플리케이션 팩토리가 무엇일까? 말 그대로 플라스크 앱 객체를 만들어 내는 일종의 공장(팩토리)이다. 애플리케이션 팩토리 메소드는 플라스크의 기본 앱 객체를 입맛대로 뜯어 고치면서 그 안에서 확장 프로그램을 초기화하고, 블루프린트를 조합해 애플리케이션 인스턴스를 만들어 내게 되며 리퀘스트 컨텍스트를 관리하기 위한 리퀘스트 콜백들과 에러 처리를 위한 에러핸들러를 등록하여 애플리케이션의 완성도를 높일 수 있다.

다음은 애플리케이션 팩토리의 간단한 예제 코드이다. 이 코드에서 애플리케이션 팩토리에서 블루프린트를 등록하고 설정값을 설정 객체를 읽어서 등록하며 에러핸들러와 리퀘스트 콜백을 모두 적용하는 예시를 볼 수 있다.

```py
from flask import Flask
from flask_jwt_extended import JWTManager
from .test import bp
from .handler import not_found_error_handle  # 커스텀 에러핸들러
from .util import logger # 리퀘스트 로깅 콜백

def create_app():
  app = Flask(__name__)

  JWTManager.init_app(app)
  
  app.register_blueprint(bp)

  app.register_errorhandler(404, not_found_error_handle)
  app.after_request(logger)

  return app
```

플라스크는 여러가지 방법으로 빌트인 설정과 확장 설정을 구성할 수 있다. 

**설정 구성방법**
1. 딕셔너리 타입인 app.config 객체에 바로 하드코딩하기
2. 파일에서 불러오기
3. 객체에서 불러오기(클래스 사용)

애플리케이션 팩토리의 진정한 장점은 어플리케이션 설정을 구성하려고 사용할때 알 수 있다. 지금 만든 팩토리 메소드는 설정 관련 부분이 없다. 이제 설정을 객체에서 불러오게 만들어 보자. 설정 객체를 어플리케이션 팩토리 메소드에 인자로 주어 어플리케이션과 확장의 설정을 관리하면 테스트 할 때는 테스트시 사용하는 테스트용 설정, 배포에는 배포용 설정을 사용하여 설정을 일일이 하드코딩하거나 커맨드라인 인자로 주는것 대신 객체로 관리하여 편하게 사용할 수 있을것이고 클래스의 성질 중 하나인 상속을 이용해 다양한 설정의 공통 설정 또한 편하게 관리할 수 있다.

다음은 설정 객체에서 설정을 로딩하는 팩토리 메소드 코드이다.

```py

def create_app(config_obj):
  app = Flask(__name__)

  app.config.from_object(config_obj)  # 설정 로딩 부분

    ...

  return app
```

설정은 다음과 같이 만들어 두면 된다.
```py
class Config:
    SERVICE_NAME = 'M A S O'
    SERVICE_NAME_UPPER = SERVICE_NAME.upper()
    REPRESENTATIVE_HOST = None

class DevConfig(Config):  # Config를 상속받는다
    HOST = 'localhost'
    PORT = 5000
    DEBUG = True

    RUN_SETTING = dict(Config.RUN_SETTING, **{
        'host': HOST,
        'port': PORT,
        'debug': DEBUG
    })
```
다음의 코드로 앱을 실행할 수 있다.
```py
if __name__ == "__main__":
    app = create_app(DevConfig)  # 설정 주입

    app.run(**app.config["RUN_SETTING"])
```

**핵심 정리**
1. 팩토리 패턴을 사용하면 다음과 같은 이점을 가진다.
    1. 설정값 관리
    2. 블루프린트 관리
    3. 에러핸들러와 리퀘스트 콜백 관리
    4. 확장 관리
2. 설정은 객체로 로딩하며 여러가지의 설정이 있다면 공통되는 설정은 부모 클래스를 만들어 상속받자.

## BETTER WAY 03: 뷰 데코레이터를 활용하자 
플라스크로 서버를 짜게 되면 항상 중복이 많지만 뷰 로직 실행 전에 실행되어야 하는 코드가 많아서 중복 코드를 줄이려고 해도 `MethodView`를 사용한 베이스 뷰의 한 부분으로 만들기는 힘들다. 이를 위해 파이썬은 데코레이터라는 강력한 기능을 지원한다 그 중 functools.wraps를 적용한 데코레이터로 함수를 데코레이트하면 로직을 따로 변경하지 않고도 함수에 추가적인 기능을 추가하기 쉬워지고 이를 이용해 플라스크의 뷰 펑션 또는 메소드를 위한 뷰 데코레이터들을 사용해 뷰 펑션에 추가적인 검사 또는 기능을 제공할 수 있다. 또한 많은 확장들도 뷰 데코레이터 형식으로 API를 제공한다.

리퀘스트 검증, 헤더 검사, 캐싱 적용 등은 뷰 데코레이터로 구현하자.

다음은 뷰 데코레이터의 예시 중 리퀘스트 유효성 검사를 구현한 예시이다.

```py
from functools import wraps
from flask import g, request, redirect, url_for, Flask, Response

def body_required(f):  # 페이로드가 존재하는지 체크
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if reques.json is None:
            return Response(status_code=401)
        return f(*args, **kwargs)
    return decorated_function

app = Flask(__name__)

@app.route('/secret_page')
@body_required  # secret_page에 엑세스 하기 전 체크하게 됨
def secret_page():
    return "Secret Info"
```

이와 같이 뷰 데코레이터를 사용하여 로직 부분의 코드를 변경하지 않고도 기능을 추가할수 있는것을 볼 수 있다.

**핵심 정리**
1. 메인 로직 전에 실행되며 중복되고, 선택적인 로직중 일부를 뷰 데코레이터로 만들어 메인 로직을 수정하지 않으면서 기능을 추가하자
2. 뷰 데코레이터는 다음과 같은 동작을 위해 사용하자
    1. 리퀘스트 유효성 검사(시간당 요청수 제한, 형식 검사 등)
    2. 인증 검사
    3. 캐싱
    4. 리스폰스 압축


## BETTER WAY 04: 에러핸들러를 사용하자
플라스크는 리퀘스트 처리 과정에 일어난 예외나, 특정한 HTTP 상태 코드에 대해 에러 핸들러를 등록할 수 있다. 

많은 경우 특정한 HTTP 상태 코드에 대해 간단하게 사용자 정의 에러 메시지를 처리하는데만 사용한다. 그런데 에러핸들러는 예외에 대해서도 처리가 가능하다는 것을 기억하자. 기본적으로 플라스크는 특정한 HTTP 상태 코드와 매칭되지 않는 예외에 대해서는 500 Internal Server Error를 반환한다. 
이에 대해 에러핸들러를 적용하여 클라이언트에게 좀 더 자세한 정보를 제공할 수 있거나 숨길 수 있는 사용자 정의 에러 메시지를 전달할 수 있다.

뷰 데코레이터와 리퀘스트 콜백같이 메인 로직 외부에 정의된 코드나 메인 로직에서 예외가 발생할 경우에도 try-except 문 안에서 상태 코드를 직접 반환하는 대신 예외를 발생시키고 앱 또는 블루프린트에 에러핸들러를 등록하면 특정한 오류에 대해 상태 코드 또는 처리 로작을 일괄적으로 적용할 수 있다.

예를 들면 다음과 같은 코드에서는 현재 리퀘스트 페이로드에서 name이란 필드와 description 이라는 필드가 필요하다. 만약 요청을 할때 필드가 없다면 KeyError를 일으킬 것이고, Flask는 상태 코드로 500을 반환할 것이다. 

```py
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/', method=['POST'])
def test():
    name = request.json['name']
    desc = request.json['description']

    return jsonify(dict(name=name, description=desc))
```
그러나 이같은 잘못된 요청에 대해선 HTTP 코드로 400 Bad request를 반환해야 함으로 ValidationError라는 사용자 예외를 만들고 로그 출력 기능을 제공하는 에러핸들러를 사용해 코드를 구성해 보자.

```py
from flask import Flask, request, jsonify, Response

app = Flask(__name__)

class ValidationeError(Exception):  # 사용자 에러
    pass

@app.errorhandler(ValidationError)  # 에러핸들러 등록
def valid_handle(e):
    print('[ERR] Bad Request from {}'.format(reques.remote_addr))
    return Response(data=str(e), status_code=400)

@app.route('/', method=['POST'])
def test():
    payload = request.json
    name = payload.get('name', None)
    desc = payload.get('description', None)

    if (name is None) and (desc is None):
        raise ValidationError

    return jsonify(dict(name=name, description=desc))
```
아까전보단 복잡해 졌지만 메인 로직에서는 예외를 발생시키기만 하면 에러핸들러에서 응답을 400으로 반환하며 로그를 찍는 것을 볼수 있다.

데코레이터에서 발생할 예외도 위와 같이 처리하자. 예를 들어 유효한 JWT 토큰을 받았는지 검사하는 token_required 데코레이터가 있을 때 토큰으로 사용자 인증이 실패하면 발생시킬 UserNorfoundError를 만들고 그에 대한 에러 핸들러를 등록하면 비슷한 데코레이터, 예를 들어 특정 권한을 가졌는지 검사하는 데코레이터 등을 만들 때 일관적인 에러 관리가 가능하다.

추가로 에러핸들러는 블루프린트 별로 적용 가능하기 때문에 블루프린트마다 다른 에러핸들러를 사용할 수 있다. 등록방법은 전역 앱 객체에 등록할때와 매우 유사하다. 
```py
@app.error_handler
```
app에서 등록하던 것을 블루프린트 객체에서 등록하려면 다음과 같이 변경하자.
```py
@<blueprint name>.error_handler
```
또는 
```py
<blueprint_name>.register_error_handler(error, errorhandler)
```

**핵심 정리**
1. 에러핸들러로 반환할 에러 메시지를 사용자화하자
2. 예외 처리 없이는 500을 반환할 에러는 바로 상태코드를 주는 것 대신 예외를 만들고 에러핸들러로 처리하자
3. 경우에 따라 다른 에러핸들링이 필요할 경우 블루프린트 별로 에러핸들러를 등록하자

## BETTER WAY 05: 리퀘스트 훅을 잘 사용하자
<!-- 요청 컨텍스트랑 컨택스트 스택 그리구 콜백같은거 비롯해서 컨택스트 설명 --> 
플라스크에는 리퀘스트가 들어오고 리스폰스를 보낼 때까지 존재하는 리퀘스트 컨텍스트가 존재한다. 그리고 이 리퀘스트 컨택스트를 관리하기 위해 리퀘스트 훅이란 것을 이용해 리퀘스트 전후에 코드를 실행할 수 있다. 

리퀘스트 훅은 4개가 존재하는데 이 두개의 리퀘스트 훅은 리퀘스트 디스패치 전후에 실행되어 리퀘스트 로그 남기기, 리스폰스 헤더 변경 등등 리스폰스 및 리퀘스트 객체를 이용한 코드를 실행할 수 있다. 
- before_request: 리퀘스트 처리 전에 실행되는 함수를 등록하는 데코레이터다. 리퀘스트 처리 전에 리퀘스트 객체를 이용해야 할 경우 실행될 코드를 이 데코레이터를 사용해 등록할 수 있다. 
- after_request: 리퀘스트 처리 후에 실행되는 함수를 등록한다. 리퀘스트 후에 리스폰스 객체를 이용해야 할 경우 실행될 코드를 이 데코레이터를 사용해 등록할 수 있다. 그러나 에러가 발생하여 정상적으로 종료되지 않을경우 실행이 보장되지 않는다.

리퀘스트 훅 중에는 after_request와는 다르게  teardown_request란 것이 존재하는데 이 데코레이터는 리퀘스트 컨택스트가 종료되면서 실행되기 때문에 데이터베이스 커넥션 종료나 에러 로깅 처럼 예외가 발생하는것과 상관 없이 꼭 실행되어야 하는 코드들을 실행하게 된다. 

```py
from flask import *

app = Flask(__name__)

@app.route('/')
def test():
    raise Exception

@app.after_request
def aft(r):
    print("AFTER REQUEST")  # 실행되지 않음
    return r

@app.teardown_request
def tdn(r):
    print("TEAR DOWN")  # 에러가 발생해도 실행됨. 여기서 r은 Exception 객체
```

마지막으로 after_this_request라는 특별한 훅이 존재한다. 모든 리퀘스트 컨텍스트에서 사용되는 after_request 와 달리 이 훅은 현재 진행되는 리퀘스트 컨택스트에서만 실행되는 함수를 등록한다. 데코레이터에서 after_this_request 훅을 사용하면 뷰 펑션 실행 후에 사용되는 코드를 삽입할 수 있다. 

다음은 리스폰스를 gzip알고리즘으로 압축하는 after_this_request 등록 데코레이터의 예시이다.

```py
from functools import wraps
from flask import g, request, redirect, url_for, Flask, after_this_request, Response
import gzip

def gzipper(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        @after_this_request  # after_this_request 콜백 등록
        def zipper(response):
            response.data = gzip.compress(data=response.data)
            return response
        return fn(*args, **kwargs)
    return wrapper


app = Flask(__name__)

@app.route('/')
@gzipeer
def secret_page():
    return Response("This will be compressed")
```

**핵심 정리**
1. before_request 훅을 이용해 미들웨어를 구현하자.
2. after_request로 헤더 변경과 같은 후처리 기능을 추가하자.
3. teardown_request로  
4. after_this_request와 데코레이터를 조합해 현재 진행되는 리퀘스트 컨택스트에서만 실행될 코드를 데코레이터를 이용해 추가하자.

## BETTER WAY 06: TDD를 도입하자
TDD, Test Driven Development는 반복적인 테스팅을 이용한 소프트웨어 개발 기법이다. 먼저 작은 테스트들을 만들고, 그 테스트들을 통과하는 코드를 만들고 개선하는 방식으로 개발을 진행하면서 코드를 개선해 나가데 된다. 테스트를 먼저 작성한 후 코드를 짜게 되면 깔끔한 코드를 작성할 수 있게 된다. 

**TDD with 백앤드**
1. URI와 리퀘스트 리스폰스를 포함한 API 명세를 작성
2. 명세를 통해 각 URI에 대해 테스트 케이스를 작성
3. 더미 데이터를 보내는 코드를 작성
4. 실제 코드를 작성

파이썬의 테스트 라이브러리로 unittest와 pytest가 존재하는데 여기선 unittest 버전으로 설명한다.

유닛테스트는 TestCase를 상속받아 만든 클래스에 셋업 코드와 테스트 코드, 티어다운 코드를 작성하여 테스트를 구성한다.
```py
from unittest import TestCase as TC

class TestCase(TC):
    def setUp(self):
        <set up code>
    def test_something(self):
        <test code>
    def tearDown(self):
        <teardown code>
```
플라스크 웹서버를 테스팅 할 때는 엔드포인트별로 요청을 날려 응답을 검증하는데 이를 위해 플라스크 앱 객체에서 제공하는 테스트 클라이언트를 사용하도록 하자.

```py
tester = app.test_client()
tester.get('/')
```

셋업이나 티어다운, 더미 데이터 삽입 등 기본적인 테스트 환경 코드는 중복이 많이 발생하기 때문에 기본적인 코드를 묶어서 베이스 테스트케이스를 만들고 베이스 테스트 케이스를 이용해 각 uri 별 테스트를 진행하자

```py
from datetime import datetime
from unittest import TestCase as TC
from app import create_app
from config.test import TestConfig

class TCBase(TC):
    def __init__(self, *args, **kwargs):
        self.app = create_app(TestConfig)
        self.client = self.app.test_client()

        ...  # 데이터베이스 연결 등 기반 설정

    def _create_fake_data(self):
        pass  #  DB에 더미 데이터를 삽입

    def _generate_tokens(self):
        pass  # JWT 토큰 생성

    def setUp(self):
        pass  # 공통 셋업 코드 ex) 토큰 가져오기

    def tearDown(self):
        pass  # 공통 티어다운 코드 ex) DB 테이블 드롭

    def request(self, method: function, target_url_rule, token=None, *args, **kwargs):
        pass  # 리퀘스트 헬퍼 메소드 ex) 토큰을 헤더에 자동 삽입
```

**핵심 정리**
1. 플라스크를 테스트할때 제공하는 테스트 클라이언트를 사용하자.
2. 테스트 셋업 코드를 묶어서 베이스 테스트케이스를 만들자.
3. 각 uri 별로 리퀘스트와 리스폰스를 체크하여 테스트를 수행하자.
 
## BETTER WAY 07: CI 서비스와 깃허브를 사용하자
CI란 지속적 통합의 약자로 여러명의 개발자들이 작성한 코드를 지속적으로 통합하고 관리하여 배포하는 시간을 단축하는 것을 말한다. 그리고 수많은 CI 자동화 도구가 제공되어 자동화된 테스팅과 배포등의 기능을 제공하고 있다. 
최근에는 수많은 개발자나 회사가 소스코드 관리용으로 사용하는 깃허브와 연동하여 수많은 CI 도구와 커버리지 측정 서비스가 제공되어서 깃허브에 push 하면 빌드-테스팅-배포 및 커버리지 측정까지의 CI 환경을 손쉽게 구축할 수 있다.  가장 손쉽게 접할수 있는 서비스는 Travis CI와 coveralls이다.

브랜치 전략을 사용하여 개발 브랜치에서 master 브랜치로 PR을 하려고 할 때 Travis CI가 설정대로 자동으로 테스트를 실행해 아무런 문제가 없는지 검증하고, PR이 머지된 후 master 브랜치에서 CI가 자동으로 테스팅을 하게 되고 아무런 문제가 없다면 헤로쿠 업로드나 도커 이미지를 만드는 등 배포를 수행하고 커버리지를 측정하여 coveralls가 자동으로 커버리지 리포트를 만들게 된다. 

여기서 개발자가 하는 일은 개발 브랜치에 푸시하고, master로 머지시키는 일밖에 존재하지 않는다. 모든 테스트 과정과 배포는 설정 파일에 적힌 대로 CI가 수행하게 된다. 

플라스크에서의 지속적 통합은 블루프린트 단위로 작업이 가능하다는 점을 이용하여 수행할 수 있다. 각각의 개발 브랜치에서 블루프린트를 구성하고 master 브랜치에서 통합하여 테스팅 및 배포를 수행하는 형식으로 진행하자.

**핵심 정리**
1. 깃허브와 연동된 CI 서비스와 커버리지 리포트를 사용하여 소스관리-테스팅-배포-커버리지 리포트를 자동화하자.
2. 블루프린트 단위로 작업을 쪼개서 CI를 수행하자.

## 마지막으로
지금까지 소개한 7가지 방법들은 플라스크 애플리케이션을 만들면서 반복적인 코드를 모듈화하고 유연한 코드를 작성하며 개발 과정을 좀 더 편하게 만드는 것들이다. 그리고 이런 방법들은 모두 프로젝트를 하면서 축적된 경험이다. 
프로젝트를 하는것에 막연한 두려움을 가지지 말고 간단한 아이디어라도 직접 만들어 보자. 또 프로젝트를 진행하면서 어느정도 플라스크에 익숙해 졌다면 REST API 대신 GraphQL을 시도해 보거나 MySQL 대신 MongoDB를 사용해 보거나 Flask와 유사한 경험을 제공하는 비동기 프레임워크 Sanic으로의 마이그레이션 등 새로운 것들을 시도해 보자.  

지면상의 한계로 인해 더 많은 것들을 소개하지 못해 아쉽지만 마지막으로 플라스크 애플리케이션 개발에 도움이 될 수 있는 자료와 익스텐션들을 간단히 소개하고자 합니다. 

- 개발 자료
    - JoMingyu 님의 Flask-Large-Application-Example
    - libsora 님의 Flask-Large-Application-Example
    - Explore Flask
    - Snippets Archive
- 익스텐션
    - Flask-RESTful - 플러거블 뷰 대체
    - flasgger - Swagger API 문서 생성
    - Flask-JWT-Extended - JWT 인증 기능
    - Flask-Validation - 리퀘스트 페이로드 검증 기능

마지막으로 플라스크를 처음 소개해주고 많은 도움을 준 JoMingyu님에게 감사드립니다 :)

### P.S.
이 글은 단순히 일회성으로 끝나는 글이 되지 않고 지속 가능한 콘텐츠가 되기 위해 이 글은 추후 GitHub에 공개되어 플라스크 프로그래머들의 Best Practice들이 담길수 있는 공간이 되고자 합니다.