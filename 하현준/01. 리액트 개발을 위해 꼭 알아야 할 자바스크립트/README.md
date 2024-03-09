# 1장 리액트 개발을 위해 꼭 알아야 할 자바스크립트

## 자바스크립트의 데이터 타입

### **자바스크립트의 원시타입**

`null`은 **typeof null**로 확인 했을때 `object` 결과가 반환됌

```tsx
typeof null === "object";
```

- 이는 버그다?
- 기존에 쓰이던 코드에 영향이 많아서 고칠수가 없었다

### **typeof 를 확인하는 순서**

https://2ality.com/2013/10/typeof-null.html

```tsx
JS_PUBLIC_API(JSType)
    JS_TypeOfValue(JSContext *cx, jsval v)
    {
        JSType type = JSTYPE_VOID;
        JSObject *obj;
        JSObjectOps *ops;
        JSClass *clasp;

        CHECK_REQUEST(cx);
        if (JSVAL_IS_VOID(v)) {  // (1)
            type = JSTYPE_VOID;
        } else if (JSVAL_IS_OBJECT(v)) {  // (2)
            obj = JSVAL_TO_OBJECT(v);
            if (obj &&
                (ops = obj->map->ops,
                 ops == &js_ObjectOps
                 ? (clasp = OBJ_GET_CLASS(cx, obj),
                    clasp->call || clasp == &js_FunctionClass) // (3,4)
                 : ops->call != 0)) {  // (3)
                type = JSTYPE_FUNCTION;
            } else {
                type = JSTYPE_OBJECT;
            }
        } else if (JSVAL_IS_NUMBER(v)) {
            type = JSTYPE_NUMBER;
        } else if (JSVAL_IS_STRING(v)) {
            type = JSTYPE_STRING;
        } else if (JSVAL_IS_BOOLEAN(v)) {
            type = JSTYPE_BOOLEAN;
        }
        return type;
    }
```

- At (1), the engine first checks whether the value  is  (VOID). This check is performed by comparing the value via equals: `#define JSVAL_IS_VOID(v)  ((v) == JSVAL_VOID)`
  (1) 에서 값이 **VOID** 값(**undefined**)인지 확인한다.
- The next check (2) is whether the value has an object tag. If it additionally is either callable (3) or its internal property [[Class]] marks it as a function (4) then  is a function. Otherwise, it is an object. This is the result that is produced by .
  (2)에서 객채 태그를 가졌는지 확인, 그리고 (3)에서 호출이 되거나, 내부 property (Class) 가 함수로 표현되면 이는 ‘**function**’으로 체크한다. 아니면 ‘**Object**’로 판단하는데 여기서 `null` 이 걸려서 ‘**Object**’ 로 판단되게 된다.
- The subsequent checks are for number, string and boolean.
  나머지는 이제 ‘**number**’,’**string**’,’**boolean**’ 으로 체크한다.

`undefined` vs `null`

`undefined` 는 ‘선언됐지만 할당되지 않은 값’, `null` 은 명시적으로 비어 있음을 나타내는 값

null 은 가비지 컬랙터에 수집이 된다.

React에서 코드를 짤때 null과 undefined 중 무엇을 선호하며 그 이유는 무엇인가요?

### **자바스크립트의 또 다른 비교 공식, Object.is**

`==` vs `Object.is`

- ==(비교 연산자): 같은 타입이 아니라면 강제로 형변환(type casting) 을 한 뒤에 비교한다.
- Obect.is: ===와 동일하게 타입이 다르면 `false` 를 리턴

`===` vs `Object.is`

- `===`(엄격한 비교 연산자) : 타입 형변환을 진행하지 않고 있는 그대로를 비교
- `Object.is` : === 으로는 비교가 불가능 했던 ‘NaN’, +0, -0을 우리가 생각하는 보통의 방식으로 비교를 해줌

```tsx
-0 === +0; // true
Object.is(-0, +0); // false

Number.NaN === NaN; // false
Object.is(Number.NaN, Nan); // true
```

### **리액트에서의 동등 비교**

https://github.com/facebook/react/blob/main/packages/shared/shallowEqual.js

