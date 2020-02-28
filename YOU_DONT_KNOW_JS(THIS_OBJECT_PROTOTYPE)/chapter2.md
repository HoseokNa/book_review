# this가 이런 거로군!

* [2-1 호출부](#2-1-호출부)
* [2-2 단지 규칙일 뿐](#2-2-단지-규칙일-뿐)
* [2-3 모든 건 순서가 있는 법](#2-3-모든-건-순서가-있는-법)
* [2-4 바인딩 예외](#2-4-바인딩-예외)
* [2-5 Lexical this](#2-5-Lexical-this)
* [2-6 정리하기](#2-6-정리하기)

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

### 2-2-4 new 바인딩

사실 자바스크립트에서 new는 의미상 클래스 지향적인 기능과 아무 상관이 없다.

먼저 자바스크립트에서 '생성자'의 정의를 내려보자. 자바스크립트 생성자는 앞에 new 연산자가 있을 때 호출되는 일반 함수에 불과하다. 클래스에 붙은 것도 아니고 클래스 인스턴스화 기능도 없다. 단지 new를 사용하여 호출할 때 자동을로 실행되는 평범한 함수다.

예를 들어, 생성자 Number() 함수 ES5.1 명세를 보자.

##### 15.7.2 Number 생성자 : new 표현식의 일부로 호출 시 Number는 생성자이며 새로 만들어진 객체를 초기화 한다.

따라서 Number() 같은 부류의 내장 객체 함수는 물론이고 상당수의 옛 함수는 앞에 new를 붙여 호출할 수 있다. 그리고 이것은 결국 '생성자 호출'이나 다름 없다. 미묘한 차이를 잘 구분해야한다. '생성자 함수'가 아니라(실제로 이런 건 없다) '함수를 생성하는 호출' 이라고 해야 옳다.

함수 앞에 new를 붙여 생성자 호출을 하면 다음과 같은 일들이 발생한다.

1. 새 객체가 만들어진다.
2. 새로 생성된 객체의 [[Prototype]]이 연결된다.
3. 새로 생성된 객체는 해당 함수 호출 시 this로 바인딩 된다.
4. 이 함수가 자신의 또 다른 객체를 반환하지 않는 한 new와 함께 호출된 함수는 자동으로 새로 생성된 객체를 반환한다.

1,3,4 내용을 살펴보자.(2번은 5장에서 설명)

```js
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

앞에 new를 붙혀 foo()를 호출했고 새로 생성된 객체는 foo 호출 시 this에 바인딩 된다. 따라서 결국 new는 함수 호출 시 this를 새 객체와 바인딩 하는 방법이며 이것이 'new' 바인딩이다.

## 2-3 모든 건 순서가 있는 법

지금까지 함수를 호출할 때 4가지 this 바인딩 규칙을 알아봤다. 그런데 여러 개의 규칙이 중복으로 해당할 때는 어떻게 할까?

기본 바인딩은 가장 뒷순위다. 그럼 암시적 바인딩과 명시적 바인딩 중 어느 쪽이 우선일까?

```js
function foo() {
	console.log( this.a );
}

var obj1 = {
	a: 2,
	foo: foo
};

var obj2 = {
	a: 3,
	foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

결과는 명시적 바인딩이 암시적 바인딩보다 우선순위가 높다. 따라서 암시적 바인딩을 확인하기 전에 먼저 명시적 바인딩이 적용됐는지 반드시 살펴야 한다.

new 바인딩은 어떨까?

```js
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

new 바인딩이 암시적 바인딩보다 우선순위가 높다.

그럼 new 바인딩과 명시적 바인딩은? Function.prototype.bind()는 어떤 종류든 자체 this 바인딩을 무시하고 주어진 바인딩을 적용하여 하드 코딩된 새 래퍼 함수를 생성한다. 따라서 명시적 바인딩의 한 형태인 하드 코딩이 new 바인딩보다 우선순위가 높고 new로 오버라이드할 수 없다.

```js
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

앞에서 '가짜' bind 헬퍼 함수를 만들었는데 특성상 new 연산자를 써서 obj로 하드 바인딩 된 걸 오버라디으할 방도가 없다.

```js
function bind(fn, obj) {
	return function() {
		fn.apply( obj, arguments );
	};
}
```

그러나 ES5에 내장된 진짜 함수는 실제로 더 정교하다. 다음은 형태를 다듬은 bind() 폴리필이다.

```js
if (!Function.prototype.bind) {
	Function.prototype.bind = function(oThis) {
		if (typeof this !== "function") {
			// ECMAScript 5의 내부 함수 IsCallable을 최대한 비슷하게 흉내 낸 것
			throw new TypeError( "Function.prototype.bind - 바인딩 하려는 대상이 " +
				"호출 가능하지 않습니다."
			);
		}

		var aArgs = Array.prototype.slice.call( arguments, 1 ),
			fToBind = this,
			fNOP = function(){},
			fBound = function(){
				return fToBind.apply(
					(
						this instanceof fNOP &&
						oThis ? this : oThis
					),
					aArgs.concat( Array.prototype.slice.call( arguments ) )
				);
			}
		;

		fNOP.prototype = this.prototype;
		fBound.prototype = new fNOP();

		return fBound;
	};
}
```

new 오버라이드를 가능케 하는 코드는 이 부분이다.

```js
this instanceof fNOP &&
oThis ? this : oThis

// 중략

fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

하드 바인딩 함수가 new로 호출되어 this가 새로 생성된 객체로 세팅됐는지 조사해보고 맞으면 하드 바인딩에 의한 this를 버리고 새로 생성된 this를 대신 사용한다.

굳이 new로 하다 바인딩을 오버라이드하려는 이유는 뭘까? 기본적으로 this 하드 바인딩을 무시하는 (new로 객체를 생성할 수 있는) 함수를 생성하여 함수 인자를 전부 또는 일부만 미리 세팅해야 할 때 유용하다. bind() 함수는 최초 this 바인딩 이후 전달된 인자를 원 함수의 기본 인자로 고정하는 역할을 한다. ('Curring'의 일종)

```js
function foo(p1,p2) {
	this.val = p1 + p2;
}

// 'null'을 입력한 건 여기서 'this' 하드 바인딩은
// 어차피 'new' 호출 시 오버라이드되므로 신경 쓰지 않겠다는 의미다.
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```

### 2-3-1 this 확정 규칙

다음 항목을 순서대로 따져보고 그중 맞아떨어지는 최초 규칙을 적용한다.

1. new로 함수를 호출(new 바인딩) 했는가? -> 맞으면 새로 생성된 객체가 this다.
```js
var bar = new foo();
```

2. call과 apply로 함수를 호출(명시적 바인딩), 이를테면 bind 하드 바인딩 내부에 숨겨진 형태로 호출됐는가? -> 맞으면 명시적으로 지정된 객체가 this다.
```js
var bar = foo.call(obj2);
```

3. 함수를 콘텍스트(임시적 바인딩), 즉 객체르 ㄹ소유 또는 포함하는 형태로 호출됐는가? -> 맞으면 바로 이 콘텍스트 객체가 this다.
```js
var bar = obj1.foo();
```

그 외의 경우는 this는 기본값(염격 모드는 undefined, 비엄격모드는 전역 객체)으로 세팅된다(기본 바인딩)
```js
var bar = foo();
```

## 2-4 바인딩 예외

기본 바인딩 규칙이 적용되는 예외 사례를 보자.

### 2-4-1 this 무시

call, apply, bind 메서드에 첫 번째 인자로 null 또는 undefined를 넘기면 this 바인딩이 무시되고 기본 바인딩 규칙이 적용된다.

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```

그런데 왜 null 같은 값으로 this 바인딩을 하려는 걸까? apply()는 함수 호출 시 다수의 인자를 배열 값으로 즉 펼쳐 보내는 용도로 자주 쓰인다. bind()도 유사한 방법으로 인자들(미리 세팅된 값들)을 커링하는 메서드로 많이 사용한다.

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// 인자들을 배열 형태로 펼친다.
foo.apply( null, [2, 3] ); // a:2, b:3

// `bind(..)`로 커링한다.
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

apply와 bind 모두 반드시 첫 번째 인자로 this 바인딩을 지정해야 한다. 하지만 this가 로직 상 아무래도 좋다면 일종의 Placeholder 값으로 null 정도의 값을 전달하는 편이 합리적이다.

그러나 this 바인딩이 어찌 되든 상관없다고 null을 애용하는 건 약간의 '리스크'가 있다. 예를 들어 어떤 함수(여러분이 어찌할 수 없는 서트 파티 라이브러리 함수) 호출 시 null을 전달했는데 마침 그 함수가 내부적으로 this를 레퍼런스로 참조하면 기본 바인딩이 적용되어 전역 변수를 참조하거나 최악으로는 예기치 못한 일이 발생 할 수 있다.

#### 더 안전한 this

더 안전하게 가고자 한다면 프로그램에서 부작용과 100% 무관한 객체를 this로 바인딩 하는게 좋다. 업계의 용어로 표현하면 'DMZ(비무장지대)' 객체, 즉 내용이 하나도 없으면서 전혀 위임되지 않은(Nondelegated) 객체 정도가 필요하다.

이 DMZ 객체를 전달하면, 받는 쪽에서 this를 어찌 사용하든지 어차피 대상은 빈 객체로 한정되므로 최소한 전역 객체를 건드리는 부작용은 방지할 수 있다.

빈 객체를 만드는 가장 간단한 방법은 Object.create(null)이다(5장 프로토타입 참고). Object.create(null)은 {}와 비슷하나 Object.prototype으로 위임하지 않으므로 {} 보다 '더 텅 빈' 객체라고 볼 수 있다.

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// DMZ 객체 생성
var ø = Object.create( null );

// 인자들을 배열 형태로 펼친다.
foo.apply( ø, [2, 3] ); // a:2, b:3

// `bind(..)`로 커링한다.
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

기능적으로 '더 안전하다'는 의미 외에도 ø 처럼 표기하면 'this는 텅 빈 객체로 하겠다'는 의도를 null 보다 더 확실하게 밝히는 효과가 있다. 어쨋거나 DMZ 객체의 이름은 각자의 취향에 맡긴다.

### 2-4-2 간접 레퍼런스

한 가지 더 유의할 점은 간접 레퍼런스가 생성되는 경우로, 함수를 호출하면 무조건 기본 바인딩 규칙이 적용되어 버린다. 간접 레퍼런스는 할당문에서 가장 빈번하게 발생한다.

```js
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```

할당 표현식 p.foo = o.foo의 결괏값은 원 함수 객체의 레퍼런스이므로 실제로 호출부는 foo()다.

그리고 호출됨 함수 본문(호출부가 아니라)의 엄격모드에 따라 기본 바인딩 값이 달라진다는 걸 잊지 말자.

### 2-4-3 소프트 바인딩

하드 바인딩은 함수의 유연성을 크게 떨어뜨리기 때문에 this를 암시적 바인딩 하거나 나중에 다시 명시적 바인딩 하는 식으로 수동으로 오버라이드하는 것이 불가능하다.

암시적/명시적 바인딩 기법을 통해 임의로 this 바인딩을 하는 동시에 전역 객체나 undefined가 아닌 다른 기본 바인딩 값을 세팅하는 소프트 바인딩 유틸리티를 고안했다.

```js
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this,
			curried = [].slice.call( arguments, 1 ),
			bound = function bound() {
				return fn.apply(
					(!this ||
						(typeof window !== "undefined" &&
							this === window) ||
						(typeof global !== "undefined" &&
							this === global)
					) ? obj : this,
					curried.concat.apply( curried, arguments )
				);
			};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}
```

호출 시점에 this를 체크하는 부분에서 주어진 함수를 래핑하여 전역 객체나 undefined일 경우엔 미리 준비한 대체 기본 객체(obj)로 세팅한다. 그 외의 경우 this는 손대지 않는다. 그리고 선택적인 커링 기능도 있다.

```js
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   <---- 주목!!!

fooOBJ.call( obj3 ); // name: obj3   <---- 주목!

setTimeout( obj2.foo, 10 ); // name: obj   <---- 소프트 바인딩이 적용됐다.
```

소프트 바인딩이 탑재된 foo() 함수는 this를 obj2나 obj3으로 수동 바인딩 할 수 있고 기본 바인딩 규칙이 적용되어야 할 땐 다시 obj로 되돌린다.

## 2-5 Lexical this

ES6부터는 앞에 말한 4가지 규칙들을 따르지 않는 특별한 함수가 있다. 바로 화살표 함수다. 화살표 함수는 Enclosing Scope(함수 또는 전역)를 보고 this를 알아서 바인딩 한다.

다음은 화살표 함수의 렉시컬 스코프를 나타낸 예제다.

```js
function foo() {
	// 화살표 함수를 반환
	return (a) => {
		// 여기서 `this`는 lexical로 'foo()'에서 상속된다.
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 3이 아니다!
```

화살표 함수의 lexical 바인딩은 절대로 (심지어 new로도!) 오버라이드할 수 없다.

화살표 함수는 이벤트 처리기나 타이머 등의 콜백에 가장 널리 쓰인다.

```js
function foo() {
	setTimeout(() => {
		// 여기서 `this`는 lexical로 'foo()'에서 상속된다.
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

화살표 함수는 this를 확실히 보장하는 수단으로 bind()를 대체할 수 있는 것처럼 보인다. 그렇지만 결과적으로 더 잘 알려진 렉시컬 스코프를 쓰겠다고 기존의 this 체계를 포기하는 형국이란 점을 간과하면 안된다.

ES6 이전에도 화살표 함소의 기능과 크게 다르지 않은 패턴이 있었다.

```js
function foo() {
	var self = this; // `this` 렉시컬 적으로 포착한다.
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

"self = this" 나 화살표 함수 모두 bind() 대신 사용 가능한 좋은 해결책이지만 this를 제대로 이해하고 수용하기보단 골치 아픈 this에서 도망치려는 꼼수라고 할 수 있다.

어쨋거나 this 스타일의 코드를 작성해야 할 경우 어휘적 "self = this"든, 화살표 함수 꼼수든 꼭 다음 두가지 중 하나만 선택하자.

1. 오직 렉시컬 스코프만 사용하고 가식적인 this 스타일의 코드는 접어둔다.
2. 필요하면 bind() 까지 포함하여 완전한 this 스타일의 코드를 구사하되 self = this나 화살표 함수 같은 소위 'lexical this' 꼼수는 삼가야 한다.
* 두 스타일 모두 적절히 혼용하여 효율적인 프로그래밍을 할 수도 있겠지만 관리도 잘 안되고 이해하기 곤란한 골칫덩이 코드가 될 것이다.

## 2-6 정리하기

함수 실행에 있어서 this 바인딩은 함수의 직접적인 호출부에 따라 달라진다. 일단 호출부를 식별한 후 다음 4가지 규칙을 열거한 우선순위에 따라 적용한다.

1. new로 호출했다면 새로 생성된 객체로 바인딩 된다.
2. call이나 apply 또는 bind로 호출됐다면 주어진 객체로 바인딩 된다.
3. 호출의 주체인 콘텍스트 객체로 호출됐다면 바로 이 콘텍스트 객체로 바인딩 된다.
4. 기본 바인딩에서 엄격 모드는 undefined, 그 밖엔 전역 객체로 바인딩 된다.

실수로 또는 예기치 않게 기본 바인딩 규칙이 적용되는 경우를 조심해야 한다. this 바인딩을 안전하게 하고 싶으면 ø = Object.create( null ) 처럼 DMZ 객체를 자리 끼움 값으로 바꿔 넣어 뜻하지 않은 부수 효과가 전역 객체에서 발생하지 않게 한다.

ES6 화살표 함수는 표준 바인딩 규칙을 무시하고 렉시컬 스코프로 this를 바인딩 한다. 즉, enclosing 함수 호출로부터 어떤 값이든 this 바인딩을 상속한다. 이는 ES6 이전 시절 self = this 구문을 대체한 장치다.
