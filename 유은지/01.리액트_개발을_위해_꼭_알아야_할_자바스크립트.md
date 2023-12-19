# 1. 자바스크립트의 동등 비교

리액트 컴포넌트의 렌더링이 일어나는 이유 중 하나가 props의 동등 비교

props의 동등 비교 → 객체의 얕은 비교를 기반

## 1. Object.is

- `==` (Equality Operator)
    - 값이 동등한지 여부를 검사하고 자동 형 변환이 이루어짐
- `===` (Strict Equality Operator)
    - 값과 타입이 모두 동일한지 여부를 검사한다
    - 형 변환이 없이 엄격한 비교를 수행한다
- `Object.is`
    - 두 값이 같은지 여부를 엄격하게 비교한다
    - `===` 와 유사하지만 몇가지 경우에 다르게 동작한다
    
    ```jsx
    -0 === +0 // true
    Object.is(-0, +0) // false
    
    Number.NaN === NaN // false
    Object.is(Number.NaN, NaN) // true
    
    NaN === 0 / 0 // false
    Object.is(NaN, 0 / 0) // true
    ```
    
    - 각각 결과가 다른 이유
        1. 0과 +0 비교:
            - JavaScript에서 0과 -0을 내부적으로 다르게 표현하는데, `===`는 값과 타입만을 고려하므로 둘을 같다고 판단하고, `Object.is()`는 엄격한 동등성 비교를 수행하여 둘을 다르다고 판단하기 때문입니다.
        2. NaN 비교:
            - `Object.is()`는 NaN 값에 대해서는 특별한 규칙을 적용하여 두 NaN을 같다고 평가합니다. 따라서 `Object.is(Number.NaN, NaN)`은 `true`가 됩니다.
        3. NaN과 0/0 비교:
            - `0 / 0`는 NaN 값을 생성하는데, `===`는 NaN과 다르다고 판단하고, `Object.is()`는 NaN 값에 대해 특별한 동등성을 적용하여 둘을 같다고 판단합니다.
        
        기본적으로 NaN은 자기 자신과의 비교에도 false를 반환한다.
        
        `Object.is()`는 두 NaN 값이 실제로 같은 부류인 ‘Not a Number’를 나타내기 때문에 같다고 판단한다. 즉 `===` 이 있음에도 `Object.is()`가 등장한 이유는  NaN값 비교, -0 와 +0비교 때문이다.
        

## 2. 리액트에서 동등 비교