```tsx
import is from "./objectIs";
import hasOwnProperty from "./hasOwnProperty";

/**
 * Performs equality by iterating through keys on an object and returning false
 * when any key has values which are not strictly equal between the arguments.
 * Returns true when the values of all keys are strictly equal.
 */
function shallowEqual(objA: mixed, objB: mixed): boolean {
  if (is(objA, objB)) {
    return true;
  }

  if (typeof objA !== "object" || objA === null || typeof objB !== "object" || objB === null) {
    return false;
  }

  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) {
    return false;
  }

  // Test for A's keys different from B.
  for (let i = 0; i < keysA.length; i++) {
    const currentKey = keysA[i];
    if (
      !hasOwnProperty.call(objB, currentKey) ||
      // $FlowFixMe[incompatible-use] lost refinement of `objB`
      !is(objA[currentKey], objB[currentKey])
    ) {
      return false;
    }
  }

  return true;
}

export default shallowEqual;
```

Object.is를 통해 먼저 비교한 뒤, 수행하지 못하는 비교는 객체간의 얕은 비교를 한 번 더 수행한다.

```tsx
Object.is({ hello: "world" }, { hello: "world" }); // false

shallowEqual({ hello: "world" }, { hello: "world" }); // true

shallowEqual({ hello: { hi: "world" } }, { hello: { hi: "world" } }); // false
// 얕은 비교이므로 false 를 리턴
```

즉, 객체간의 비교를 얕게 비교하기 때문에 react에서도 props로 2depth의 객체를 넘기면

`memo` 키워드를 사용하더라도 최적화가 실행되는 코드가 아니다. (올바르게 동작시키려면 이러한 이유를 안다면 성능적으로 개선이 가능할 것)

## 함수

<aside>
💡 **표현식과 문**

**값(value)**은 식(표현식expression)이 평가(evaluate)되어 생성된 결과를 말한다.

**문(Statement)** 은 프로그램을 구성하는 기본 단위이자 최소 실행 단위이다.

**표현식(expression)**은 **값으로 평가될 수 있는 문**이다. 즉, 표현식이 평가되면 새로운 값을 생성하거나 기존 값을 참조한다. > 중요한 점은 값으로 평가가 될 수 있어야 한다.

</aside>

표현식이 아닌 문

```tsx
// 함수 선언문은 값으로 평가될 수 없으므로 표현식이 아닌 문이다.
function add(a, b) {
  return a + b;
}
```

표현식인 문

```tsx
const sum = function sum(a, b) {
  return a + b;
};

sum(10, 24);
// 값으로 평가되므로 표현식인 문이다.
// 물론 sum 자체는 표현식이 아닌 문이다.
```

sum을 익명함수로 작성하는 이유는 개발자들이 개발에 함에 있어 혼란을 야기할 수 있는 부분을 제가함으로써 읽는데 방해요소를 제거 할 수 있기 때문이다. (`const sum = (a, b) => a + b`)

### **함수는 ‘일급 객체’ 다**

> 일급객체(First-class Object)란 다른 객체들에 일반적으로 적용 가능한 연산을 모두 지원하는 객체를 가리킨다.

**일급객체**의 조건

- 변수에 할당(assignment)할 수 있다.
  ```tsx
  const sum = function sum(a, b) {
    return a + b;
  };
  ```
- 다른 함수를 인자(argument)로 전달 받는다.

  ```tsx
  function sum(func) {
    return func;
  }

  function add(a, b) {
    return a + b;
  }
  sum(add(10, 20)); // 30
  ```

- 다른 함수의 결과로서 리턴될 수 있다.

  ```tsx
  function add(num1) {
    return function (num2) {
      return num1 + num2;
    };
  }

  add(10)(20); // 30
  // 리턴되는 함수에 다시 인자를 넣어서 함수를 실행한다.
  ```

### **함수를 만들 때 주의해야 할 사항**

- 함수의 부수 효과(side-effect)를 최대한 억제하라
- 가능한 한 함수를 작게 만들어라
- 누구나 이해할 수 있는 이름을 붙여라

  - 한글 변수명도 고려해보는 것도 나쁘지 않을지도(회사에서 적용해봤음)

  ```tsx
  // 회사에서 각 영역의 노출조건이 모두 복잡해서 하나로 묶고
  // 이를 한글변수명을 통해 관리했음

  const 00_영역_노출여부_조건 = ~~~
  const 01_영역_노출여부_조건 = ~~~
  const 02_영역_노출여부_조건 = ~~~
  const 03_영역_노출여부_조건 = ~~~
  ```

