## 파이썬스럽다(pythonic)의 의미

모든 프로그래밍 언어는 각자의 고유한 관용구(idiom)를 가지고 있다. 여기서 관용구란 특정 작업을 수행하기 위한 특별한 방법이며, 패턴이라고도 불린다. 이런 관용구를 따른 코드를 관용적이라고 부르며, 파이썬에서는 파이썬스럽다고 한다. 이런 관용구를 따른 코드는 더 나은 성능을 낸다.

## 인덱스와 슬라이스

파이썬은 인덱스에 음수를 사용하여 끝에서부터 접근이 가능하다. 또한 슬라이스를 사용하여 특정 구간의 인덱스 요소롤 반환할 수 있다. 마지막 예제를처럼 슬라이스에서 세번째 값을 추가로 입력하여 간격을 설정할 수 있다.

```python
> num = (2,3,4,5,6,7)
> num[-1]
7
> num[1:3]
(3,4)
>num[0:4:2]
(2,4)
```

## 자체 시퀀스 생성

슬라이스를 가져오는 또 다른 방법이 있다. 아래의 예시를 보자.

```python
>interval = slice(0,4,2)
>num[interval]
(2,4)
```

결과는 위에서 직접 슬라이스를 한 결과와 같다. 위 방법은 클래스 내부에 내장되어 있는 `_getitem_`이라는 메서드를 이용한 것이다. 파이썬의 클래스는 기본적으로 내장한 몇 가지의 매서드가 존재한다. 이런 기능들을 **매직 메서드**라고 한다.

**매직 메서드란**  
매직 메서드란 사용자가 클래스안에 정의할 수 있는 스페셜 메서드로서 클래스를 int, str, list등의 파이썬의 빌트인 타입(built-in type)과 같은 작동을 하게 해준다.

```python
class items:
    def __init__(self, *values):
        self._values = list(values)

    def __len__(self):
        return len(self._values)

    def __getitem__(self, item):
        return self._values.__getitem__(item)
```

위의 예시에서는 클래스 `items`에 3개의 매직 메서드를 정의했다. 하나씩 간단하게 기능을 살펴보자.

```python
> exam = items(1,2,3,4,5,6)
> exam._values
[1, 2, 3, 4, 5, 6]
>len(exam)
6
>exam[interval]
[1, 3]
```

- `__init__`: 클래스에 파라미터로 들어오는 값들을 객체로 저장한다. 위의 예시에서 클래스의 인스턴스인 `exam`은 `exam._values`에 리스트화된 `num`이 저장된다.
- `__len__`: `__init__`에 정의된 데이터의 개수를 반환한다. 위의 예시에서는 `exam._values`의 개수를 반환한다.
- `__getitem__`: key에 해당하는 대괄호 안의 값을 파라미터로 전달한다. 위의 예시에서는 `exam[interval]`을 입력하면 `interval`이 `__getitem__`의 파라미터로 전달되어 해당 함수의 반환값을 제공한다.

이외에도 다양한 매직 메서드들이 많이 존재한다. 그렇다면 이런 기능을 왜 사용하는 것일까? 바로 데이터 타입을 맞춰주기 위함이다. 위와 같이 내장 함수나 사칙연산 함수 등을 데이터 타입에 맞게 함수로 지정해두면 편리하게 사용할 수 있다.

## 컨텍스트 관리자

일반적으로 파일을 열면 파일 디스크럽터의 누수를 막기 위해 사용이 끝난 후 파일을 닫아야 한다. (파일 디스크럽터란 간단히 설명하면 데이터의 출입구이다.) 하지만 파일을 닫기 위해서는 할당된 모든 리소스를 해제하여야 하는데 코드가 복잡해지는 경우 모든 경로를 찾아서 해제하기는 어렵다. 이때 컨텍스트 관리자를 사용하면 이런 부분들을 자동으로 관리해준다. 아래 예시를 확인해보자.

```python
with open(filename) as fd:
    process_file(fd)
```

위 코드는 `with`로 컨텍스트 관리자로 진입한 후 `open()`으로 관리자 내부에서 파일을 오픈한 다음 `process_file()`함수에 들어있는 과정을 모두 실행한다. 여기서 예외가 발생해도 해당 함수에 들어있는 코드가 모두 진행되면 자동으로 해당 파일을 닫는다.

