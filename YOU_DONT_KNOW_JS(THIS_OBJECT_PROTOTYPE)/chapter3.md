# 객체

* [3-1 구문](#3-1-구문)
* [3-2 타입](#3-2-타입)
* [3-3 내용](#3-3-내용)
* [3-4 순회](#3-4-순회)
* [3-5 정리하기](#3-5-정리하기)

다양한 객체를 가리키는 this 바인딩의 원리를 확인했었다. 그럼 객체는 무엇이고 왜 객체를 가리켜야 할까?

## 3-1 구문

객체는 선언적(리터럴) 형식과 생성자 형식, 두 가지로 정의한다.

리터럴 형식은 다음과 같다.

```js
var myObj = {
	key: value
	// ...
};
```

생성자 형식.

```js
var myObj = new Object();
myObj.key = value;
```

두 형식 모두 결과적으로 생성되는 객체는 같다. 차이점은 리터럴 형식은 한 번의 선언으로 다수의 키/값 쌍을 프로퍼티로 추가할 수 있지만, 생성자 형식은 한 번에 한 프로퍼티만 추가할 수 있다.

## 3-2 타입

자바스크립트 객체의 7개의 주요 타입은 다음과 같다.

* null
* undefined
* boolean
* number
* string
* object
* symbol(ES6에서 추가)

'단순 원시 타입'은 객체가 아니다. 가끔 null의 타입이 객체라고 잘못 알고 있는 사람들이 있는데, 이는 tpyeof null을 하면 'object'라는 문자열이 반환된 것에 비롯된 오해다. "자바스크립트는 모든 것이 객체다" 라는 말도 옳지 않다.

반면 '복합 원시 타입'이라는 독특한 객체 하위 타입이 있다. function은 객체(정확히는 호출 가능한 객체)의 하위 타입이다. 자바스크립트 함수는 기본적으로는 (호출 가능한 특성이 고정된) 객체이므로 '일급(First Class)'이며 여타의 일반 객체와 똑같이 취급된다. 배열 역시 추가 기능이 구현된 객체의 일종이다.

## 3-2-1 내장 객체

내장 객체라고 부르는 객체 하위 타입도 있다.

* String
* Number
* Boolean
* Object
* Function
* Array
* Date
* RegExp
* Error

내장 객체는 진짜 타입처럼 보이는데다 클래스처럼 느껴진다. 그러나 이들은 단지 자바스크립트의 내장 함수일 뿐 각각 생성자로 사용되어 주어진 하위 타입의 새 객체를 생성한다.

```js
var strPrimitive = "나는 문자열이야!";
typeof strPrimitive;							// "string"
strPrimitive instanceof String;					// false

var strObject = new String( "나는 문자열이야!" );
typeof strObject; 								// "object"
strObject instanceof String;					// true

// 객체 하위 타입을 확인한다.
Object.prototype.toString.call( strObject );	// [object String]
```

toString() 메서드의 기본 구현체를 빌려서 내부 하위 타입을 조사한다. 그 결과 strObject가 String 생성자에 의해 만들어진 객체임을 알 수 있다.

"나는 문자열이야!"라는 원시 값은 객체가 아닌 원시 리터럴이며 불변값이다. 문자 개수를 세는 등 문자별로 접근할 때엔 String 객체가 필요하다.

다행히도 자바스크립트 엔진은 상황에 맞게 문자열 원시 값을 String 객체로 자동 강제변환하므로 명시적으로 객체를 생성할 일은 거의 없다. 여러 커뮤니티에서도 되도록 생성자 형식보다 리터럴 형식을 사용하라고 적극 권장한다.

```js
var strPrimitive = "나는 문자열이야!";

console.log( strPrimitive.length );			// 13

console.log( strPrimitive.charAt( 3 ) );	// "m"
```

문자열 원시 값에 대해 프로퍼티/메서드를 호출하면 자바스크립트 엔진은 원시 값을 자동으로 String 객체로 강제변환하여 메서드 접근이 가능하게 도와준다.

숫자나 불리언 값도 마찬가지다.

객체 래퍼 형식이 없는 null과 undefined는 그 자체로 유일 값이다. 반대로 Date 값은 리터럴 형식이 없어서 반드시 생성자 형식으로 생성해야 한다.

Objects, Arrays, Functions, RegExps 는 형식(리터럴/생성자)과 무관하게 모두 객체다. 어느 쪽이든 결국 생성되는 객체는 같으므로 좀 더 간단한 리터럴 형식을 더 많이 쓴다. 추가 옵션이 필요한 경우에만 생성자 형식을 사용하자.

Error 객체는 예외가 던져지면 알아서 생성되니 명시적으로 생성할 일은 드물다. 생성자 형식 new Error()로 생성은 가능하지만 거의 쓸 일이 없다.

## 3-3 내용

객체는 특정한 위치에 저장된 모든 타입의 값, 즉 프로퍼티로 내용(Contents)이 채워진다고 했다. 엔진이 값을 저장하는 방식은 구현 의존적인데, 이는 객체 컨테이너에 담지 않는게 일반적이다. 객체 컨테이너에는 실제로 프로퍼티 값이 있는 곳을 가리키는 포인터(레퍼런스) 역할을 담당하는 프로퍼티명이 담겨 있다.

```js
var myObject = {
	a: 2
};

myObject.a;		// 2

myObject["a"];	// 2
```

myObject 객체에서 a 위치의 값에 접근하려면 '.' 연산자 또는 '[]'연산자를 사용한다. .a 구문을 프로퍼티 접근, ["a"] 구문을 '키 접근'이라고 한다. 같은 위치에 접근하여 같은 값 2를 조회하므로 어느 방법을 사용하든 상관없다.

. 연산자는 뒤에 식별자 호환(Identifier-Compatible) 프로퍼티명이 와야 하지만 [""] 구문은 UTF-8/유니코드 호환 문자열이라면 모두 프로퍼티명으로 쓸 수 있다는 점에서 차이가 있다.

```js
var wantA = true;
var myObject = {
	a: 2
};

var idx;

if (wantA) {
	idx = "a";
}

// 중략

console.log( myObject[idx] ); // 2
```

객체 프로퍼티명은 언제나 문자열이다. 문자열 이외의 다른 원시 값을 쓰면 우선 문자열로 변환된다. 배열 인데그로 사용하는 숫자도 마찬가지이므로 공연히 객체와 배열 사이에 숫자를 써서 헷갈리는 코드를 만들지 않도록 하자.

```js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];	// "baz"
```

### 3-3-1 계산된 프로퍼티명

ES6부터는 계산된 프로퍼티명(Computed Property Names)이라는 기능이 추가 됐는데, 객체 리터럴 선언 구문의 키 이름 부분에 해당 표현식을 넣고 []로 감싸면 된다.

```js
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

### 3-3-2 프로퍼티vs메서드

함수는 결코 객체에 속하는 것이 아니며, 객체 레퍼런스로 접근한 함수를 그냥 메서드라 칭하는 건 그 의미를 지나치게 확대해 해석한 것이다.

this 레퍼런스를 스스로 지닌 함수도 있고 호출부의 객체 레퍼런스를 가리킬 때도 있긴 하지만 그렇다고 이렇게 사용되는 함수가 다른 함수들보다 메서드답다고 말하는 건 이상하다. this는 호출부에서 런타임 시점에 동적으로 바인딩 되므로 객체와는 기껏해야 간접적인 관계일 수 밖에 없기 때문이다.

프로퍼티 접근 결과 반환된 함수 역시 (앞에서 설명한 암시적 this 바인딩이 있을지 모른다는 점만 빼고) 마찬가지다.

```js
function foo() {
	console.log( "foo" );
}

var someFoo = foo;	// `foo`에 대한 변수 레퍼런스

var myObject = {
	someFoo: foo
};

foo;				// function foo(){..}

someFoo;			// function foo(){..}

myObject.someFoo;	// function foo(){..}
```

someFoo나 myObject.someFoo 모두 같은 함수를 가리키는 개별 레퍼런스일 뿐, 뭔가 특별한 다른 객체가 '소유한' 함수라는 의미는 아니다.

무난하게 결론을 내리자면, 자바스크립트에서 '함수'와 '메서드'란 말은 서로 바꿔 사용할 수 있다.

함수 표현식을 객체 리터럴의 한 부분으로 선언해도 이 함수가 저절로 객체에 달라붙는 건 아니며 해당 함수 객체를 참조하는 레퍼런스가 하나 더 생기는 것뿐이다.

```js
var myObject = {
	foo: function foo() {
		console.log( "foo" );
	}
};

var someFoo = myObject.foo;

someFoo;		// function foo(){..}

myObject.foo;	// function foo(){..}
```

### 3-3-3 배열

배열도 []로 접근하는 형태지만 이미 언급한 대로 값을 저장하는 방법과 장소가 더 체계적이다. 배열은 인덱스라는 양수로 표기된 위치에 값을 저장한다.

```js
var myArray = [ "foo", 42, "bar" ];

myArray.length;		// 3

myArray[0];			// "foo"

myArray[2];			// "bar"
```

배열 자체는 객체여서 배열에 프로퍼티를 추가하는 것도 가능하다. 이름 붙은 프로퍼티를 추가해도 배열 길이에는 변함이 없다.

```js
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";

myArray.length;	// 3

myArray.baz;	// "baz"
```

키/값 저장소로는 객체, 숫자 인덱스를 가진 저장소로는 배열을 쓰는게 좋다.

배열에 프로퍼티를 추가할 때 프로퍼티명이 숫자와 유사하면 숫자 인덱스로 잘못 해석되어 배열 내용이 달라질 수 있으니 주의하자.

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length;	// 4

myArray[3];		// "baz"
```

### 3-3-4 객체 복사

다음 객체를 보자.

```js
function anotherFunction() { /*..*/ }

var anotherObject = {
	c: true
};

var anotherArray = [];

var myObject = {
	a: 2,
	b: anotherObject,	// 사본이 아닌 레퍼런스!
	c: anotherArray,	// 역시 레퍼런스!
	d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

myObject의 사본은 정확히 어떻게 표현해야 할까?

얕은 복사(Shallow Copy)를 하면 생성된 새 객체의 a 프로퍼티는 원래 값 2가 그대로 복사되지만 b, c, d 프로퍼티는 원 객체의 레퍼런스와 같은 대상을 가리키는 또 다른 레퍼런스다. 깊은 복사를 하면 myObject는 물론이고 anotherObject와 myObject를 가리키는 레퍼런스를 갖고 있으므로 원래 레퍼런스가 보존되는게 아니라 이들까지 함께 복사된다. 결국, 환영 참조 형태가 되어 무한 복사에 빠지고 만다.

이 난관을 어떻게 극복할까? 이 문제는 꽤 오랫동안 뾰족한 답이 없었다.

'JSON-Safe'([JSON 문자열 <-> 객체 직렬화 및 역직렬화]를 해도 구조와 값이 같은) 객체'는 쉽게 복사할 수 있으므로 하나의 대안이 될 수는 있다.

```js
var newObj = JSON.parse(JSON.stringfy(someObj));
```

물론, 100% 'JSON-Safe 객체'여야 한다. 이 요건이 별로 중요하지 않을 때도 있지만 이것만으로는 충분하지 않을 경우가 있다.

한편, 얕은 복사는 이해하기 쉽고 별다른 이슈가 없기에 ES6부터는 Object.assign() 메소드를 제공한다.

```js
var newObj = Object.assign( {}, myObject );

newObj.a;						// 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true
```

### 3-3-5 프로퍼티 서술자

ES5 이전에는 읽기 전용과 같은 프로퍼티의 특성을 확인할 방법이 없었으나 ES5 부터 모든 프로퍼티는 프로퍼티 서술자(Property Descriptor)로 표현된다.

```js
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

보시다시피, 2 말고도 wirtable, enumerable, configurable의 세 가지 특성이 더 있다.

Object.defineProperty()로 새로운 프로퍼티를 추가하거나 기존 프로퍼티의 특성을 원하는 대로 수정할 수 있다.(configurable이 true일 떄만 가능)

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );

myObject.a; // 2
```

#### Wirtable

프로퍼티의 값의 쓰기 가능 여부는 writable로 조정한다.

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // 쓰기 금지!
	configurable: true,
	enumerable: true
} );

myObject.a = 3;

myObject.a; // 2
```

쓰기 금지된 값을 수정하려고 하면 조용히 실패하며 엄격 모드에선 에러가 난다.

```js
"use strict";

var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false, // 쓰기 금지!
	configurable: true,
	enumerable: true
} );

myObject.a = 3; // TypeError
```

#### Configurable

프로퍼티가 설정 가능하면 프로퍼티 서술자를 변경할 수 있다.

```js
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;					// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,	// 설정 불가!
	enumerable: true
} );

myObject.a;					// 4
myObject.a = 5;
myObject.a;					// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumerable: true
} ); // TypeError
```

configurable은 일단 false가 되면 돌아올 수 없고 절대로 복구되지 않으니 유의하자.

configurable:false로 설정하면 이미 delete 연산자로 존재하는 프로퍼티 삭제도 금지 된다.

```js
var myObject = {
	a: 2
};

myObject.a;				// 2
delete myObject.a;
myObject.a;				// undefined

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: false,
	enumerable: true
} );

myObject.a;				// 2
delete myObject.a;
myObject.a;				// 2
```

delete는 객체에서 (삭제 가능한) 프로퍼티를 곧바로 삭제하는 용도로만 쓰인다.(C/C++ 같은 메모리 해제할 때는 쓰는 도구랑 다름)

#### Enumerable

for in 루프처럼 객체 프로퍼티를 열거하는 구문에서 해당 프로퍼티의 표출 여부를 나타낸다. enumerable:false로 지정된 프로퍼티는 접근할 수는 있지만 루프 구문에서 감춰진다.

감추고 싶은 특별한 프로퍼티에 한하여 enumerable:false 세팅하자.

### 3-3-6 불변성

ES5부터는 프로퍼티/객체가 변경되지 않도록 할 수 있는 여러 가지 방법을 제공한다. 그러나 이러한 방법은 얕은 불변성(Shallow Immutablity)만 지원한다. 즉, 객체 자신과 직속 프로퍼티 특성만 불변으로 만들 뿐 다른 객체(배열, 객체, 함수 등)를 가리키는 레퍼런스가 있을 때 해당 객체의 내용까지 불변으로 만들지느 ㄴ못한다.

```js
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
```

myImmutableObject.foo의 내용(자신의 객체 - 배열) 까지 보호하려면 뒤에서 나열할 방법으로 foo를 불변 객체로 바꾸어야 한다.

NOTE: 자바스크립트에서 뼛속까지 고정된 불변 객체를 쓸 일은 거의 없다. 필요한 경우도 있겠지만 객체 값이 변경되어도 문제가 없는 견고한 프로그램을 설계할 다른 방법은 없는지 일반적인 디자인 패턴 관점에서 재고해야 한다.

#### 객체 상수

wirtable:false와 configurable:false를 같이 쓰면 객체 프로퍼티를 다음과 같이 상수처럼 쓸 수 있다.

```js
var myObject = {};

Object.defineProperty( myObject, "FAVORITE_NUMBER", {
	value: 42,
	writable: false,
	configurable: false
} );
```

#### 확장 금지

객체에 더는 프로퍼티를 추가할 수 없게 차단하고 현재 프로퍼티는 있는 그대로 놔두고 싶을 때 Object.preventExtensions()를 호출한다.

```js
var myObject = {
	a: 2
};

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b; // undefined
```

비엄격 모드에선 실패하고 엄격모드에서는 TypeError가 발생한다.

#### 봉인

Object.seal()은 봉인된 객체를 생성한다. 더는 프로퍼티를 추가할 수 없을뿐더러 기존 프로퍼티를 재설정하거나 삭제할 수도 없다. 물론 값은 얼마든지 바꿀 수 있다.

#### 동결

Object.freeze()는 객체를 얼린다. Object.seal()을 적용하고 '데이터 접근자(Data Accessor)' 프로퍼티를 모두 writable:false 처리해서 값도 못 바꾸게 한다.(하지만 이 객체가 참조하는 다른 객체의 내용까지 봉쇄하는 건 아니다)
> 아래의 Deep Freeze 내용을 살펴보면 참조하는 모든 객체를 재귀 순회하면서 적용하는 것 같은데 조금 더 알아보야 할 것 같다. 말이 다른 느낌?

Object.freeze()를 적용하면 지금까지는 전혀 영향을 받지 않았던 해당 객체가 참조하는 모든 객체를 재귀 순회하면서 Object.freeze()를 적용하여 깊숲이 동결(Deep Freeze)한다. 하지만 자칫 의도하지 않은 다른 동유된 객체까지 동결시킬 수 있어 주의해야 한다.

### 3-3-7 [[GET]]

프로퍼티에 접근하기까지의 세부 과정은 미묘하면서도 중요하다.

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

myObject.a 코드는 myObject에 대해 [[Get]] 연산을 한다.

기본으로 [[Get]] 연산은 주어진 이름의 프로퍼티를 먼저 찾아보고 있으면 그 값을 반환한다. 없을 경우에는 다른 중요한 작업을 하도록 정의되어 있는데, 이 부분은 5장 프로토타입에서 따로 설명한다.

주어진 프로퍼티 값을 어떻게 해도 찾을 수 없으면 [[Get]] 연산은 undefined를 반환한다.

```js
var myObject = {
	a: 2
};

myObject.b; // undefined
```

```js
var myObject = {
	a: undefined
};

myObject.a; // undefined
myObject.b; // undefined
```

내부적으로 [[Get]] 연산을 수행할 테니 대체로 myObject.b가 myObject.a보다 '더 일을 많이 한다'고 볼 수 있다.

### 3-3-8 [[PUT]]

[[Put]]을 실행하면 주어진 객체에 프로퍼티가 존재하는지(가장 결정적 영향을 미치는 요소다) 등 여러 가지 요소에 따라 이후 작동 방식이 달라진다. [[Put]] 알고리즘은 이미 존재하는 프로퍼티에 대해 대략 다음의 확인 절차를 밟는다.

1. 프로퍼티가 접근 서술자(3-3-9 Getters-Setters 참고)인가? 맞은면 세터를 호출한다.
2. 프로퍼티가 writable:false인 데이터 서술자인가? 맞으면 비엄격 모드에서 조용히 실패하고 엄격모드는 TypeError가 발생한다.
3. 이 외에는 프로퍼티에 해당 값을 세팅한다.

객체에 존재하지 않는 프로퍼티라면 [[Put]] 알고리즘은 훨씬 더 미묘하고 복잡해진다. 5장 프로토타입에서 더 명쾌하게 설명한다.

### 3-3-9 Getters-Setters

ES5부터는 Getters/Setters를 통해 프로퍼티 수준에서 이러한 기본 로직을 오버라이드할 수 있다. Getters/Setters는 각각 실제로 값을 가져오는/세팅하는 감춰진 함수를 호출하는 프로퍼티다.

프로퍼티가 getter 또는 setter 어느 한쪽이거나 동시에 getter/setter가 될 수 있게 정의한 것을 ('데이터 서술자의'의 반대말로) '접근 서술자(Accessor Descriptor)'라고 한다. 접근 서술자에서는 프로퍼티의 값과 wirtable 속성은 무시되며 (configurable, enumerable과 더불어) 대신 프로퍼티의 Get/Set 속성이 중요하다.

```js
var myObject = {
	// `a`의 getter를 정의
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,	// target
	"b",		// property name
	{			// descriptor
		// `b`의 getter를 정의
		get: function(){ return this.a * 2 },

		// `b`가 객체 프로퍼티로 확실히 표시되게 한다.
		enumerable: true
	}
);

myObject.a; // 2

myObject.b; // 4
```

프로퍼티에 접근하면 자동으로 getter 함수를 호출하여 어떤 값이라도 getter 함수가 반환한 값이 결괏값이 된다.

```js
var myObject = {
	// `a`의 getter를 정의한다.
	get a() {
		return 2;
	}
};

myObject.a = 3;

myObject.a; // 2
```

할당문으로 값을 세팅하려고 하면 에러 없이 무시된다. setter가 커스텀 getter가 2만 반환하게 하드 코딩되어 있어서 마찬가지다.

프로퍼티 단위로 기본 [[Put]] 연산(할당)을 오버라이드하는 setter가 정의되어야 한다. getter와 setter는 항상 둘 다 선언하는 것이 좋다.(한쪽만 선언하면 예상외의 결과가 나올 수 있다)

```js
var myObject = {
	// `a`의 getter를 정의
	get a() {
		return this._a_;
	},

	// `a`의 setter를 정의
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```

### 3-3-10 존재 확인

객체에 어떤 프로퍼티가 존재하는지는 굳이 프로퍼티 값을 얻지 않고도 확인할 수 있다.

```js
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

in 연산자는 어떤 프로퍼티가 해당 객체에 존재하는지 아니면 이 객체의 [[Prototype]] 연쇄(5장 프로토타입 참고)를 따라 갔을 때 상위 단계에 존재하는지 확인한다. 이와 달리 hasOwnProperty()는 단지 프로퍼티가 객체에 있는지만 확인하고 [[Prototype]] 연쇄는 찾지 않는다.

거의 모든 일반 객체는 Object.prototype 위임(5장 프로토타입 참고)을 통해 hasOwnProperty()에 접근할 수 있지만 간혹 (Object.create(null)로, 5장 프로토타입 참고) Object.prototype과 연결되지 않은 객체는 myObject.hasOwnProperty() 처럼 사용할 수 없다.

이럴 경우엔 Object.prototype.hasOwnproperty.call(myObject, "a") 처럼 기본 hasOwnProperty() 메서드를 빌려와 myObject에 대해 명시적으로 바인딩하면 좀 더 확실하게 확인할 수 있다.

NOTE: 언뜻 in 연산자가 내부 값이 존재하는지까지 확인하는 것처럼 보이지만 실은 프로퍼티명이 있는지만 본다.

#### 열거

'열거 가능성' 개념으로 돌아가 좀 더 자세히 보자

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// make `a` enumerable, as normal
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// make `b` NON-enumerable
	{ enumerable: false, value: 3 }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (var k in myObject) {
	console.log( k, myObject[k] );
}
// "a" 2
```

myObject.b는 for-in 루프에서는 자취가 사라진다. 이처럼 '열거 가능'하다는 건 기본적으로 '객체 프로퍼티 순회 리스트에 포함' 된다는 뜻이다.

NOTE: for in 루프를 배열에 사용하면 배열 인덱스뿐만 아니라 다른 열거 가능한 프로퍼티까지 순회 리스트에 포함되는 원치 않은 결과가 발생할 수 있다. for in은 객체에만 쓰고 배열은 과거처럼 숫자 인덱스로 순회하는 편이 바람직하다.

프로퍼티가 열거 가능한지는 다른 방법으로도 확인할 수 있다.

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// `a`를 열거가 가능하게 세팅한다.(기본값이다)
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// 'b'를 열거가 불가능하게 세팅한다.
	{ enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

propertyIsEnumerable()은 어떤 프로퍼티가 해당 객체의 직속 프로퍼티인 동시에 enumerable:true인지 검사한다.

Object.keys()는 Object.getOwnPropertyNames()의 열거 가능 여부와 상관없이 객체에 있는 모든 열거 가능한 프로퍼티를 배열 형태로 반환한다.

in과 hasOwnProperty()가 [[Prototype]] 연쇄의 확인에 따라 차이가 있는 반면, Object.keys()와 Object.getOwnPropertyNames()는 모든 주어진 객체만 확인한다.

in 연산자와 결과(모든 프로퍼티를 전체 [[Prototype]] 연쇄에서 순회한다)가 동등한 프로퍼티 전체 리스트를 조회하는 기능은 없다. 단계마다 Object.keys()에서 열거 가능한 프로퍼티 리스트를 포착하여 재귀적으로 주어진 객체의 [[Prototpye]] 연쇄를 순회하는 식의 로직을 구현하여 대략 비슷한 유틸리티를 만들어 쓰면 된다.

## 3-4 순회

프로퍼티 값을 순회하려면 어떻게 할까?

```js
var myArray = [1, 2, 3];

for (var i = 0; i < myArray.length; i++) {
	console.log( myArray[i] );
}
// 1 2 3
```

그러나 이 코드는 인덱스를 순회하면서 해당 값(myArray[i])을 사용할 뿐 값 자체를 순회하는 것은 아니다.

ES5부터는 forEach(), every(), some() 등의 배열 관련 순회 헬퍼가 도입됐다. 이 함수들은 배열의 각 원소에 적용할 콜백 함수를 인자로 받으며, 원소별로 반환 값을 처리하는 로직만 다르다.

forEach()는 배열 전체 값을 순회하지만 콜백 함수의 반환 값은 무시한다. every()는 배열 끝까지 또는 콜백 함수가 false(또는 'falsy' 값)을 반환할 때까지 순회하며 some()은 이와 정반대로 배열 끝까지 또는 콜백 함수가 true(또는 'truthy' 값)를 반환할 때까지 순회한다. every()와 some()의 이러한 특별한 반환 값은 일반적인 for 루프의 break 문처럼 사용한다.

for in 루프를 이용한 객체 순회는 실제로 열거 가능한 프로퍼티만 순회하고 그 값을 얻으려면 일일이 프로퍼티에 접근해야 하므로 간접적인 값 추출이다.

NOTE: 객체 프로퍼티의 순회 순서는 일정하지 않고 자바스크립트 엔진별로도 조금씩 다를 수 있다.

배열 인덱스(혹은 객체 프로퍼티)가 아닌 값을 직접 순회하는 것도 가능할까? 다행히도 ES6붜 배열 순회용 for of 구문을 제공한다.

```js
var myArray = [ 1, 2, 3 ];

for (var v of myArray) {
	console.log( v );
}
// 1
// 2
// 3
```

for of 루프는 순회할 원소의 순회자 객체(Iterator Object 명세식으로 말하면 @@iterator라는 기본 내부 함수)가 있어야 한다. 순회당 한 번씩 이 순회자 객체의 next() 메서드를 호출하여 연속적으로 반환 값을 순회한다.

배열은 @@iterator가 내장된 덕분에 다음 예제에서 보다시피 손쉽게 for of 루프를 사용할 수 있다. 내장 @@iterator를 이용하여 수동으로 배열을 순회하면서 작동 원리 살펴보자.

```js
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
```

NOTE: ES6부터는 Symbol.iterator 심볼로 객체 내부 프로퍼티인 @@iterator에 접근할 수 있다. 이런 특수 프로퍼티는 심볼에 포함될지 모를 특수값보다는 심볼명으로 참조하는 것이 좋다. @@iterator라는 명칭 때문에 순회자 객체란 느낌이 강한데 실은 순회자 객체를 반환하는 함수다.

배열은 for of 루프 내에서 알아서 순회하지만, 일반 객체는 내부에 @@iterator가 없다.

순회하려면 객체의 기본 @@iterator를 손수 정의할 수도 있다.

```js
var myObject = {
	a: 2,
	b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// `myObject`를 수동으로 순회한다.
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

//`myObject`를 `for..of` 루프로 순회한다.
for (var v of myObject) {
	console.log( v );
}
// 2
// 3
```

예제 코드에선 단순히 값 대 값으로 순회하고 있지만 필요에 따라 사용자 자료 구조에 딱 맞는 임의의 복잡한 순회 알고리즘을 정의할 수도 있다. ES6의 for of 루프와 커스텀 순회자는 사용자 정의 객체를 조작하는데 아주 탁월한 새로운 구문 도구다.

순회가 절대로 끝나지 않고 항상 새로운 값(랜덤값, 중분값, 유일한 식별자 등)을 반환하는 무한 순회자도 가능하다. 하지만 이러헥 순회자로 for of 루프의 경계를 무너뜨리면 결국 실행이 멈추지 않아 프로그램이 멎을 수 있다.

```js
var randoms = {
	[Symbol.iterator]: function() {
		return {
			next: function() {
				return { value: Math.random() };
			}
		};
	}
};

var randoms_pool = [];
for (var n of randoms) {
	randoms_pool.push( n );

	// 제한 없이 사용한다.
	if (randoms_pool.length === 100) break;
}
```

이 코드의 순회자는 난수를 무한히 생성하므로 프로그램이 멎지 않도록 100개만 추출해야 한다.

## 3-5 정리하기

자바스크립트 객체는 리터럴 형식(var a= {})과 생성자 형식(var a = new Array()) 두 가지 형태를 가진다. 대부분 리터럴 형식을 쓰는 편이 좋지만 생성 시 옵션을 더 주기 위해 생성자 형식을 쓰는 경우도 더러 있다.

많은 사람이 "자바스크립트는 모든 것이 다 객체다"라고 말하지만 사실과 다르다. 객체는 6개(관점에 따라 7개가 될 수도 있다)의 원시 타입 중 하나고 함수를 비롯한 하위 타입이 있다. 이를테면 내부적으로 [object Array]라는 레이블로 표시되는 배열 객체라는 독특한 하위 타입도 가능하다.

객체는 키/값의 쌍을 모아 놓은 저장소고 값은 프로퍼티를 통해 (.propname or ["propName"]) 접근할 수 있다. 프로퍼티에 접근하면 엔진 내부에서는 실제로 기본 [[Get]] (값을 세팅할 때는 [[Put]]) 연산을 호출하는데, 객체 자체에 포함된 프로퍼티뿐만 아니라 필요하면 [[Prototype]] 연쇄를 순회하며 찾아본다.

프로퍼티는 프로퍼티 서술자를 통해 제어 가능한 writable, configurable 등의 특정한 속성을 지닌다. 그리고 객체는 Object.preventExtensions(), Object.seal(), Object.freeze() 등을 이용하여 자신에게 (그리고 자신이 가지고 있는 프로퍼티까지) 여러 단계의 불변성을 적용할 수 있다.

프로퍼티가 반드시 값을 가져야 하는 것은 아니며 게터/세터로 '접근자 프로퍼티' 형태를 취할수도 있다. 예를 들어, 열거 가능성을 조정하여 for in 루프 순회 시 노출 여부를 마음대로 바꿀 수 있다.

ES6부터는 for of 구문에서 한 번에 하나씩 다음 데이터값으로 이동하는 next() 메서드를 가진 내장/커스텀 @@iterator 객체를 통해 자료 구조(배열, 객체 등)에서 여러 값을 순회할 수 있다.
