# 파이썬과 타입 힌트
## 목차
- PEP484에 대한 소개
    - 타입 힌트의 의의와 목표
    - 그러나 파이썬이 정적 타입을 지향하는 것은 아니다.
- 타입 힌트를 표현하는 문법
    - 간단한 타입 표기
    - 타입 별칭
    - 콜백
    - 클래스 타입
    - 제네릭
    - 유니온
    - 오버로딩
    - 전방 참조
    - 제네레이터와 코루틴

## PEP 484에 대한 소개
Python 3.5 버전에는 다음과 같은 형식으로 IDE와 코드 가독성에 도움을 줄 수 있도록 함수의 인자와 반환값에 대한 타입 힌트가 처음으로 도입 되었다. 
```py
def greeting(name: str) -> str:
    return 'Hello ' + name
```
그리고 후에 나온 3.6 버전에서는 인자와 반환값 만이 아니라 변수에도 타입 힌트 표기가 가능해졌다.
```py
def greeting(name: str) -> str:
    s: str = 'Hello ' + name
    return s
```

### 타입 힌트의 의의와 목표
타입 힌트 기능은 타입 표시에 관한 표준 구문을 제공하고, 더 쉬운 정적 분석과 리팩토링 및 타입 정보를 추론하는 것에 대한 도움을 주기 위해 만들어졌다.

예를 들어 예상하지 못한 타입이 변수에 할당될 때나 함수에 전달될 때 IDE나 정적 검사기는 쉽게 오류라고 판단할 수 있을것이다. 또한 다른 사람이나, 쉽게 잊어버릴것 같은 코드에 어떤 타입이 기대되는지 쉽게 알려줄 수 있다.

타입 힌트는 적절한 도구와 함께 사용될 경우 정적 언어가 가지는 장점인 타입 시스템의 견고함을 동적 언어로써 조금이라도 따라잡을 수 있도록 도와줄 것이다.

### 그러나 파이썬이 정적 타입을 지향하는 것은 아니다.
타입 힌트 기능은 말 그대로 타입 "힌트" 기능일 뿐이다. 타입 힌트는 정적 검사기와 IDE를 사용하며 코드의 질을 높이기 위해 사용 될 수 있으나 결코 런타임에 영향을 끼치지 않는다. 정수형을 가질 변수에 문자열 타입을 힌트로 작성해 놓는다고 해서 파이썬은 아무런 문제가 있다고 생각하지 않을 것이다. 

사실 타입 힌트는 코드에 붙은 주석에 가깝다. 독스트링을 __doc__을 사용하여 가져올 수 있는 것 처럼 타입 힌트 정보 또한 __annotations__속성을 통하여 타입 힌트를 가져올 수 있다.

## 타입 힌트를 표현하는 문법
파이썬의 타입 힌트는 typing 모듈을 사용하여 작성할 수 있다. 

### 간단한 타입 표기
먼저 함수 선언부에 관한 타입 힌트는 인수 뒤에 콜론을 붙여서 인수의 타입 힌트를 붙이고, 괄호 뒤 콜론 전에 "-> 타입" 을 붙이는 형식으로 반환값에 대한 타입 힌트를 지정할 수 있다.
```py
def make_post(title: str, author_id: int=0) -> str:
    ...
```
변수의 타입은 함수 인자와 비슷한 형식으로 힌트를 붙일 수 있다.
```py
num: int = 34  # int type
str: string = "Hello types!"  # str type
test: Test = Test(num)  # class "Test" type
```
클래스 멤버 변수도 변수와 비슷하다.
```py
class A:
    x: int  # int type
    y: str  # str type
    z: float  # float type

    def __init__(self, x: int, y: str, z: float):
        self.x = x
        self.y = y
        self.z = z
```
### 특별한 타입들
타이핑 모듈에는 특별한 타입들 몇가지가 존재한다. 
그 중 Any와 NoReturn에 대해 소개한다.

Any는 말 그대로 모든 타입을 허용한다.
```py
x: Any = 3  # 된다
y: Any = "anyone"  # 된다
```
NoReturn은 리턴이 되는것이 아니라 예외를 발생시키는 등의 경우에 사용한다.
```py
from typing import NoReturn

def stop() -> NoReturn:
    raise RuntimeError('no way')
```