## 클래스

### **인스턴스 메서드**

인스턴스 메서드란 클래스 내부에 선언한 메서드를 말한다. 인스턴스 메서드들은 자바스크립트의 `prototype` 에 선언되어서 프로토타입 메서드로 불리기도 한다.

```tsx
const myCar = new Car("자동차");

Object.getPrototypeOf(myCar) === Car.prototype; // true
myCar.__proto__ === Car.prototype; // true
```

`__proto__` 와 `Object.getPrototypeOf`는 동일하지만, `__proto__` 사용은 가급적 자제한다.

앞서 `typeof null === 'object'` 와 비슷하게 버그이지만 호환성을 위해서만 존재했던 기능이기에 사용하지 말자

### **클래스와 함수의 관계**

클래스는 문법적 설탕이므로 프로토타입을 조금더 편리하게 사용하게 해주는 문법이다.

즉, 클래스를 활용하는 것은 자바스크립트의 프로토타입을 활용하는 것

## 클로저

클로저는 함수와 함수가 선언된 어휘적 환경(lexical scope)의 조합이다.

리액트는 클로저 덩어리이므로 클로저의 정의에 쓰여있던 ‘어휘적 환경’을 먼저 보자

```tsx
function add() {
  const a = 10;
  function innerAdd() {
    const b = 20;
    console.log(a + b);
  }
}
```

a 와 b 변수의 유효 범위는 선언된 위치에 따라 달라지고 이에 결정되므로 이를 ‘선언된 어휘적 환경’이라 볼 수 있다. this와 다르게 코드가 작성되는 순간 **정적으로 결정된다.**

변수의 유효 범위

- 전역 스코프: 이 스코프에 변수를 선언하면 어디에서든 호출할 수 있는 환경이 된다.
- 함수 스코프: `{ }` 블록이 스코프 범위를 결정하는 것이 아닌 함수 레밸 스코프를 따름

  ```tsx
  if (true) {
    var global = "global scope";
  }

  console.log(global); // {} 블록 안에 있지만 접근이 가능
  ```

### **클로저의 활용**

```tsx
function outerFunction() {
  var x = "hello";
  function innerFunction() {
    console.log(x);
  }
  return innerFunction;
}

const innerFunction = outerFunction();
innerFunction(); // 'hello'
```

`outerFunction` 은 `innerFunction` 함수를 반환하며 실행은 이미 종료되었다.

그러나 `outerFunction` 라는 선언된 어휘적 환경에 `x` 라는 변수가 존재하며 이를 접근할 수 있어서 x라는 변수에 저장된 ‘hello’ 도 출력하고 가비지 컬랙션에 의해 수집도 되지 않는 것이다.

### **리액트에서의 클로져**

`useState` 는 `setState`를 반환함으로써 외부에서도 선언된 환경을 기억하기 때문에 state를 사용할 수 있다.

```tsx
function Component() {
  const [state, setState] = useState();

  function handleClick() {
    // 내부의 최신 값(prev) 를 알고 이를 + 1 로 갱신할 수 있게 된다.
    setState((prev) => prev + 1);
  }
}
```

### **클로저를 사용할 때 주의할 점**

```tsx
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
}
```

위 함수는 0, 1, 2, 3, 4 차례대로 출력하는 것을 의도했지만 정작 5만 5번 출력되게 된다.

`i` 가 `var`이라는 키워드로 선언되어 있고 `var`는 함수 레밸 스코프를 따르고 있기 때문에 `setTimeout` 이 실행되려고 볼 때 이미 `i`는 5로 업데이트가 완료되어 버렸기 때문에 5가 5번 실행되는 것이다.

이를 해결하면 당연히 `var` 가 아닌 `let` 으로 바꾸면 해결된다.

다른 방법으로는 클로저를 ‘제대로’ 활용하면 된다.

```tsx
for (var i = 0; i < 5; i++) {
  setTimeout(
    (function (sec) {
      return function () {
        console.log(sec);
      };
    })(i),
    i * 1000
  );
}
```