컨텍스트 관리자는 `__enter__`와 `__exit__`라는 두 개의 매직 메서드를 가지고 있다. 처음 `with`는 `__enter__`을 호출하여 해당 메서드에서 반환하는 객체를 `as` 다음에 오는 변수에 할당한다. 그후 코드가 끝나면 `__exit__`를 호출하여 해당 컨텍스트 관리자를 종료한다.

앞서 매직 메서드를 구현했던 예제와 같이 컨텍스트 관리자 또한 직접 구현할 수 있다. 바로 `contextlib`모듈을 사용하는 것이다. 아래의 예제를 확인해보자.

```python
import contextlib

@contextlib.contextmanager
def db_handler():
    stop_database()
    yield
    start_database()

with db_handler():
    db_backup
```

위와 같이 컨텍스트 관리자로 설정할 함수 위에 `@contextlib.contextmanager`를 추가하면 된다. 컨텍스트 관리자로 사용될 함수는 **제너레이터**라는 함수의 형태여야 한다.

**제너레이터란**  
제너레이터는 이터레이터를 생성해주는 함수이다. 이터레이터는 클래스에 __iter__, __next__ 또는 __getitem__ 메서드를 구현해야 하지만 제너레이터는 함수 안에서 yield라는 키워드만 사용하면 된다. 그래서 제너레이터는 이터레이터보다 훨씬 간단하게 작성할 수 있다.

`db_handler()`에서 `yield`는 `__enter__`와 `__exit__`를 나눠주는 역할을 한다.

## 프로퍼티, 속성과 객체 메서드의 다른 타입들

다른 언어들과 다르게 파이썬 객체의 모든 프로퍼티와 함수는 public이다. 즉 호출자가 객체의 속성을 호출하지 못하도록 할 방법이 없다는 것이다.

엄격한 강제사항은 없지만 몇 가지 규칙이 있다. 밑줄로 시작하는 속성은 해당 객체에 대해 private을 의미하여, 외부에서 호출되지 않기를 기대하는 것이다.

### 파이썬에서의 밑줄

파이썬에서 밑줄을 사용하는 몇 가지 규칙이 있다. 아래의 예시를 먼저 살펴보자.

```python
class Connector:
    def __init__(self, source):
        self.source = source
        self._timeout = 60

> con = Connector('mysql://localhost')
> con.source
'mysql://localhost'
>con._timeout
60
>con.__dict__
{'source': 'mysql://localhost', '_timeout': 60}
```

위 속성에서 볼 수 있듯이 `source`는 public이고 `_timeout`은 private이다. 이 코드를 해석해보면 `_timeout`은 해당 클라스 자체에서만 사용되고 바깥에서는 호출되지 않아야 한다. 이렇게 내부에서 사용할 것들을 밑줄로 정리해두면 훨씬 안전하게 리팩토링 할 수 있다.

어떤 이들은 밑줄 2개를 이용하면 private 객체를 만들 수 있다고 한다. 하지만 그것은 사실이 아니다. 아래의 예시를 보자.

```python
class Connector:
    def __init__(self, source):
        self.source = source
        self.__timeout = 60

> con = Connector('mysql://localhost')
> con.source
'mysql://localhost'
>con.__timeout
AttributeError: 'Connector' object has no attribute '__timeout'
```

위처럼 밑줄이 두 개인 경우는 메서드를 바로 인식하지 않는다. 하지만 이것은 파이썬이 다른 이름의 메서드를 만든 것이다. 이를 이름 맹글링이라고 한다. 이름 맹글링이란 파이썬이 본인의 규칙에 맞게 알아서 함수 또는 변수의 이름을 변형하는 것을 말한다. 밑줄 2개는 아래와 같은 이름의 메서드를 생성한다.

```python
>con._Connector__timeout
60
```

### 프로퍼티

객체에 저장되는 값이 일반적인 속성을 가져야 할 경우에 프로퍼티를 보편적으로 사용된다.
예를 들어서 사용자가 올바르지 않은 양식의 이메일을 입력하지 않도록 만드는 코드를 짠다고 가정해보자. 그럼 코드는 다음과 같다.

```python
import re

email_format= re.compile(r"[^@]+@[^@]+[^@]+")

def is_valid_email(user_email: str):
    return re.match(email_format,user_email) is not None

class User:
    def __init__(self, username):
        self.username = username
        self._email = None

    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, new_email):
        if not is_valid_email(new_email):
            raise ValueError(f"{new_email}은 유효한 이메일이 아닙니다.")
        self._email = new_email
```

