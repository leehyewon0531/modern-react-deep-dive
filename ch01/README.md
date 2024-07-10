# [1장] 리액트 개발을 위해 꼭 알아야할 자바스크립트

## 자바스크립트의 데이터 타입

**원시타입**

- boolean
- undefined
- number
- string
- symbol
- bigint

**객체 타입**

- object

<br/>
<br/>

**Falsy Value**
<br/>
<br/>
if 문에서 조건에 아래 값을 삽입할 경우 `False`로 판단

```
false
0 -0 0n 0x0n
NaN
'' "" ``
null
undefined
```

`주의⚠️`

```
[] {} // 얘네는 truthy value
```

<br/>
<br/>

**String**
<br/>
<br/>
' ' 또는 " " 또는 백틱(``)으로 사용 가능

```
			const longText = `
안녕하세요
` // 줄바꿈 가능

const hello = `hello ${'sung chul'}` // 문자열 내부 표현식 사용 가능
```

`주의⚠️`
<br/>
String은 원시 타입 → 즉, `변경 불가능`하다. 한 번 문자열이 생성되면 그 문자열을 변경할 수 없다!!!

```
let foo = "bar";

console.log(foo[0]); // b

foo[0] = "a";

console.log(foo[0]); // b
```

<br/>
<br/>