즉시 실행 함수에서 `i` 를 인자로 받고 즉시 실행 함수는 `sec` 로 받아서 `setTimeout` 의 콜백 함수에 넘기게 되고, 매 for문이 실행될 때 즉시 실행 함수는 `i`를 각각 받아 고유한 `sec`를 가져 올바르게 실행될 수 있게 된다.

**긴 작업을 일반적인 함수로 처리하는 방법과 클로저로 처리하는 방법의 차이**

클로저를 사용할 경우 어디에 사용하는지 상관없이 해당 내용을 기억(클로저는 공짜가 아니다)해야 둬야 하기 때문에 메모리에 올려야 한다.

즉 메모리를 사용하므로 클로저를 사용할때는 적절한 스코프로 가둬둬야 한다.

## 이벤트 루프와 비동기 통신의 이해

### **싱글 스레드 언어인 자바스크립트**

초기에 자바스크립트는 다양한 일을 처리해야 할 것이라고 생각하지 못했음.

멀티 스레드를 지원하면 돔을 여러 스레드가 조작한다면 큰 문제를 야기 할 수 있음(나름 합리적인 결정??)

**Run-to-completion** : 자바스크립트의 모든 코드는 동기식으로 한번에 하나씩 순차적으로 처리되는 특징

### **이벤트 루프란?**

- 호출 스택

자바스크립트에서 수행해야 할 코드나 함수를 순차적으로 담아두는 스택

> 호출 스택이 비어있는지 여부를 확인하는 것이 바로 **이벤트 루프**

‘코드를 실행하는 것’ 과 ‘호출 스택이 비어있는지 확인하는 것’ 모두가 단일 스레드에서 실행

- 테스크 큐

실행해야 할 테스크(비동기 함수의 콜백 함수나 이벤트 헨들러) 의 집합(이름과 다르게 set형태를 띠고 있다)

### **태스크큐와 마이크로 태스크 큐**

마이크로 태스크에는 대표적으로 `promise` 가 있다

- 태스크 큐: `setTimeout`, `setInterval`, `setImmediate`
- 마이크로 태스크 큐: `process.nextTick`, `Promises`, `queueMicroTask`, `MutationObserver`

마이크로 태스크 큐 작업이 끝날 때마다 한 번씩 랜더링할 기회를 얻게되고,
마이크로 태스크 큐가 태스크 큐보다 우선순위를 가짐

실행 순서: 마이크로 태스크 큐 > 렌더링 > 태스크 큐 실행

## 리액트에서 자주 사용하는 자바스크립트 문법

### 구조 분해 할당(Desturcturing assignment)

lodash의 omit코드

```tsx
var omit = flatRest(function (object, paths) {
  var result = {};
  if (object == null) {
    return result;
  }
  var isDeep = false;

  // 제거할 키를 복제해서 새로운 객체를 만듬
  paths = arrayMap(paths, function (path) {
    path = castPath(path, object);
    isDeep || (isDeep = path.length > 1);
    return path;
  });
  copyObject(object, getAllKeysIn(object), result);

  if (isDeep) {
    result = baseClone(result, CLONE_DEEP_FLAG | CLONE_FLAT_FLAG | CLONE_SYMBOLS_FLAG, customOmitClone);
  }

  // 키가 없어질때까지 반복
  var length = paths.length;
  while (length--) {
    baseUnset(result, paths[length]);
  }
  return result;
});

// 사용법
var object = { a: 1, b: "2", c: 3 };

_.omit(object, ["a", "c"]);
```

### 전개 구문(Spread Syntax)

### 객체 초기자(object shorthand assignment)

### 삼항 조건 연산자

리액트에서 삼항연산자가 아닌 다른 방법으로 조건부 랜더링을 하는 방법

```jsx
<div>
  {(() => {
    if (color === "red") return "빨간색이다";
    else return "빨간색이 아니다.";
  })()}
</div>
```

가독성을 해치고 즉시 실행 함수를 사용해야 한다는 점에 때문에 선호 되지 않는다.

## 타입스크립트

`unknown`, `any`, `never`

https://velog.io/@haryan248/INFCON-2023-타입스크립트는-왜-그럴까

### 인덱스 시그니처

