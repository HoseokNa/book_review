# this가 이런 거로군!

* [2-1 호출부](#2-1-호출부)
* [2-2 단지 규칙일 뿐](#2-2-단지-규칙일-뿐)

1장에서는 this가 알고 있었던 것과는 달리 호출부(함수가 어떻게 호출됐는가?)에서 함수를 호출할 때 바인딩 된다는 사실을 이야기했다.

## 2-1 호출부

this 바인딩의 개념을 이해하려면 먼저 호출부, 즉 함수 호출(선언이 아니다) 코드부터 확인하고 'this가 가리키는 것'이 무엇인지 찾아봐야 한다.

호출부는 '함수를 호출한 지점'으로 돌아가면 금세 확인할 수 있을 것 같지만 패턴에 따라 '진짜' 호출부가 어디인지 모호할 때가 많다. 중요한 건 호출 스택을 생각하는 것이다. 이 중 호출부는 현재 실행 중인 함수 '직전'의 호출 코드 '내부'에 있다.

호출부와 호출 스택을 설명하기 위한 예제다.

```js
function baz() {
    // 호출 스택 : `baz`
    // 따라서 호출부는 전역 스코프 내부다.

    console.log( "baz" );
    bar(); // `bar`의 호출 부
}

function bar() {
    // 호출 스택 : `baz` -> `bar`
    // 그래서 호출부는 `baz` 내부다

    console.log( "bar" );
    foo(); // <-- `foo`의 호출부
}

function foo() {
    // 호출 스택 : `baz` -> `bar` -> `foo`
    // 그래서 호출부는 `bar` 내부다

    console.log( "foo" );
}

baz(); // <-- `baz`의 호출부
```

this 바인딩은 오직 호출부와 연관되기 때문에 호출 스택에서 진자 호출부를 찾아내려면 코드를 주의 깊게 잘 봐야 한다.

## 2-2 단지 규칙일 뿐

함수가 실행되는 동안 this가 무엇을 참조할지를 호출부가 어떻게 결정하는지 알아보자.

호출부를 살펴보고 다음에 열거할 4가지 규칙 중 어느 것이 해당되는지 확인한다. 중복될 경우 우선순위도 제시해주겠다.

### 2-2-1 기본 바인딩

첫째로 가장 평범한 함수 호출인 '단독 함수 실행'에 관한 규칙이다. 나머지 규칙에 해당하지 않을 경우 적용되는 this의 기본(Default) 규칙이다.

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo(); // 2
```

전역 스코프에 변수를 선언하면 변수명과 같인 이름의 전역 객체 프로퍼티가 생성된다. 이는 서로의 사본이 아니고 같은 동전의 앞뒷면이라고 보면 된다.

그리고 foo() 함수 호출 시 this.a는 전역 객체 a다. 기본 바인딩이 적용되어 this는 전역 객체를 참조한다.

foo()는 지극ㄱ히 평범한 있는 그대로의 함수 레퍼런스로 호출했다. 나머지 규칙을 논할 필요도 없이 기본 바인딩이 그대로 적용됐다.

엄격 모드(Strict Mode)에서는 전역 객체가 비본 바인딩 대상에서 제외된다. 그래서 this는 undefined가 된다.

```js
function foo() {
	"use strict";

	console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```

미묘하지만 중요한 사실은 보통 this 바인딩 규칙은 오로지 호출부에 의해 좌우되지만 비엄격 모드에서 foo() 함수의 본문을 실행하면 전역 객체만이 기본 바딩딩의 유일한 대상이라는 점이다. foo() 호출부의 어격 모드 여부는 상관없다.

```js
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```

### 2-2-2 암시적 바인딩

두 번째 규칙은 호출부에 콘텍스트 객체가 있는지, 즉 객체의 소유(Owing)/포함(Containing) 여부를 확인하는 것이다.

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

obj.foo(); // 2
```

obj 객체가 이 함수를 정말로 '소유'하거나'포함'한 것은 아니다. 그러나 호출부는 obj 콘텍스트로 foo()를 참조하므로 obj 객체는 함수 호출 시점에 함수의 레퍼런스를 '소유'하거나 '포함'한다고 볼 수 있다.

foo() 호출 시점에 이미 obj 객체 레퍼런스는 준비된 상태다. 이렇게 함수 레퍼런스에 대한 콘텍스트 객체가 존재할 때 암시적 바인딩 규칙을 따르고 바로 이 콘텍스트 객체가 함수 호출 시 this에 바인딩 된다. foo() 호출 시 obj는 this이니 this.a는 obj.a가 된다.

다음 예제처럼 객체 프로퍼티 참조가 체이닝 된 형태라면 최상위/최하위 수준의 정보만 호출부와 연관된다.

```js
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### 암시적 소실

'암시적으로 바인딩 된' 함수에서 바인딩이 소실되는 경우가 있는데 this 바인딩이 뜻밖에 헷갈리기 쉬운 경우다. 엄격 모드 여부에 따라 전역 객체나 undefined 중 한 가지로 기본 바인딩 된다.

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // function reference/alias!

var a = "엥? 전역이네"; // `a` 역시 전역 객체의 프로퍼티

bar(); // "엥? 전역이네"
```

bar는 obj의 foo를 참조하는 변수처럼 보이지만 실은 foo를 직접 가리키는 또 다른 레퍼런스다. 게다가 호출부에서 그냥 평범하게 bar()를 호출하므로 기본 바인딩이 적용된다.

콜백 함수를 전달하는 경우에는 예상외의 결과가 나온다.

```js
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn`은 'foo'의 또 다른 reference 일 뿐.

	fn(); // <-- 호출부
}

var obj = {
	a: 2,
	foo: foo
};

var a = "엥? 전역쓰"; // `a` 역시 전역 객체의 프로퍼티

doFoo( obj.foo ); // "엥? 전역쓰"
```

인자로 던달하는 건 일종의 암시적인 할당이다. 따라서 예제처럼 함수를 인자로 넘기면 암시적으로 레퍼런스가 할당되어 이전 예제와 결과가 같다.

콜백을 받아 처리하는 함수가 내장 함수면? 그래도 마찬가지다.

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var a = "엥? 전역이네"; // `a` 역시 전역 객체의 프로퍼티

setTimeout( obj.foo, 100 ); // "엥? 전역이네"
```

setTimeout() 함수의 이론적인 구현체는 대략 다음과 같은 형태일 것이다.

```js
function setTimeout(fn,delay) {
	// `delay` milliseconds 동안 기다린다.
	fn(); // <-- 호출부!
}
```

이처럼 콜백과정에서 this 바인딩의 행방이 묘연해지는 경우가 많다.

어떤 까닭으로 예기치 않게 this가 바뀌게 됐든 여러분이나 나나 콜백 함수의 레퍼런스를 마음대로 통제할 수 없다. 바로 뒷부분에서 this를 '고정'해 이 문제를 해결하는 방법을 소개한다.

### 2-2-3 명시적 바인딩

그런데 함수 레퍼런스 프로퍼티를 객체에 더하지 않고 어떤 객체를 this 바인딩에 이용하겠다는 의지를 코드에 명확히 밝힐 방도는 없을까?

이럴 때 call()과 apply() 메서드가 아주 적당하다.

두 메서드는 this에 바인딩 할 객체를 첫째 인자로 받아 함수 호출 시 이 객체를 this로 세팅한다. this를 지정한 객체로 직접 바인딩 하므로 이를 '명시적 바인딩'이라 한다.

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

foo.call()에서 명시적으로 바인딩 하여 함수를 호출하므로 this는 반드시 obj가 된다.

객체 대신 단순 원시 값(문자열, 불리언, 숫자)을 인자로 전달하면 원시 값에 대응되는 객체(new String(), new Boolean(), new Number())로 래핑 된다.

Note: this 바인딩 기능만 따지면 call()과 apply()는 같은 메서드다.

하지만 아쉽게도 이렇게 명시적으로 바이딩 해도 앞에서 언급한 this 바인딩이 도중에 소실되거나 프레임워크가 임의로 덮어써 버리는 문제는 해결할 수 없다.

#### 하드 바인딩

명시적 바인딩을 약간 변형한 꼼수가 있다.

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

var bar = function() {
	foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// 하드 바인딩 된 'bar'에서 재정의된 'this'는 의미 없다.
bar.call( window ); // 2
```

bar를 어떻게 호출하든 이 함수는 항상 obj를 바인딩 하여 foo를 실행한다. 이런 방인딩을 '하드 바인딩'이라고 한다.

하드 바인딩으로 함수를 감싸는 형태의 코드는 다음과 같이 인자를 넘기고 반환 값을 돌려받는 창구가 필요할 때 주로 쓰인다.

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = function() {
	return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

재사용 가능한 헬퍼 함수를 쓰는 것도 같은 패턴이다.

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

// 간단한 `bind` 헬퍼
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

var obj = {
	a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

하드 바인딩은 매우 자주 쓰는 패턴이여서 ES5 내장 유틸리티 Function.prototype.bind 역시 다음과 같이 구현되어 있다.

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

bind()는 주어진 콘텍스트로 원본 함수를 호출하도록 하드 코딩된 새 함수를 반환한다.

#### API 호출 콘텍스트

많은 라이브러리 함수와 자바스크립트 언어 및 호스트 환경에 내장된 여러 새로운 함수는 대개 '콘텍스트'라 불리는 선택적인 인자를 제공한다. 이는 bind()를 써서 콜백 함수의 this를 지정할 수 없는 경우를 대비한 일종의 예비책이다.

예를 들어, 다음과 같은 함수는 편의상 내부적으로 call()이나 apply()로 명시적 바인딩을 대신해준다.

```js
function foo(el) {
	console.log( el, this.id );
}

var obj = {
	id: "awesome"
};

// 'foo()' 호출 시 'obj'를 'this'로 사용한다.
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```