위 코드에서 `@property`는 클래스 User의 private 값인 `_email`을 반환한다. 두번째 메서드인 `@email.setter`는 프로퍼티에 값이 할당되면 해당 값이 올바른 이메일인지 검사한 후 클래스 User의 `_email` 속성을 업데이트 할지 결정한다.

프로퍼티는 명령-쿼리 분리 원칙을 따르기 위한 좋은 방법이다. 명령-쿼리 분리 원칙이란 객체의 메서드가 무언가의 상태를 변경하는 명령이거나 무언가의 값을 반환하는 쿼리이거나 둘 중 하나만 수행해야 한다는 것이다,

프로퍼티를 사용하면 `@property`는 무언가의 응답을 하기 위한 쿼리이고, `@<property_name>.setter`는 무언가를 하기 위한 커맨드(명령)이다.

## 이터러블 객체

파이썬에서는 기본적으로 반복 가능한 객체가 있다. for 루프를 사용하여 객체를 반복할 수도 있지만, 반복을 위한 자체 이터러블을 만들 수도 있다. 엄밀히 말하면 이터러블은 `__iter__` 매직 메서드를 구현한 객체, 이터레이터는 `__next__` 매직 메서드를 구현한 객체를 말한다.

파이썬의 반복은 이터러블 프로토콜이라는 자체 프로토콜을 사용해 동작한다. 객체를 반복할 수 있는지 확인하기 위해서 다음 2가지를 검사한다.

- 객체가 `__next__`나 `__iter__` 중 하나를 포함하는가
- 객체가 시퀀스이고 `__len__`과 `__getitem__`를 모두 가져는가

객체를 반복하려고 하면 파이썬은 해당 객체의 `iter()`를 호출한다. 이것은 해당 객체에 `__iter__` 메서드가 있는지 확인하는 것이다. `__iter__`가 있다면 해당 메서드를 실행한다. 예시를 보자.

```python
from datetime import timedelta, date

class DataRangeContainerIterable:
    def __init__(self,start_date,end_date):
        self.start_date = start_date
        self.end_date = end_date

    def __iter__(self):
        current_day = self.start_date
        while current_day < self.end_date:
            yield current_day
            current_day += timedelta(days=1)
```

위 코드는 `__iter__`에서 제너레이터를 생성하고 제너레이터는 다시 그 안의 `__iter__`를 생성한다. 이런 형태의 객체를 '컨테이너 이터러블'이라고 한다. 예시를 통해 위 코드를 확인해보자.

```python
>temp = DataRangeContainerIterable(date(2020,1,1),date(2020,1,5))
>",".join(map(str,temp))
'2020-01-01,2020-01-02,2020-01-03,2020-01-04'
```

한번에 날짜들이 나오는 것을 볼 수 있다.

그렇다면 만약 객체에 `__iter__`가 없는 경우에는 어떻게 반복해야 할까? 객체는 `__iter__`가 없는 경우에 `__getitem__`을 찾으며, 이것도 없다면 `TypeError`를 일으킨다.

따라서 시퀀스는 기본적으로 `__len__`과 `__getitem__`를 수현하고 첫 번째 인덱스 0부터 시작하여 포함된 요소를 한 번에 하나씩 차례로 가져올 수 있어야 한다. 그렇지 않으면 반복이 작동하지 않는다.

이터러블 방식은 데이터를 하나씩 가져오기 때문에 메모리를 적게 사용하지만 시간복잡도는 O(n)이다. 이와 반대로 시퀀스의 경우 메모리가 많이 사용되지만 시간 복잡도는 O(1)로 상수에 가능하다.

```python
class DateRangeSequence:
    def __init__(self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date
        self._range = self._create_range()

    def _create_range(self):
        days = []
        current_day = self.start_date
        while current_day < self.end_date:
            days.append(current_day)
            current_day += timedelta(days=1)
        return days

    def __getitem__(self, day_no):
        return self._range[day_no]

    def __len__(self):
        return len(self._range)
```

위의 코드는 `_create_range()`을 사용하여 시퀀스를 차례대로 불러온다. 해당 코드가 어떻게 동작하는지 확인해보자.

```python
> temp = DateRangeSequence(date(2020,1,1),date(2020,1,5))
> for i in temp:
    print(i)
2020-01-01
2020-01-02
2020-01-03
2020-01-04
> temp[-1]
datetime.date(2020, 1, 4)
```

보시다시피 음수 인덱스도 작동하는 것을 확인할 수 있다. 이는 DateRangeSequence 객체가 모든 작업을 래핑된 객체인 `days`에 위임했기 때문이다.