[예제 코드](https://www.typescriptlang.org/play?#code/C4TwDgpgBAEhA28D2UC8UBKEDGSBOAJgDwDkAFgsiQD5TkCWJANFAM7B70B2A5gHwBYAFDDcXdlAqIkALliUU6AN7CokhXPILmqyfU1lGwgL7DhAeQBGAKxzAAdAGsIIVgAopyAJT2AtgEMwNzcvND4oFSE1KDEJADd-eABXaHRPJABtZxAAXQBuXSg8CGAkvC4oBOSIEy8gA)

```jsx
type Hello = Record<"hello" | "hi", string>;

const hello: Hello = {
  hello: "hello",
  hi: "hi",
};

Object.keys(hello).map(() => {
  const value = hello[key]; // 오류 발생
  return value;
});
```

hello[key]에서 타입 오류가 발생한다.
그 이유는 Obect.keys가 string[]를 반환하기 때문인데,
이를 해결하려면 타입 단언을 사용하면 된다.

```tsx
(Object.keys(hello) as Array<keyof Hello>).map(() => {
  const value = hello[key]; // 오류 발생
  return value;
});
```

또는 KeysOf이라는 타입가드 함수를 만들어 사용하면 된다.

```tsx
function keysOf<T extends Object>(obj: T): Array<keyof T> {
  return Array.from(Object.keys(obj)) as Array<keyof T>;
}

keysOf(hello).map((key) => {
  const value = hello[key]; // 오류 발생
  return value;
});
```

💡 **덕 타이핑(Duck Typing)과 구조적 타이핑(Structural Typing)**

[https://medium.com/@yujso66/번역-왜-타입스크립트는-object-keys의-타입을-적절하게-추론하지-못할까요-477253b1aafa](https://medium.com/@yujso66/%EB%B2%88%EC%97%AD-%EC%99%9C-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-object-keys%EC%9D%98-%ED%83%80%EC%9E%85%EC%9D%84-%EC%A0%81%EC%A0%88%ED%95%98%EA%B2%8C-%EC%B6%94%EB%A1%A0%ED%95%98%EC%A7%80-%EB%AA%BB%ED%95%A0%EA%B9%8C%EC%9A%94-477253b1aafa)

```tsx
interface Lion {
  age: number;
  sounds: string;
}

interface Cat {
  age: number;
  sounds: string;
  favorite: "mouse" | "tuna" | "chur";
}

const simba: Lion = {
  age: 10,
  sounds: "roar",
};

const jerry: Cat = {
  age: 6,
  sounds: "meow",
  favorite: "chur",
};

function introduceLion({ age, sounds }: Lion): string {
  return `This lion is ${age} years old and sounds like ${sounds}`;
}

console.log(introduceLion(simba));
// "This lion is 10 years old and sounds like roar"

console.log(introduceLion(jerry));
// "This lion is 6 years old and sounds like meow"
```

`introduceLion`에는 Lion 타입인 simba만 받을수 있을 것 같지만 jerry도 들어가진다.

타입스크립트는 프로퍼티가 누락되었거나 잘못된 타입일 때 에러를 표시하지만 추가적인 프로퍼티가 포함되어 있어도 에러를 표시하지 않는다.

```tsx
function saveUser(user: { name: string; age: number }) {}

const user = { name: "Alex", age: 25, city: "Reykjavík" };
saveUser(user); // 타입 에러가 아님
```

이는 구조적 타입 시스템에서 의도된 동작이다.

Object.keys의 불안정한 사용법

```tsx
interface User {
  name: string;
  password: string;
}

const validators = {
  name: (name: string) => (name.length < 1 ? "Name must not be empty" : ""),
  password: (password: string) => (password.length < 6 ? "Password must be at least 6 characters" : ""),
};

function validateUser(user: User) {
  let error = "";
  for (const key of Object.keys(user)) {
    const validate = validators[key]; // 타입에러가 발생함
    error ||= validate(user[key]);
  }
  return error;
}

const user = {
  name: "Alex",
  password: "123456",
  email: "alex@example.com",
};
validateUser(user); // OK!
```

만약 타입에러를 주지 않는다면 ‘email’이 들어갔다면 validators[key] 가 undefined로 에러가 발생할 것이다.

즉, 타입 시스템이 인식하지 못하는 프로퍼티를 객체에 포함할 수 있다.
