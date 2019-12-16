# CHAPTER 1 타입

'타입'이란 자바스크립트 엔진, 개발자 모두에게 어떤 값을 다른 값과 분별할 수 있는, 고유한 내부 특성의 집합이다. 다시 말해, 기계(엔진)와 사람(개발자)이 42(숫자)란 값을 "42"(문자열)란 값과 다르게 취급한다면, 두 값은 타입(즉, 숫자와 문자열)이 서로 다르다.

* [1-1 타입, 그 실체를 이해하자](#1-1-타입-그-실체를-이해하자)
* [1-2 내장 타입](#1-2-내장-타입)

## 1-1 타입 그 실체를 이해하자

타입별로 내재된 특성을 제대로 알고 있어야 값을 다른 타입으로 변환하는 방법을 정확이 이해할 수 있다. 어떤 형태로든 거의 모든 자바스크립트 프로그램에서 강제변환이 일어나므로 타입을 확실하게 인지하고 사용하는 것이 중요하다.

## 1-2 내장 타입

자바스크립트에는 다음 7가지 내장 타입이 있다.
* null
* undefined
* boolean
* number
* string
* object
* symbol(ES6 부터)

값 타입은 typeof 연산자로 알 수 있다.
```js
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 42            === "number";    // true
typeof "42"          === "string";    // true
typeof { life: 42 }  === "object";    // true

// added in ES6!
typeof Symbol()      === "symbol";    // true
```

null은 object를 반환, 그래서 타입으로 null 값을 정확히 확인하려면 조건이 하나 더 필요하다.
```js
typeof null === "object"; // true

var a = null;

(!a && typeof a === "object"); // true
```

typeof가 반환하는 문자열은 하나 더 있다.
```js
typeof function a(){ /* .. */ } === "function"; // true
```

function은 object의 '하위 타입'이다. 구체적으로 설명하면 함수는 '호출 가능한 객체'이다.
사실 함수는 객체라서 유용하다. 무엇보다 함수에 프로퍼티를 둘 수 있다.

```js
function a(b,c) {
	/* .. */
}

a.length; // 2
```
함수 a는 인자 2 개(b, c)를 가지므로 '함수의 길이는'는 2다.

배열은 (키가 문자열인 객체와 반대로) 숫자 인덱스를 가지며, length 프로퍼티가 자동으로 관리되는 등의 추가 특성을 지닌, 객체의 '하위 타입'이라고 할 수 있다.

```js
typeof [1,2,3] === "object"; // true
```