**Symbol**
<br/>
다른 값과 중복되지 않는 `유일무이한` 값, 충돌 위험이 없는 프로퍼티 키를 만들 때 사용
<br/>
<br/>
참고링크
<br/>
[Symbol을 사용하는 이유는 뭘까 | symbol usage](https://another-light.tistory.com/105)

<br/>
<br/>

**객체**
<br/>
7가지 원시 타입 이외의 모든 것 == `객체`
<br/>
함수도 객체가 될 수 있음

```
const hello1 = function(){}
const hello2 = function(){}

hello1 === hello2 // false, 참조가 다르기 때문
```

<br/>

**원시 타입과 객체 타입이 값을 저장하는 방식의 차이**

`원시 타입` → 불변 형태의 값으로 저장
<br/>
`객체` → 변경 가능한 형태로 저장, 값을 복사할 때도 값이 아닌 참조를 전달

```
let hello = "hello world"
let hi = hello
console.log(hi === hi) // true

------------------------

let hello = "hello world"
let hi = "hello world"
console.log(hello === hi) // true
```

```
const hello = {
  greet: "hello world",
};

const hi = {
  greet: "hello world",
};

console.log(hello === hi); // false

------------------------

const hello = {
  greet: "hello world",
};

const hi = hello // 참조를 전달

hello.greet = "hi world";

console.log(hello) // { greet: 'hi world' }
console.log(hi) // { greet: 'hi world' }
```

<br/>

**자바스크립트에서의 동등 비교**

`==` vs `Object.is`
<br/>
<br/>
`==`비교는 같음을 비교하기 전에 양쪽이 같은 타입이 아니라면 비교할 수 있도록 강제로 형변환(`type casting`)을 한 후에 비교

<br/>
<br/>

`===` vs `Object.is`

`===`의 몇가지 예외 상황에서 대해서 `Object.is`가 좀 더 개발자가 기대하는 방식으로 정확히 비교

<br/>

```
-0 === +0 // true
Object.is(-0, +0) // false

Number.NaN === NaN // false
Object.is(Number.NaN, NaN) // true
```

리액트에서 사용하는 동등 비교는 `Object.is`이고 이를 기반으로 동등 비교를 하는 `shallowEqual`이라는 함수를 만들어 사용한다.
<br/>
-> `Object.is`로 먼저 비교를 수행한 다음에 수행하지 못하는 비교는 `shallowEqual`을 한 번 더 수행한다.

```
Object.is({ hello: "world" }, { hello: "world" }); // false
shalloWEqual({ hello: "world" }, { hello: "world" }); // true
ShallowEqual({ hi: { hello: "world" } }, { hi: { hello: "world" } }); // false
```

<br/>

## 함수

```
function sum(a,b){
	return a + b
}
```

`일급 객체` : 다른 객체들에 일반적으로 적용 가능한 연산을 모두 지원하는 객체를 의미

-> 매개변수가 될 수도 있고, 반환값이 될 수도 있고, 할당도 가능

<br/>

**함수의 호이스팅**

함수 선언문이 마치 코드 맨 앞단에 작성된 것처럼 작동하는 자바스크립트의 특징

- 함수에 대한 선언을 실행 전에 `미리 메모리에 등록`하는 작업
- 함수 선언문이 미리 메모리에 등록됐고, 코드의 순서에 상관없이 정상적으로 함수 호출 가능

<br/>

**변수의 호이스팅**
<br/>
<br/>
`var`의 경우 호이스팅되는 시점에서 `undefined`로 초기화

```
console.log(typeof hello); // undefined

hello(); // TypeError: hello is not a function

var hello = function () {
  console.log("hello ");
};

hello(); // hello
```

<br/>
<br/>

**화살표 함수**

- `constructor` 사용 불가
- `arguments` 존재하지 않음
- `this` 바인딩
  - this는 함수가 어떻게 호출되느냐에 따라 동적으로 결정되어왔음
  - 일반 함수로 호출될 경우 전역 객체를 가리킴
  - 화살표 함수에서는 상위 스코프의 `this`를 그대로 따라감

<br/>

**즉시 실행 함수**

```
const a = (function (a, b) {
  return a + b;
})(10, 24);

console.log(a); // 34
```

<br/>

**함수를 만들 때 주의 사항**

- `side-effect` 억제
- 순수함수, 비순수함수
- 가능한 한 함수를 작게 만들어라
- 누구나 이해할 수 있는 이름을 붙여라

## 클래스

- `constructor`
  - 객체를 생성하는 데 사용하는 특수 메서드
  - 하나만 존재 가능
  - 생략 가능
- `프로퍼티`
  - 클래스로 인스턴스를 생성할 때 내부에 정의할 수 있는 속성값
- `getter`와 `setter`
  - `getter`는 클래스에서 어떤 값을 가져올 때 사용
  - `get`을 앞에 붙여야 하고 뒤에 이름 선언
  - `setter`는 클래스에 값을 할당할 때 사용
  - `set`을 먼저 선언하고 그 뒤에 이름을 붙임
- `인스턴스`
  - 클래스 내부에서 선언한 메서드 == 인스턴스 메서드
  - 직접 객체에서 선언하지 않았음에도 프로토타입에 있는 메서드를 찾아서 실행을 도와주는 것을 프로토타입 체이닝이라고 함
- `정적 메서드`
  - 클래스의 인스턴스가 아닌 이름으로 호출 가능한 메서드
  - `this` 사용 불가
  - 인스턴스 생성하지 않아도 사용가능
  - 객체를 생성하지 않더라도 여러 곳에서 재사용 가능
  - 앱 전역에서 사용하는 유틸 함수를 정적 메서드로 많이 활용

```
class Car {
  constructor(name) {
    this.name = name;
  }

  hello() {
    console.log(`안녕하세요. 저는 ${this.name}입니다.`);
  }

  static hello() {
    console.log("안녕하세요.");
  }
}
const myCar = new Car("자동차");
myCar.hello(); // 안녕하세요 저는 자동차입니다.
Car.hello(); // 안녕하세요.
```

<br/>

- `상속`
  - 기존 클래스 상속 받아 자식 클래스에서 상속받은 클래스를 기반으로 확장

```
class Car {
  constructor(name) {
    this.name = name;
  }

  honk() {
    console.log(`${this.name}이 경적을 울립니다.`);
  }
}

class Truck extends Car {
  constructor(name) {
    super(name);
  }

  load() {
    console.log(`${this.name}이 짐을 실었습니다.`);
  }
}
const myCar = new Car("자동차");
const myTruck = new Truck("트럭");

console.log(myCar.name, myTruck.name);
myTruck.honk();
myTruck.load();
```

## 클로저

함수와 함수가 `선언된 어휘적 환경`(`Lexical Scope`)의 조합
<br/>
선언된 어휘적 환경 : 변수가 코드 내부에서 어디서 선언됐는지

```
function add() {
  const a = 10;
  function innerAdd() {
    const b = 20;
    return a + b;
  }
  console.log(innerAdd()); // 30
}
```

`a` 유효 범위 : `add` 전체

`b `유효 범위 : `innerAdd` 전체

→ `innerAdd`에서 `a`에 접근 가능

`스코프` = 변수의 유효 범위

`전역 스코프` = 전역 레벨에 선언

`함수 스코프` = {} 블록이 스코프 범위를 결정하는 것이 아니라 함수 레벨 스코프를 따라감

가장 가까운 스코프에서 변수가 존재하는지를 먼저 확인

```
function outerFunction() {
  const x = "hello";
  function innerFunction() {
    console.log(x);
  }

  return innerFunction;
}

const innerFunction = outerFunction();
innerFunction(); // hello
```

```
function Counter() {
  var counter = 0;
  return {
    increase: function () {
      return ++counter;
    },
    decrease: function () {
      return --counter;
    },
    counter: function () {
      console.log("counter에 접근!");
      return counter;
    },
  };
}

const c = Counter();

console.log(c.increase()); // 1
console.log(c.increase()); // 2
console.log(c.counter()); // 2
```

## 이벤트 루프와 비동기 통신의 이해

`synchronous` - `동기적`(직렬) 방식으로 작업을 처리하는 것

`asynchronous` - `비동기적`(병렬) 방식으로 작업을 처리하는 것

응답이 오건 말건 상관없이 다음 작업이 이루어지며, 한 번에 여러 작업 실행 가능

`프로세스` - 프로그램을 구동해 프로그램의 상태가 메모리상에서 실행되는 작업 단위

`스레드` - 이보다 더 작은 단위

→ 하나의 프로세스에서는 `여러 개의 스레드` 생성 가능, 스레드끼리는 메모리를 공유할 수 있어 여러 가지 작업을 동시에 수행 가능

싱글 스레드 = 코드의 실행이 하나의 스레드에서 순차적으로 이루어지며 하나의 작업이 끝나기 전까지는 뒤이은 작업이 실행되지 않는다

<br/>

**이벤트 루프**

`호출 스택(call stack)` : 수행해야할 코드나 함수를 순차적으로 담아두는 스택

```
function bar() {
  console.log("bar");
}

function baz() {
  console.log("baz");
}

function foo() {
  console.log("foo");
  bar();
  baz();
}

foo(); // foo bar baz
```

```
function bar() {
  console.log("bar");
}

function baz() {
  console.log("baz");
}

function foo() {
  console.log("foo");
  setTimeout(bar, 0);
  baz();
}

foo(); // foo baz bar
```

`태스크 큐` : 실행해야 할 태스크의 집합(Set)

`이벤트 루프`는 `태스크 큐`를 한 개 이상 가지고 있다.

이벤트 루프는 호출 스택에 실행 중인 코드가 있는지, 그리고 태스크 큐에 대기 중인 함수가 있는지 반복해서 확인하는 역할을 수행

`호출 스택이 비었다면?` → 태스크 큐에 대기 중인 작업이 잇는지 확인하고 순차적으로 꺼내와서 실행

`setTimeout`, `fetch` 등의 작업은 누가 처리할 것인가?
<br/>
-> 태스크 큐가 할당되는 별도의 스레드에서 수행

코드의 실행은 `싱글 스레드`이지만 외부 Web API 등은 모두 자바스크립트 코드 외부에서 실행되고 콜백이 태스크 큐로 들어가는 것

**태스크 큐와 마이크로 태스크 큐**

마이크로 태스크 큐는 기존 태스크 큐보다 우선권을 가짐

`setTimeout`, `setInterval`은 Promise보다 늦게 실행됨

마이크로 태스크 큐가 빌 때까지는 기존 태스크 큐의 실행은 뒤로 미루어짐

```
function foo() {
  console.log("foo");
}

function bar() {
  console.log("bar");
}

function baz() {
  console.log("baz");
}

setTimeout(foo, 0);

Promise.resolve().then(bar).then(baz);

// bar baz foo
```

## React에서 자주 쓰는 javascript 문법

`구조 분해 할당` - 배열과 객체에서 사용

```
const array = [1, 2, 3, 4, 5];

const [first, second, ...rest] = array;

console.log(first); // 1
console.log(second); // 2
console.log(rest); // [3, 4, 5]
```

기본값 부여 → 반드시 `undefined`일 때만 기본값이 사용됨

```
const [a = 1, b = 1, c = 1, d = 1, e = 1] = [undefined, null, 0, ""];

console.log(a); // 1
console.log(b); // null
console.log(c); // 0
console.log(d); // ""
console.log(e); // 1
```

`객체 구조 분해 할당`

```
const object = {
  a: 1,
  b: 2,
  c: 3,
  d: 4,
  e: 5,
};

const { a, b, c, ...rest } = object;
console.log(a); // 1
console.log(b); // 2
console.log(c); // 3
console.log(rest); // { d: 4, e: 5 }
```

```
const object = {
  a: 1,
  b: 2,
};

const { a: first, b: second } = object;
console.log(first, second); // 1 2
```

변수에 있는 값으로 꺼내오는 이른바 `계산된 속성 이름 방식`도 가능

```
const key = "a";
const object = {
  a: 1,
  b: 1,
};

const { [key] : value } = object; // key에서 a라는 값을 꺼내옴

console.log(value); // 1
```

`전개 구문`

```
const arr1 = ["a", "b"];
const arr2 = [...arr1, "c", "d"];
console.log(arr2); // ["a", "b", "c", "d"]
```

```
const arr1 = ["a", "b"];
const arr2 = arr1;
console.log(arr1 === arr2); //true, 참조를 복사하기 때문


const arr1 = ["a", "b"];
const arr2 = [...arr1];
console.log(arr1 === arr2); // false, 값만 복사됐을 뿐, 참조는 다름
```

```
const obj = {
  a: 1,
  b: 1,
  c: 1,
  d: 1,
  e: 1,
};

const aObj = { ...obj, c: 10 }; // {a:1, b:1, c:10, d:1, e:1}
const bObj = { c: 10, ...obj }; // {c:1, a:1, b:1, d:1, e:1}

console.log(aObj, bObj);
```

<br/>
<br/>

Array 프로토타입 메서드 - `map`, `filter`, `reduce`, `forEach`

```
const arr = [1, 2, 3, 4, 5];
const doubledArr = arr.map((item) => item * 2);
console.log(doubledArr); // [2,4,6,8,10]
```

리액트에서는 주로 특정 배열을 기반으로 어떠한 리액트 요소를 반환하고자 할 때 `filter`를 많이 사용

```
const arr = [1, 2, 3, 4, 5];
const evenArr = arr.filter((item) => item % 2 === 0);
console.log(evenArr); // [2, 4]
```

`reduce` 메서드에서는 `reducer 콜백 함수`를 실행하고, 이를 초깃값에 누적하여 결과 반환

```
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce((result, item) => result + item, 0);
console.log(sum); // 15
```

`forEach`는 반환값이 없다. 에러를 던지거나 프로세스를 종료하지 않는 이상 멈출 수 없음

```
const arr = [1, 2, 3];

arr.forEach((item) => console.log(item)); // 1 2 3
```

<br/>
<br/>
<br/>

**삼항 조건 연산자**

`조건식` ? `true`일 때 결과 : `false`일 때 결과

형태로 사용

```
const value = 10;
const result = value % 2 === 0 ? "짝" : "홀";
console.log(result); // 짝
```

리액트에서는 JSX 내부에서 `조건부로 렌더링`하기 위해서 가장 널리 쓰이는 방법

<br/>

## 타입스크립트

**any 대신 unknown을 사용하자**

```
function doSomething(callback: any) {
  callback();
}

doSomething(1); // Error
```

위의 상황에서 타입스크립트는 에러를 발생시키지 않고 런타임에서 문제가 된다. 이는 타입스크립트의 이점을 없애 버림.

```
function doSomething(callback: any) {
  callback(); // 'callback' is of type 'unknown'
}
```

`callback`이 `unknown`, 즉 `아직 알 수 없는 값`이기 때문에 사용할 수 없다.

```
function doSomething(callback : unknown) {
  if (typeof callback === "function") {
    callback();
    return;
  }
  throw new Error("callback is not a function");
}
```

`never` : 어떠한 타입도 들어올 수 없음, 코드상으로 존재가 불가능한 타입을 나타낼 때 사용

<br/>

**타입 가드를 적극 활용하자**
<br/>
<br/>
`instanceof` : 지정한 인스턴스가 특정 클래스의 인스턴스인지 확인
<br/>
`typeof` : 특정 요소에 대한 자료형 확인

`in` : 특정 객체만 있는 프로퍼티 값을 확인

```
interface Student {
  age: number;
  score: number;
}

interface Teacher {
  name: string;
}

function doSchool(person: Student | Teacher) {
  if ("age" in person) {
    person.age // person은 Student
    person.score
  }
  if ("name" in person) {
    person.name // person은 Teacher
  }
}
```

<br/>
<br/>

**제네릭**

함수나 클래스 내부에서 단일 타입이 아닌 `다양한 타입에 대응`할 수 있도록 도와주는 도구
<br/>
`타입만 다른 비슷한 작업을 하는 컴포넌트`를 단일 제네릭 컴포넌트로 선언해 작성 가능

```
function getFirstAndLast<T>(list: T[]): [T, T] {
  return [list[0], list[list.length - 1]];
}

const [first, last] = getFirstAndLast([1, 2, 3, 4, 5]);
console.log(first, last); // 1 5

const [first1, last1] = getFirstAndLast(["a", "b", "c", "d", "e"]);
console.log(first1, last1); // a e
```

**인덱스 시그니처**

`객체의 키`를 정의하는 방식

```
type Hello = {
  [key : string] : string
}

const hello : Hello = {
  hello : 'hello',
  hi : 'hi,
}

hello['hi'] // hi
hello['안녕'] // undefined
```

`Record<Key, Value>`를 사용하면 객체의 타입에 각각 원하는 키와 값을 넣을 수 있다.

```
/// Record를 사용
type Hello = Record<'hello' | 'hi', string>

const hello : Hello = {
  hello : 'hello',
  hi : 'hi,
}

hello['hi'] // hi
hello['안녕'] // undefined
```