### 타입 별칭
타입 별칭은 간단하다. 타입을 새 변수에 대입하면 그 변수는 타입 별칭으로써 기능한다. 다음은 간단한 예시이다.
```py
from typing import List

url_ls = List[str]  # List with str type 
crawling_result = List[str]  # List with str type 

def crawler(urls: url_ls) -> crawling_result:
    ...
```
물론 더욱 복잡한 타입 별칭도 만들 수 있다. 이 코드는 제네릭을 활용한 것으로 추후에 더 알아볼 것이다.
```py
from typing import TypeVar, Tuple, Iterable

T = TypeVar('T', int, float)
Vector = Iterable[Tuple[T, T]]

def inproduct(v: Vector[T]) -> T:  # 벡터의 내적
    return sum(x*y for x, y in v)
```

### 객체로써의 함수의 타입
함수를 작성하다 보면 함수를 반환하는 함수 또는 함수를  함수를 작성하게 될 것이다. 또는 변수에 함수 객체를 할당할 수도 있다. 이러한 객체로써의 함수의 타입은 Callable을 사용하여 표현할 수 있다.

```py
from typing import Callable

def callback_loader(callback: Callable[[float], int]) -> int:
    # float을 인수로 받아 int 형을 반환하는 콜백을 인수로 받는다.
    return callback(3.7)
```
```py
from typing import Callable

def closure(txt: str) -> Callable[[], str]:
    # 아무 인수도 받지 않고 str 형을 반환하는 함수를 반환한다.
    def inner_func() -> str:
        return txt

    return inner_func
```
```py
from typing import Callable

def func(txt: str) -> str:
    return txt

x: Callable[[str], str] = func  # 함수 객체를 할당
```

### 클래스 타입
파이썬의 타입 힌트는 특정한 클래스라는 것 또한 명시할 수 있다.
```py
class A:
    def print_all() -> None:
        ...

def print_class(cls_obj: A) -> None:
    cls_obg.print_all()

a = A()
print_class(a)  # 정적 검사가 통과할 것이다.
```
그런데 만약 상속받은 하위 클래스도 받아들이려면 어떻게 해야 할까? 
```py
class A:
    def print_all() -> None:
        ...

class B(A):
    ...

def print_class(cls_obj: A) -> None:
    cls_obg.print_all()

b = B()
print_class(b)  # 정적 검사기는 타입이 일치하지 않는다고 경고할 것이다.
```
이럴 때를 위해 Type[C] 문법이 준비되어 있다. (C는  클래스를 나타낸다) 그냥 C를 타입으로 사용하면 C의 인스턴스만을 받아들이지만 Type[C]을 사용하면 상위 클래스를 상속받은 모든 하위 클래스 또한 허용하게 된다.
```py
from typing import Type

class A:
    def print_all() -> None:
        ...

class B(A):
    ...

def print_class(cls_obj: Type[A]) -> None:
    # A의 하위 클래스도 허용
    cls_obg.print_all()

b = B()
print_class(b)  # 정적 검사가 통과할 것이다.
```
### 제네릭
데이터 형식에 의존하지 않고 인자, 변수 또는 반환값 등이 여러 다른 데이터 타입들을 가질 수 있는 방식을 제네릭이라고 한다. 

파이썬의 타입 힌트 기능에서도 제네릭 표현이 가능하다.

다음은 간단한 제네릭을 사용한 클래스의 선언과 사용의 예시이다.
```py
from typing import TypeVar, Generic, List

T = TypeVar('T')

class C(Generic[T]):
    def __init__(self) -> None:
        self.ls: List[T] = []
        # T 타입의 리스트를 초기화한다
    
    def put(item: T) -> None:
        # T 타입을 인수로 받는다
        self.ls.append(item)

    def get(index: int) -> T:
        # T 타입을 반환한다
        return self.ls[index]

c = C[str]()  # 타입은  str로 결정된다
c.put("hello")  # 아무런 문제가 없다
c.get(0)  # str 타입의 값을 반환할 것으로 기대된다.
```
함수에도 제네릭을 적용할 수 있다.
```py
from typing import TypeVar, Sequence

T = TypeVar('T')

def first(sqnce: Sequence[T]) -> T:
    # 입력받은 시퀀스 객체의 타입에 따라 반환 타입도 결정된다.
    return sqnce[0]
```

