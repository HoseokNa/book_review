# this or That?

* [1-1 this를 왜?](#1-1-this를-왜?)
* [1-2 헷갈리는 것들](#1-2-헷갈리는-것들)
* [1-3 this는 무엇인가?](#1-3-this는-무엇인가?)
* [1-4 정리하기](#1-4-정리하기)

## 1-1 this를 왜?

'어떻게' 이전에 '왜' 부터 따져보자.

뒤에 다 설명하기 때문에 다음 코드가 어떻게 동작하는지 혼란스러워도 걱정말고 "왜"에 집중하자.

identify()와 speak() 두 함수는 객체별로 따로따로 함수를 작성할 필요 없이 다중 콘텍스트 객체인 me와 you 모두에서 재사용할 수 있다.

```js
function identify() {
	return this.name.toUpperCase();
}

function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```

this를 안쓰고 identify()와 speak() 함수에 콘텍스트 객체를 명시할 수도 있다.

```js
function identify(context) {
	return context.name.toUpperCase();
}

function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```

하지만 암시적인 객체 레퍼런스를 '함께 넘기는(Passing Along)" this 체계가 API 설계상 좀 더 깔끔하고 명확하며 재사용하기 쉽다.

사용 패턴이 복잡해질수록 보통 명시적인 인자로 콘텍스트를 넘기는 방법이 this 콘텍스트를 사용하는 것보다 코드가 더 지저분해진다. 뒷부분에서 프로토타입을 배우고 나면 여러 함수가 적절한 콘텍스트 객체를 자동 참조하는 구조가 얼마나 편리한지 실감하게 될 것이다.

## 1-2 헷갈리는 것들

'this'란 이름 자체가 글자 그대로만 생각하게 해 헷갈리게 하는 구석이 있다. 사람들은 보통 두 가지 의미로 해석하는데 결론부터 미리 말하면 둘 다 틀렸다.

### 1-2-1 자기 자신

우선 this가 함수 그 자체를 가리킨다는 오해다. 재귀 로직이 들어가는 경우도 있고 최초 호출 시 이벤트에 바인딩 된 함수 자신을 언바인딩 할 때도 자기 참조가 필요하다.

자바스크립트가 생소한 개발자는 보통 함수를 객체(자바스크립트의 모든 함수는 객체다!)로 참조함으로써 함수 호출 간 '상태State'(프로퍼티 값)를 저장할 수 있을 거로 생각한다. 분명 그렇게 할 수 있고 제한적이나마 가능하다. 하지만 함수 객체 말고도 더 좋은 장소에 상태를 저장하는 패턴에 대해서는 이 책의 나머지 부분에서 설명한다.

우선 함수가 this로 자기 참조를 할 수 없다는 걸 증명하겠다. 다음은 함수(foo) 호출 횟수를 추적하는 예제다.

```js
function foo(num) {
	console.log( "foo: " + num );

	// 'foo'가 몇번 호출 됐는지 추적한다.
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// 'foo'는 몇 번 호출 됐을까?
console.log( foo.count ); // 0 -- ???
```

this.count++에서의 this를 너무 글자 그대로 해석하면 헷갈릴 수 있다.

foo.count = 0을 하면 foo라는 함수 객체에 count 프로퍼티가 추가된다. 하지만 this.count에서 this는 함수 객체를 바라보는 것이 아니며, 프로퍼티 명이 똑같아 헷갈리지만 근거지를 둔 객체 자체가 다르다.

많은 개발자가 "왜 this 참조가 이상하게 이루어졌을까?"라는 중요한 질문에 답을 찾지 않고 피해간다.

```js
function foo(num) {
	console.log( "foo: " + num );

	// 'foo'가 몇번 호출 됐는지 추적한다.
	data.count++;
}

var data = {
	count: 0
};

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// 'foo'는 몇 번 호출 됐을까?
console.log( data.count ); // 4
```

이렇게 해결할 수 있지만 아쉽게도 this가 뭔지, 작동 원리는 무엇인지 모르는 채 문제의 본질을 벗어나 렉시컬 스코프라는 편리한 장치에 몸을 내맡긴 꼴이다.(렉시컬 스코프를 사용하지 말라는게 아니다)

함수가 내부에서 자신을 참조할 때 일반적으로 this만으로는 부족하며 렉시컬 식별자를 거쳐 함수 객체를 참조한다.

다음 두 함수를 보자.

```js
function foo() {
	foo.count = 4; // `foo`는 자기 자신을 가리킨다.
}

setTimeout( function(){
	// 이름이 없는 익명 함수는 자기 자신을 가리킬 방법이 없다.
}, 10 );
```

'Named Function'이라 불리는 foo 함수는 foo라는 함수명 자체가 내부에서 자신을 가리키는 레퍼런스로 쓰인다. 하지만 setTimeout()에 콜백으로 전달한 함수는 이름 식별자가 없으므로 함수 자신을 참조할 방법이 마땅치 않다.

NOTE: arguments.callee도 실행 중인 함수 객체를 가리키지만 지금은 Deprecated 레퍼런스이니 더는 쓰지 말자.

앞 예제에서 this 없이 함수 객체 레퍼런스로 foo 식별자를 대신 사용해도 문제없이 작동한다.

```js
function foo(num) {
	console.log( "foo: " + num );

	// 'foo'가 몇번 호출 됐는지 추적한다.
	foo.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// 'foo'는 몇 번 호출 됐을까?
console.log( foo.count ); // 4
```

그러나 이 역시 this를 제대로 이해하지 않은 채 문제를 회피하여 foo 렉시컬 스코프에 의존하기는 마찬가지다.

foo 함수 객체를 직접 가리키도록 강제하는 것도 방법이다.

```js
function foo(num) {
	console.log( "foo: " + num );

	// 'foo'가 몇번 호출 됐는지 추적한다.
	// Note: 'this'는 'foo'를 어떻게 호출하느냐에 따라 진짜 'foo'가 된다.
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		// `call()` 함수로 호출하므로 'this'는 이제 확실히 함수 객체 'foo' 자신을 가리킨다.
		foo.call( foo, i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// 'foo'는 몇 번 호출 됐을까?
console.log( foo.count ); // 4
```

this를 피하지 않고 그대로 적용했다. 지금까지 열거한 기법들에 대해서는 곧 자세히 설명하겠다.

### 1-2-2 자신의 스코프

this가 바로 함수의 스코프를 가리킨다는 말은 아주 흔한 오해다. 어떤 면에선 맞지만 잘못 이해한 것인데 사실 까다로운 문제다.

분명한 건 this는 어떤 식으로도 함수의 렉시컬 스코프를 참조하지 않는다는 사실! 내부적으로 스코프는 별개의 식별자가 달린 프로퍼티로 구성된 객체의 일종이나 스코프 '객체'는 자바스크립트 구현체인 '엔진'의 내부 부품이기 때문에 일반 자바스크립트 코드로는 접근하지 못한다.

넘지 말아야 할 선을 넘어 this가 암시적으로 함수의 렉시컬 스코프를 가리키도록 해보자(물론 실패한다!).

```js
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); // ReferenceError: a is not defined
```

이 코드는 여러 곳에서 실수를 했다.

bar() 함수를 this.bar()로 참조하려고 한 것부터가 문제다. 곧 설명하겠지만 여기서 에러가 나지 않더라도 우연일 뿐이다. bar() 앞의 this를 빼고 식별자를 어휘적으로 참조하는 것이 가장 자연스러운 호출 방법이다.

물론 이 코드의 작성자는 foo()와 bar()의 렉시컬 스코프 사이에 어떤 연결 통로를 만들어 변수 a에 접근하게 하고 싶었을 것이다. 그러나 그런 연결 통로는 없다. 렉시컬 스코프 안에 있는 뭔가를 this 레퍼런스로 참조하기란 애당초 가능하지 않다.

this와 렉시컬 스코프 참조가 헷갈리는 독자는 연결 통로 따위는 없다는 것을 인지하자.

## 1-3 this는 무엇인가?

this는 작성 시점이 아닌 런타임 시점에 바인딩 되며 함수 호출 당시 상황에 따라 콘텍스트가 결정된다고 말했다. 함수 선언 위치와 상관없이 this 바인딩은 오로지 어떻게 함수를 호출했느냐에 따라 정해진다.

어떤 함수를 호출하면 활성화 레코드, 즉 실행 콘텍스트가 만들어진다. 여기엔 함수가 호출된 근원(콜스택)과 호출 방법, 전달된 인자 등의 정보가 담겨 있다. this 레퍼런스는 그중 하나로, 함수가 실행되는 동안 이용할 수 있다.

## 1-4 정리하기

this가 함수 자신이나 함수의 렉시컬 스코프를 가리키는 레퍼런스가 아니라는 점을 분명히 인지해야 한다.

this는 실제로 함수 호출 시점에 바인딩 되며 무엇을 가리킬지는 전적으로 함수를 호출한 코드에 달렸다.
