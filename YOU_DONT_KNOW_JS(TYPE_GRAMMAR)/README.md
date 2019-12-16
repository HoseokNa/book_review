# CHAPTER 1 타입

'타입'이란 자바스크립트 엔진, 개발자 모두에게 어떤 값을 다른 값과 분별할 수 있는, 고유한 내부 특성의 집합이다. 다시 말해, 기계(엔진)와 사람(개발자)이 42(숫자)란 값을 "42"(문자열)란 값과 다르게 취급한다면, 두 값은 타입(즉, 숫자와 문자열)이 서로 다르다.

* [1-1 타입, 그 실체를 이해하자](#1-1-타입-그-실체를-이해하자)
* [1-2 내장 타입](#1-2-내장-타입)
* [1-3 값은 타입을 가진다](#1-3-값은-타입을-가진다)
* [1-4 정리하기](#1-4-정리하기)

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

## 1-3 값은 타입을 가진다

값에는 타입이 있지만, 변수엔 따로 타입이란 없다.
자바스크립트는 '타입 강제'를 하지 않는다. 변숫값이 처음에 할당된 값과 동일한 타입일 필요는 없다. 문자열을 넣었다가 나중에 숫자를 넣어도 상관 없다.
변수에 typeof 연산자를 대어보는 건 "이 변수에 들어있는 값의 타입은 무엇이니?"라고 묻는 것이다.

```js
var a = 42;
typeof a; // "number"

a = true;
typeof a; // "boolean"

typeof typeof 42; // "string"
```

typeof 연산자의 반환 값은 언제나 문자열이다.

### 1-3-1 Undefined vs Undeclared

값이 없는 변수의 값은 undefined이며, typeof 결과는 "undefined"다.

```js
var a;

typeof a; // "undefined"

var b = 42;
var c;

// 그리고 나서,
b = c;

typeof b; // "undefined"
typeof c; // "undefined"
```

"undefined"(값이 없는)와 "undeclared"(선언되지 않은)를 동의어처럼 생각하기 쉬운데, 자바스크립트에서 둘은 완전히 다른 개념이다.
"undefined" : 접근 가능한 스코프에 변수가 선언되었으나 현재 아무런 값도 할당되지 않은 상태
"undeclared" : 접근 가능한 스코프에 변수 자체가 선언조차 되지 않은 상태

```js
var a;

a; // undefined
b; // ReferenceError: b is not defined
```

"b is no defined"는 undefined가 아닌 undeclared를 의미.

```js
var a;

typeof a; // "undefined"

typeof b; // "undefined"
```

"undeclared"(선언되지 않은) 변수도 typeof 하면 "undefined"로 나온다.
b는 분명 선언조차 하지 않은 변수인데 typeof b를 해도 브라우저는 오류 처리를 하지 않는다. 바로 이것이 typeof만의 독특한 안전 가드(safe guard)다.

### 1-3-2 typeof Undeclared

여러 스크립트 파일의 변수들이 전역 네임스페이스를 공유할 때, typeof의 안전 가드는 의외로 쓸모가 있다.
다른 애플리케이션 코드에서 ReferenceError가 나지 않게 조심해서 전역 변수를 체크할 때 typeof를 활용할 수가 있다.

```js
// 선언되지 않은 DEBUG 변수 에러 발생
if (DEBUG) {
	console.log( "Debugging is starting" );
}

// 이렇게 해야 안전하게 존재 여부를 체크할 수 있다.
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" );
}
```

(DEBUG) 같은 임의로 정의한 변수를 쓰지 않더라도 이런 식으로 체크하는 것이 편리하며, 내장 API 기능을 체크할 떄에도 에러가 나지 않게 도와준다.

```js
if (typeof atob === "undefined") {
	atob = function() { /*..*/ };
}

// 호이스팅 때문에 atob 선언문에서 var 키워드를 뺄 것.
```

다른 방법으로 전역 변수가 모두 전역 객체(브라우저는 window)의 프로퍼티라는 점을 이용하는 방법.
하지만 가급적 삼가할 것. 특히 전역 변수를 꼭 window 객체로만 호출하지 않는 다중 자바스크립트 환경(ex node.js)이라면 더욱 그렇다.

```js
if (window.DEBUG) {
	// ..
}

if (!window.atob) {
	// ..
}
```

엄밀히 말해서 typeof 안전 가드는 (비록 일반적인 경우는 아니지만) 전역 변수를 사용하지 않을 때에도 유용하다. 이를테면 다른 개발자가 여러분이 작성한 유틸리티 함수를 자신의 모듈/프로그램에 카피 앤 페이스트하여 사용하는데, 가져다 쓰는 프로그래멩 유틸리티의 특정 변숫값이 정의되어 있는지 체크해야 하는 상황을 가정해보자.

```js
function doSomethingCool() {
	var helper =
		(typeof FeatureXYZ !== "undefined") ?
		FeatureXYZ :
		function() { /*.. default FeatureXYZ ..*/ };

	var val = helper();
	// ..
}
```

doSomethingCool 함수는 FeatureXYZ 변수가 있으면 그대로 사용하고 없으면 함수 바디를 정의한다.

```js
// IIFE
(function(){
	function FeatureXYZ() { /*.. my XYZ feature ..*/ }

	// `doSomethingCool`를 포함
	function doSomethingCool() {
		var helper =
			(typeof FeatureXYZ !== "undefined") ?
			FeatureXYZ :
			function() { /*.. default feature ..*/ };

		var val = helper();
		// ..
	}

	doSomethingCool();
})();
```

'Dependecy Injection' 설계 패턴 방법

```js
function doSomethingCool(FeatureXYZ) {
	var helper = FeatureXYZ ||
		function() { /*.. default FeatureXYZ ..*/ };

	var val = helper();
	// ..
}
```

다양한 설계 옵션이 가능하지만 대체로 typeof 안전 가드가 선택할 수 있는 옵션이 많아서 좋다.

## 1-4 정리하기

1. 자바스크립트에는 7가지 내장 타입(null, undefined, boolean, number, string, object, symbol)이 있으며, typeof 연산자로 타입명을 알아낸다.

2. 변수는 타입이 없지만 값은 타입이 있고, 타입은 값의 내재된 특성을 정의한다.

3. 자바스크립트 엔진은 "undefined"와 "undeclared"를 다르게 취급하고 "undefined"는 선언된 변수에 할당할 수 있는 값이지만, "undeclared"는 변수 자체가 선언된 적이 없음을 나타낸다.

4. 불행히도 자바스크립트는 이 두 용어를 대충 섞어버려, 에러 메시지("ReferenceError: a is not defined")뿐만 아니라 typeof 반환 값도 모두 "undefined"로 뭉뚱그린다.

5. 그래도 (에러를 내지 않는) typeof 안전 가드 덕분에 선언되지 않는 변수에 사용하면 제법 쓸 만한다.