- `Object.is()` 기반의 [shallowEqual](https://github.com/facebook/react/blob/3e00e58a6ac7f73a3660f31d3129fb06d344167e/packages/shared/shallowEqual.js#L4) 함수를 사용한다
1. `Object.is()`로 먼저 비교를 수행한다
2. `Object.is()`에서 수행하지 못하는 비교, 즉 객체 간 얕은 비교를 한 번 더 수행한다
    - 객체간 얕은 비교란 객체의 첫번째에 존재하는 값만 비교한다는 의미

```jsx
// Object.is()는 참조가 다른 객체에 대해 비교가 불가능하다. 
Object.is({ hello: 'world' }, { hello: 'world' }) // false

// 반면 리액트 팀에서 구현한 shallowEqual은 객체의 1 depth까지는 비교가 가능하다.
shallowEqual({ hello: 'world' }, { hello: 'world' }) // true

// 그러나 2 depth까지 가면 이를 비교할 방법이 없으므로 false를 반환한다.
shallowEqual({ hello: { hi: 'world' } }, { hello: { hi: 'world' } }) // false
```

얕은 비교까지만 구현한 이유는?

- JSX props는 객체이고, props 값만 꺼내와서 일차적으로 비교하면되기 때문
- props에 또 다른 객체를 넘겨주면 렌더링이 예상치 못하게 작동한다
- [https://velog.io/@jinhyukoo/React-얕은-비교에-대한-얕은-오해](https://velog.io/@jinhyukoo/React-%EC%96%95%EC%9D%80-%EB%B9%84%EA%B5%90%EC%97%90-%EB%8C%80%ED%95%9C-%EC%96%95%EC%9D%80-%EC%98%A4%ED%95%B4)

---

# 2. 함수

## 1. 함수란?

- 작업을 수행하거나 값을 계산하는 등의 과정을 표현하고, 하나의 블록으로 감싸서 실행 단위로 만들어 놓은 것

## 2. 함수를 정의하는 4가지 방법

### 1) 함수 선언문

- 표현식이 아닌 문으로 분류

### 2) 함수 표현식

- 함수를 변수에 할당하는 방식
- 할당하려는 함수의 이름을 생략하는 것이 일반적

> 함수 선언문 VS 함수 표현식 
- 호이스팅 차이
- 함수 호이스팅은 함수 선언문을 코드 마치 코드 맨 앞단에 작성된 것처럼 작동하는 특징
- 함수 선언문은 미리 메모리에 등록이 됨
- 표현식은 undefined로 초기화됨
> 

### 3) Function 생성자

- `new Function` 생성자 활용
- 권장 X

### 4) 화살표 함수

- construnctor를 사용할 수 없음
- arguments가 존재하지 않음
- this 바인딩 존재하지 않음

## 3. 다양한 함수 살펴보기

### 1) 즉시 실행함수

- 함수를 정의하고 그 즉시 실행되는 함수
- 단 한 번만 호출되고, 다시 호출할 수 없는 함수
- 글로벌 스코프를 오염시키지 않는 독립적인 함수 스코프를 운용할 수 있다는 장점

### 2) 고차함수

- 함수를 인수로 받거나 결과로 새로운 함수를 반환시킬 수 있는 함수
- 고차 컴포넌트 : 함수형 컴포넌트를 인수로 받아 새로운 함수형 컴포넌트를 반환하는 고차 함수
    - 공통으로 관리되는 로직을 분리해 관리할 수 있어 효율적으로 리팩토링 가능

### 4. 함수를 만들 때 주의할 사항

### 1) 함수의 부수 효과를 최대한 억재하라

- 함수의 부수 효과 : 함수 내 작동으로 함수 외부에 영향을 끼치는 것
- 순수 함수 : 부수효과가 없는 함수, 언제 실행되든 항상 결과가 동일하기에 예측 가능하며 안정적
- 비순수 함수 : 부수 효과가 존재하는 함수
- 개발 시 부수효과를 피할 수 없다(`useEffect` 같은 경우 컴포넌트에서 부수효과를 수행하기 위해 사용됨. 즉 이런 경우를 최소화 해서 개발해라~)

### 2) 가능한 한 함수를 작게 만들어라

- eslint에 `max-lines-per-function` 같은 조건을 둬서 함수 라인을 제한하듯이 코드 길이가 길어질 수록 에러 확률이 높아진다

### 3) 누구나 이해할 수 있는 이름을 붙여라

- 가능한 한 함수 이름은 간결하고 이해하기 쉽게 붙여라

---

# 3. 클래스

- 특정한 객체를 만들기 위한 일종의 템플릿과 같은 개념

## 1. 클래스 기본 개념

- `constructor`
    - 생성자
    - 객체를 생성하고 초기화
    - 클래스 당 하나만 존재
    - 생략가능
- 프로퍼티
    - 클래스로 인스턴스를 생성할 때 내부에 정의할 수 있는 속성값
    - `#` 사용해서 private 선언
- `getter`, `setter`
    - `getter` : 값을 가져올 때 사용
    - `setter` : 필드에 값을 할당할 때 사용
- 인스턴스 메서드(프로토타입 메서드)
    - 클래스 내부에서 선언한 메서드
    - 인스턴스에서 접근 가능
    - 프로토타입 체이닝으로 메서드 찾기 가능 : 최상위 객체인 `Object`를 훑는 과정
- 정적 메서드
    - 클래스의 인스턴스가 아닌 이름으로 호출할 수 있는 메서드
    - 클래스내에서만 접근 가능
    - this에 접근 불가능
- 상속
    - `extends` 키워드를 사용해서 상속받는다

## 2. 클래스와 함수 관계

- 객체지향적 프로그래밍을 하던 개발자가 좀 더 자바스크립트에 접근하기 쉽게 해주는 문법적 설탕
- 자바스크립트 클래스는 프로토타입 기반으로 작동

# 4. 클로저

- 함수형 컴포넌트 구조와 작동 방식
- 훅의 원리
- 의존성 배열 등

## 1. 클로저의 정의

- 함수와 함수가 선언된 어휘적 환경의 조합
- 선언된 어휘적 환경 : 변수가 코드 내부에서 어디서 선언됐는지, `this` 와 다르게 코드가 작성된 순간 정적으로 결정된다

## 2. 변수의 유효 범위, 스코프

### 1) 전역 스코프

- 전역 레벨 선언
- 이 스코프에서 변수를 선언핳면 어디서든 호출할 수 있게 된다
- 브라우저의 전역 객체는 `window`, Node.js의 전역 객체는 `global`

### 2) 함수 스코프

- 자바스크립트는 기본적으로 함수 레벨 스코프를 따른다
- `{}` 블록이 스코프 범위를 결정하지 않는다
- 자바스크립트에서 스코프는 가장 가까운 스코프에서 변수가 존재하는지 먼저 확인한다

```jsx
var x = 10

function foo() {
  var x = 100
  console.log(x) // 100

  function bar() {
    var x = 1000
    console.log(x) // 1000
  }

  bar()
}

console.log(x) // 10
foo()
```

### 3) 클로저의 활용

클로저의 활용

```jsx
function Counter() {
  var counter = 0

  return {
    increase: function () {
      return ++counter
    },
    decrease: function () {
      return --counter
    },
    counter: function () {
      console.log('counter에 접근!')
      return counter
    },
  };
}

var c = Counter()

console.log(c.increase()) // 1
console.log(c.increase()) // 2
console.log(c.increase()) // 3
console.log(c.decrease()) // 2
console.log(c.counter()) // 2
```

예_직접적으로 counter변수를 노출시키지 않음으로써 사용자가 직접 수정하는 것 막음

예_접근하는 경우를 제한해 로그를 남김

- 이처럼 전역 스코프의 사용을 막고, 개발자가 원하는 정보만 개발자가 원하는 방향으로 노출시킬 수 있음

리액트에서 클로저

- 대표적으로 사용하고 있는 것이 `useState`

```jsx
function Component() {
  const [state, setState] = useState()

  function handleClick() {
    // useState 호출은 위에서 끝났지만,
		// setState는 계속 내부의 최신값(prev)를 알고 있다.
		// 이는 클로저를 활용했기 때문에 가능하다.
    setState((prev) => prev + 1)
  }

  // ...
}
```

- 클로저가 useState 내부에서 활용됨
- 외부 함수(`useState`)가 반환한 내부 함수(`setState`)는 외부함수(`useState`)의 호출이 끝났음에 자신이 선언된 외부함수가 선언된 환경(state가 저장된 어딘가)을 기억하기 때문에 계속해서 state값을 사용할 수 있는 것

### 4)주의할 점

- 생성될 떄마다 그 선언적 환경을 기억하기 위해서 비용이 발생한다 (메모리 사용)
- 클로저 사용을 적절한 스코프로 가두지 않으면 선응에 악영향을 미친다

---

# 5. 이벤트 루프와 비동기 통신의 이해

- 싱글 스레드
    - 한 번에 하나의 작업만 동기 방식
- 동기
    - 직렬 방식으로 작업 처리
    - 무조건 응답을 받은 후에 다른 작업 처리 가능
- 비동기
    - 병렬 방식으로 작업 처리
    - 한 번에 여러 작업 실행 가능
- 자바스크립트는 싱글 스레드에서 동기 방식으로 작동

## 1. 싱글 스레드 자바스크립트

- 프로세스
    - 프로그램을 구동해 프로그램 상태가 메모리상에서 실행되는 작업 단위
    - 하나의 프로그램에는 하나의 프로세스만 할당됨
    - 하지만 소프트웨어가 복잡해지면서 하나의 프로그램에서 동시에 여러 개의 복잡한 작업 수행 필요성 커짐
- 스레드
    - 더 작은 실행 단위
    - 하나의 프로세스에서 여러개의 스레드를 만들 수 있음
    - 스레드끼리 메모리 공유가능해서 여러 가지 작업을 동시에 수행 가능해짐

- 그럼 왜 자바스크립트는 싱글 스레드일까?
    - 멀티 스레드의 단점
        - 동시성 문제 (같은 자원 여러번 수정, 다른 스레드 문제 발생 시 다른 스레드에도 영향)
        - 여러 스레드에서 DOM을 조작하면 메모리 공유로 인해 타이밍 이슈 발생, DOM 표시에 큰 문제 발생할 수 있음
- 자바스크립트는 싱글 스레드
    - 자바스크립트 코드의 실행이 하나의 스레드에서 순차적으로 이루어진다라는 뜻(Run-to-completion)

## 2. 이벤트 루프

- 자바스크립트 런타임 외부에서 자바스크립틔 비동기 실행을 돕기 위해 만들어진 장치

### 1)  호출 스택과 이벤트 루프

- 호출 스택
    - 자바스크립트에서 수행해야 할 코드나 함수를 순차적으로 담아두는 스택이다.
- 이벤트 루프
    - 이벤트 루프만의 단일 스레드 내부에서 호출 스택에 수행해야 할 작업이 있는지 확인하고, 수행해야 할 코드가 있다면 자바스크립트 엔진을 이용해 실행한다
    - ‘코드를 실행하는 것’과 ‘호출 스택이 비어있는지 확인하는 것’ 모두가 단일 스레드에서 일어난다. 즉, 동시에 일어날 수 없으며 한 스레드에서 순차적으로 일어남
    - 한 개 이상의 태스크 큐를 가지고 있다
    - 호툴 스택에 실행 중인 코드가 있는 지, 태스크 큐에 대기 중인 함수가 있는 지 반복적으로 확인한다 (태스크 큐가 빌 때까지)
- 태스크 큐
    - 실행해야 할 태스크의 집합
    - 큐가 아닌 set의 형태를 띠고 있음 → 선택한 큐 중에서 실행 가능한 가장 오래된 테스크를 가져와야 하기 때문이다
    - 비동기 함수는 메인 스레드가 아닌 태스크 큐가 할당되는 별도의 스레드에서 수행된다
    - 이 별도의 스레드에서 태스크 큐에 작업을 할당해 처리하는 것은 브라우저나 Node.js의 역할
    - 자바스크립트 코드 실행은 싱글 스레드에서 이루어지지만 이러한 외부 Web API 등은 모두 자바스크립트 코드 외부에서 실행되고 콜백이 태스크 큐로 돌아가는 것

## 3. 태스크 큐와 마이크로 태스크 큐

- 이벤트 루프에는 1개의 마이크로 태스크 큐
- 기존의 태스크 큐와는 다른 태스크를 처리함
- 마이크로 태스크 큐는 기존 태스크 큐보다 우선권을 갖음 (예_`Promise`)
- 마이크로 태스크 큐가 빌 때까지는 기존 태스크 큐의 실행은 뒤로 미뤄짐
- 마이크로 태스크 큐 → 렌더링 → 태스크 큐
- 태스크 큐: `setTimeout`, `setInterval`, `setImmediate`
- 마이크로 태스크 큐: `process.nextTick` ,`Promise`,`queueMicroTask`,`MutationObserver`

---

# 6. 리액트에서 자주 사용하는 자바스크립트 문법

## 1. 구조 분해 할당

- 배열 또는 객체의 값을 그대로 분해해 개별 변수에 즉시 할당

### 1) 배열 구조 분해 할당

```jsx
const array = [1, 2, 3, 4, 5]

// 1. 각 변수에 할당
const [first, second, third, ...arrayRest] = array

// 2. ,의 위치에 따라 값이 결정된다
const [first, , , , fifth] = array // 2,3,4는 아무런 표현식이 없어 변수 할당 생략

// 3. 초기값 설정 -> 실수 유발 가능성 높음
const array2 = [1, 2]
const [a = 10, b = 20, c = 30] = array2
```

```jsx
// 4. undefined일 때만 기본 값을 사용한다
const [a = 1, b = 1, c = 1, d = 1, e = 1] = [undefined, null, 0, '']
// a 1
// b null
// c 0
// d ''
// e 1
```

### 2) 객체 구조 분해 할당

```jsx
// 새로운 이름으로 다시 할당하는 것 가능
const object = {
  a: 1,
  b: 2,
}

const { a: first, b: second } = object
// first 1
// second 2
```

```jsx
// 계산된 속성 이름 방식으로 꺼내오기 가능
const key = 'a'
const object = {
  a: 1,
  b: 2,
}

const { [key]: a } = object

// a = 1
// a라는 값을 거내 오기 위해 [key] 문법을 사용하고, 반드시 이름을 선언하는 :a와 같은 변수 네이밍이 필요함
```

- 트랜스파일할 경우 번들링 크기가 상대적으로 크다

## 2. 전개 구문

- 배열이나 객체, 문자열과 같이 순회할 수 있는 값에 대해 전개해 간결하게 사용할 수 있는 구문

### 1) 배열의 전개 구문

- push(), concat(), splice() 등 메서드를 사용하지 않아도 됨
- 배열 내부에서 `…배열` 을 사용하면 배열을 마지 전개하는 것 처럼 선언하고, 이를 내부 배열에서 활용 가능
- 기존 배열에 영향을 미치지 않고 배열을 복사하는 것이 가능

```jsx
const arr1 = ['a', 'b']
const arr2 = [...arr1, 'c', 'd', 'e'] // ['a', 'b', 'c', 'd', 'e']
```

```jsx
const arr1 = ['a', 'b']
const arr2 = arr1

arr1 === arr2; // true. 내용이 아닌 참조를 복사함

const arr1 = ['a', 'b']
const arr2 = [...arr1]

arr1 === arr2 // false. 실제로 값만 복사될 뿐, 참조는 다름
```

### 2) 객체의 전개 구문

```jsx
const obj1 = {
  a: 1,
  b: 2,
}

const obj2 = {
  c: 3,
  d: 4,
}

const newObj = { ...obj1, ...obj2 }
// { "a": 1, "b": 2, "c": 3, "d": 4 }
```

- 객체 전개 구문은 순서가 중요함

## 3. 객체 초기자

- 객체를 선언할 때 객체에 넣고자 하는 키와 값을 가지고 있는 변수가 존재한다면 해당 값을 간결하게 넣어줄 수 있는 방법

```jsx
const a = 1
const b = 2

const obj = {
  a,
  b,
}

// {a : 1, b : 2}
```

## 4. Array 프로토타입의 메서드 : map, filter, reduce, forEach

## 5. 삼항 조건 연산자

JSX 내부에서 조건부 삼항 조건 연산자가 아닌 방식으로 조건부 렌더링 구현하기

```jsx
<div>
  {(() => {
    if (color === 'red') {
			 return '빨간색이다.'
		}else {
		 return '빨간색이 아니다.'
		}
  })()}
</div>
```

→ JSX 내부에 자바스크립트 할당식을 사용할 수 있기 때문이지만 이런 구문은 가독성을 해피고 불필요한 즉시 실행 함수를 사용한다.

---

# 7. 선택이 아닌 필수, 타입스크립트

타입 체크를 정적으로 런타임이 아닌 빌드(트랜스 파일) 타임에 수행할 수 있게 해준다.

## 1. 리액트 코드를 효과적으로 작성하기 위한 타입스크립트 활용법

### 1) `any` 대신에 `unknown`을 활용하자

- `unknown`은 모든 값을 할당할 수 있는 top type, 어떠한 값도 할당할 수 있음
- `unknown` 으로 선언된 변수를 사용하기 위해 type narrowing 필요

### 2) 타입 가드를 적극 활용하자

- instancof, typeof : 어떤 클래스의 인스턴스인지 / 자료형을 확인
- in : 어떤 객체에 키가 존재하는지 확인하는 용도

### 3) 제네릭

- 함수나 클래스 내부에서 단일 타입이 아닌 다양한 타입에 대응할 수 있도록 도와줌

### 4) 인덱스 시그니처

- 객체의 키를 정의하는 방식

```jsx
// 기본
type Hello = {
	[key : string] : string
}

// record 사용
type Hello = Record<'hello' | 'hi', string>

// 타입을 사용한 인덱스 시그니처
type Hello = { [key in 'hello' | 'hi'] : string }

// 사용
const hello : Hello = {
	hello : 'hello',
	hi : 'hi',
}

```

```jsx
Object.keys(hello).map((key) => {
  // Element implicitly has an 'any' type
  // because expression of type 'string' can't be used to index type 'Hello'
  // No index signature with a parameter of type 'string' was found on type 'Hello'
  const value = hello[key];
  return value;
});
```

- string[]을 반환해 문제가 생기는 경우
- https://github.com/microsoft/TypeScript/issues/45390

```jsx
// Object.keys(hello)를 as로 타입을 단언하는 방법
(Object.keys(hello) as Array<keyof Hello>).map((key) => {
  const value = hello[key]
  return value
})
```

```jsx
// 타입 가드 함수를 만드는 방법
function keysOf<T extends Object>(obj: T): Array<keyof T> {
  return Array.from(Object.keys(obj)) as Array<keyof T>
}

keysOf(hello).map((key) => {
  const value = hello[key]
  return value
})
```

```jsx
// 가져온 key를 단언하는 방법
Object.keys(hello).map((key) => {
  const value = hello[key as keyof Hello];
  return value;
});
```

타입스크립트는 객체의 타입에 구애받지 않고 객체의 타입에 열려 있는 자바스크립트의 특징에 맞춰져 있다. 

## 3. 타입스크립트 전환 가이드

### 1) tsconfig.json 먼저 작성하기

### 2) JSDoc과 @ts-check를 활용해 점진적으로 전환하기

- 상단에 `// @ts-check` 를 선언
- JSDoc을 활용해 변수나 함수에 타입을 제공

### 3) 타입 기반 라이브러리 사용을 위해 @types 모듈 설치하기

### 4) 파일 단위로 조금씩 전환하기