## 컨테이너 객체

컨테이너는 `__contains__` 메서드를 구현한 객체로 `__contains__` 메서드는 일반적으로 불리언 값을 반환한다. 해당 메서드는 파이썬에서 `in`키워드가 발견될 때 호출된다.

```python
element in container # container.__contains__(element)
```

이 메서드를 잘 사용하면 코드의 가독성이 정말 높아진다. 아래의 예시는 2차원 게임 지도에서 특정 위치에 표시를 해야 하는 경우를 구현한 코드이다.

```python
# 난해한 코드
def mark_coordinate(grid, coord):
    if 0 <= coord.x < grid.width and 0 <= coord.y < grid.height:
        grid[coord] = MARKED

# 정리된 코드
class Boundaries:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __contains__(self, coord):
        x,y = coord
        return 0 <= x < self.width and 0 <= y < self.height

class Grid:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.limits = Boundaries(width, height)

    def __contains__(self, coord):
        return coord in self.limits
```

위의 난해한 코드는 복잡하고 이해하기도 직관적이지 않다. 아래의 코드처럼 클래스로 정리한다면 훨씬 가독성이 좋고 깔끔하다.

## 객체의 동적인 속성

`__getattr__` 매직 메서드를 사용하여 객체에서 속성을 얻는 방법을 제어할 수 있다.

```python
class DynamicAttributes:
    def __init__(self, attribute):
        self.attribute = attribute

    def __getattr__(self, attr):
        if attr.startswith("fallback_"):
            name = attr.replace("fallback_","")
            return f"[fallback resolved] {name}"
        raise AttributeError(f"{self.__class__.__name__}에는 {attr} 속성이 없음")
```

위의 예시를 이용하여 `__getattr__`의 작동원리를 확인해보자.

```python
> temp = DynamicAttributes('value')
> temp.fallback_test # __getattr__ 적용
'[fallback resolved] test'
```

첫 번째 예시는 `fallback_test`라고 attribute 메서드로 보냄으로써 `f"[fallback resolved] {name}"` 해당 코드값이 반환된다.

```python
>temp.__dict__['fallback_new'] = 'new_value'
>temp.__dict__
{'attribute': 'value', 'fallback_new': 'new_value'}
>getattr(temp, "fallback_new")
'new_value'
```

두 번째 예시는 인스턴스 `temp`에 새로운 속성을 삽입한 후 `getattr()`을 통해 불러오면 속성값이 나오는 것을 확인할 수 있다.

```python
>getattr(temp, "someting")
AttributeError: DynamicAttributes에는 someting 속성이 없음
> getattr(temp, "someting","yaah")
'yaah'
```

마지막 예시는 값을 검색할 수 없는 경우에는 `AttributeError`가 일어나는 것을 확인할 수 있다. 신기한 것은 뒤에 하나를 추가하면 마지막 값을 리턴하는 것이다. 이것은 기존에 파라미터 2개로 설정된 `getattr()`가 작동하는 것 같다.

## 호출형 객체

함수처럼 작동하는 객체는 매우 편리하다. 매직 메서드인 `__call__`을 사용하면 일반 객체를 함수처럼 호출할 수 있다.

```python
from collections import defaultdict

class CallCount:

    def __init__(self):
        self._counts = defaultdict(int)

    def __call__(self, argument):
        self._counts[argument]+=1
        return self._counts[argument]
```

위 코드는 해당 객체를 반복할수록 `__call__`를 호출하여 객체 값을 변경한다.

```python
> cc = CallCount()
> cc(1)
1
> cc(1)
2
> cc(1)
3
> cc(2)
1
> cc.__dict__
{'_counts': defaultdict(int, {1: 3, 2: 1})}
```

`__dict__`를 사용하여 객체들을 확인해보면 지금까지 변경된 부분이 저장되어 있는 것을 알 수 있다.

## 파이썬에서 유의할 점

언어의 주요 기능을 이해하는 것 외에도 흔히 발생하는 잠재적인 문제를 피할 수 있는 관용적인 코드를 작성하는 것도 중요하다. 이번 섹션에서 논의되는 대부분은 완전히 해결할 수 있는 문제들이다. 따라서 지금부터 설명하는 방식대로 리팩토링하는 것이 좋다.

### 변경가능한 파라미터 값

쉽게 말해 변경 가능한 객체를 함수의 기본 인자로 사용하면 안된다.
