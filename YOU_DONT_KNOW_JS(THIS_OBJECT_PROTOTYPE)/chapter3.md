# 객체

* [3-1 구문](#3-1-구문)
* [3-2 타입](#3-2-타입)
* [3-3 내용](#3-3-내용)

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
