# CHAPTER 3 네이티브


* [3-1 내부 [[Class]]](#3-1-내부-[[Class])
* [3-2 래퍼 박싱하기](#3-2-래퍼-박싱하기)
* [3-3 언박싱](#3-3-언박싱)
* [3-4 네이비트, 나는 생성자다](#3-4-네이비트,-나는-생성자다)
* [3-5 정리하기](#3-5-정리하기)

네이티브란 특정 환경(브라우저등의 클라이언트 프로그램)에 종속되지 않은, ECMAScript 명세의 내장 객체를 말한다.

다음은 가장 많이 쓰는 네이티브들이다.

* String()
* Number()
* Boolean()
* Array()
* Object()
* Function()
* RegExp()
* Date()
* Error()
* Symbol() - ES6에서 추가 됨

네이티브는 생성자처럼 사용할 수 있지만 실제로 생성되는 겨로가물은 여러분의 예상과 다를 수 있다.

```js
var a = new String( "abc" );

typeof a; // "object" ... "String"이 아니다

a instanceof String; // true

Object.prototype.toString.call( a ); // "[object String]"
```

(new String("abc)) 생성자의 결과는 원시 값 "abc"를 감싼 객체 래퍼다.

typeof 연산자로 이 객체의 타입을 확인해보면 object의 하위 타입에 가깝다.

```js
console.log( a ); // var a = new String( "abc" );
```

이 코드의 실행 결과는 브라우저마다 다르다.

요지는 new String("abc")은 "abc"를 감싸는 문자열 래퍼를 생성하며 원시 값 "abc"는 아니라는 점이다.

## 3-1 내부 [[Class]]

typeof가 'object'인 값(배열 등)에는 [[Class]](객체지향의 class 의미보다는 내부 분류법인 classfication의 일부라고 보자)라는 내부 프로퍼티가 추가로 붙는다. 이프로퍼티는 직접 접근할 수 없고 Object.prototype.toString()라는 메서드에 값을 넣어 호출함으로써 존재를 엿볼 수 있다.

```js
Object.prototype.toString.call( [1,2,3] );			// "[object Array]"

Object.prototype.toString.call( /regex-literal/i );	// "[object RegExp]"
```

대부분 내부 [[Class]]는 해당 값과 관련된 내장 네이티브 생성자(다음에 설명한다)를 가리키지만, 그렇지 않을 때도 있다.

원시 값에도 내부 [[Class]]가 있을까?

```js
Object.prototype.toString.call( null );			// "[object Null]"
Object.prototype.toString.call( undefined );	// "[object Undefined]"
```

Null(), Undefined() 같은 네이티브 생성자는 없지만 내부 [[Class]] 값을 확인해보니 "Null", "Undefined"이다.

하지만 그 밖의 문자열, 숫자 같은 단순 원시 값은 이른바 '박싱Boxing' 과정을 거친다.

```js
Object.prototype.toString.call( "abc" );	// "[object String]"
Object.prototype.toString.call( 42 );		// "[object Number]"
Object.prototype.toString.call( true );		// "[object Boolean]"
```

단순 원시 값은 해당 객체 래퍼로 자동 박싱됨을 알 수 있다.

## 3-2 래퍼 박싱하기

원시 값엔 프로퍼티나 메서드가 없으므로 .length, .toString()으로 접근하려면 원시 값을 객체 래퍼로 감싸줘야 한다. 고맙게도 자바스크립트는 원시 값을 알아서 박싱(래핑)하므로 다음과 같은 코드가 가능하다.

```js
var a = "abc";

a.length; // 3
a.toUpperCase(); // "ABC"
```

그러면 자바스크립튼 엔진이 암시적으로 객체를 생성할 필요가 없도록 처음부터 객체로 생성해야할까?(ex. 루프 조건 i < a.length)

놉!. 오래전 부터 브라우저는 이런 흔한 경우를 스스로 최적화하기 때문에 개발자가 직접 객체 형태로 '선 최적화(Pre-Optimize)'하면 프로그램이 더 느려질 수 있다.

그러므로 new String("abc"), new Number (42) 처럼 코딩하지 말고, 그냥 알기 쉽게 원시 값 "abc", 42를 사용하자.

### 3-2-1 객체 래퍼의 함정

그래도 객체 래퍼를 사용해야 한다면 정말 조심해야할 함정이 있다.

예를 들어, Boolean으로 래핑한 값이 있다.

```js
var a = new Boolean( false );

if (!a) {
	console.log( "Oops" ); // 실행되지 않는다.
}
```

false를 객체 래퍼로 감쌌지만 문제는 객체가 'truthy'(4장 강제변환 참고)란 점이다. 그래서 예상과는 달리, 안에 들어있는 false 값과 반대의 결과다.

수동으로 원시 값을 박싱하려면 Object() 함수를 이요하자.(앞에 new 키워드는 없다)

```js
var a = "abc";
var b = new String( a );
var c = Object( a );

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call( b ); // "[object String]"
Object.prototype.toString.call( c ); // "[object String]"
```

다시 한번 말하지만 객체 래퍼로 직접 박싱하는건 권하고 싶지 않다. 물론, 드물긴 하지만 그렇게 해야 할 경우가 전혀 없진 않다.

## 3-3 언박싱

객체 래퍼의 원시 값은 valueOf() 메서드로 추출한다.

```js
var a = new String( "abc" );
var b = new Number( 42 );
var c = new Boolean( true );

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

이때에도 암시적인 언박싱이 일어난다. 자세한건 4장 강제변환에서 다시 다룬다.

```js
var a = new String( "abc" );
var b = a + ""; // `b` 에는 언박싱된 원시 값 "abc"가 대입된다.

typeof a; // "object"
typeof b; // "string"
```

## 3-4 네이비트, 나는 생성자다

### 3-4-1 Array()

```js
var a = new Array( 1, 2, 3 ); // Array(1, 2, 3,) 과 같다. new 없어도 됨.
a; // [1, 2, 3]

var b = [1, 2, 3];
b; // [1, 2, 3]
```

Array 생성자에는 특별한 형식이 하나 있는데 인자로 숫자를 하나만 받으면 그 숫자를 원소로 하는 배열을 생성하는게 아니라 '배열의 크기를 미리 정하는' 기능이다.

브라우저 개발자 콘솔 창마다 객체를 나타내는 방식이 제각각이라 혼란은 가중된다. 예를 들어,

```js
var a = new Array( 3 );

a.length; // 3
a;
```

(이 책의 집필 시점에서) 크롬의 직렬화 결과는 [ undefined x 3 ]이다. 슬롯 자체가 존재하지 않아, 배열 슬롯에 3개의 undefined 값을 밀어 넣은 것이다.

다음 코드를 실행해보고 차이점을 확인하자.

```js
var a = new Array( 3 );
var b = [ undefined, undefined, undefined ];
var c = [];
c.length = 3;

a;
b;
c;
```

> 2019.12.30 크롬에서 확인 결과 => a, c : [empty × 3] b : [undefined, undefined, undefined]

설상가상으로 파이어폭스에서 실행하면 a와 c가 [ , , , ]로 나온다. (최신 버전에는  Array [ <3 empty slots>] )로 콘솔 창에 찍힌다.)

그런데 콘솔 결과가 헷갈리는 건 그렇다 치고 a와 b가 어떨 때는 같은 값처럼 보이다가도 그렇지 않을 때도 있다는 점이 더 나쁘다.

```js
// var a = new Array( 3 );
// var b = [ undefined, undefined, undefined ];

a.join( "-" ); // "--"
b.join( "-" ); // "--"

a.map(function(v,i){ return i; }); // [ undefined x 3 ]
b.map(function(v,i){ return i; }); // [ 0, 1, 2 ]
```

a.map()은 a에 슬롯이 없기 때문에 map() 함수가 순회할 원소가 없다. join은 다르다. 기본적인 join()의 구현 로직은 다음과 같다.

```js
function fakeJoin(arr,connector) {
	var str = "";
	for (var i = 0; i < arr.length; i++) {
		if (i > 0) {
			str += connector;
		}
		if (arr[i] !== undefined) {
			str += arr[i];
		}
	}
	return str;
}

var a = new Array( 3 );
fakeJoin( a, "-" ); // "--"
```

('빈 슬롯'말고) 진짜 undefined 값 원소로 채워진 배열은 (손으로 입력하지 않고) 어떻게 생성할까?

```js
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```

apply() 첫번째 인자 this는 객체 바인딩, 두 번째 인자는 인자의 배열(또는 유사 배열)로, 이 안에 포함된 원소들이 펼쳐져 함수의 인자로 전달된다.

즉 호출하면 Array (undefined, undefined, undefined,)처럼 되어 (말도 안되는) 빈 슬롯이 아닌, undefined로 채워진 배열이 탄생한다.

어쨋든 이런 이국적인 빈 슬롯 배열을 애써 만들지 말자.

### 3-4-2 Object(), Funtion(), and RegExp()

일반적으로 Object(), Function(), RegExp() 생성자도 선택 사항이다.(어떤 분명한 의도가 아니라면 사용하지 말자)

```js
var c = new Object();
c.foo = "bar";
c; // { foo: "bar" }

var d = { foo: "bar" };
d; // { foo: "bar" }

var e = new Function( "a", "return a * 2;" );
var f = function(a) { return a * 2; };
function g(a) { return a * 2; }

var h = new RegExp( "^a*b+", "g" );
var i = /^a*b+/g;
```

new Object() 같은 생성자 폼은 사실상 사용할 일이 없다. 번거럽게 하나씩 일일이 프로퍼티를 지정할 필요가?

Function 생성자는 함수의 인자나 내용을 동적으로 정의해야 하는, 매우 드문 경우에 한해 유용하다.

정규표현식은 리터럴 형식(/^a*b+/g)으로 정의할 것을 적극 권장한다. 구문이 쉽고 무엇보다 성능상 이점(자바스크립트 엔진이 실행 전 정규 표현식을 미리 컴파일 후 캐시한다)이 있다. RegExp()는 정규 표현식 패턴을 동적으로 정의할 경우 의미 있는 유틸리티다.

```js
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );

var matches = someText.match( namePattern );
```

이런 코드는 자바스크립트 코드에서 수시로 등장하는데 new RegExp("패턴", "플래그") 형식으로 사용하자.

### 3-4-3 Date() and Error()

네이티브 생성자 Date() 와 Error()는 리터럴 형식이 없으므로 다른 네이티브에 비해 유용하다.

date 객체 값은 new Date()로 생성한다. 이 생성자는 날짜/시각을 인자로 받는다.(인자를 생략하면 현재 날씨/시가긍로 대신)

date 객체는 유닉스 타임스탬프 값을 얻는 용도로 가장 많이 쓰일 것이다. date 개체의 인스턴스로부터 getTime()을 호출하면 된다.

하지만 ES5에 정의된 정적 도우미 함수(Helper Function), Date.now()를 사용하는게 더 쉽다. ES5 이전 브라우저에선 다음 폴리필을 쓰자.

```js
if (!Date.now) {
	Date.now = function(){
		return (new Date()).getTime();
	};
}

// new 키워드 없이 Date()를 호출하면 현재 날짜/기각에 해당하는 문자열을 반환
```

Error() 생성자는 (Array ()와 마찬가지로) 앞에 new가 있든 없든 결과는 같다.

error 객체의 주 용도는 현재의 실행 스택 콘텍스트를 포착하여 객체에 담는 것이다. 이 실행 스택 콘텍스트는 디버깅에 도움이 될 만한 정보들을 담고 있다.

error 객체는 보통 throw 연산자와 함께 사용한다.

```js
function foo(x) {
	if (!x) {
		throw new Error( "x를 안줬어요" );
	}
	// ..
}
```

사람이 읽기 편한 포맷으로 에러 메시지를 보려면 그냥 error 객체의 (강제변환 과정을 거쳐) toString()을 호출하는 것이 가장 좋다.

### 3-4-4 Symbol()

Symbol은 ES6에서 처음 선보인, 새로운 원시 값 타입니다. 심벌은 충돌 염려 없이 객체 프로퍼티로 사용 가능한, 특별한 '유일 값'이다.(절대적으로 유일함이 보장되지는 않는다!)

Symbol은 프로퍼티명으로 사용할 수 있으나, 프로그램 코드나 개발자 콘솔 창에서 Symbol의 실제 값을 보거나 접근하는 건 불가능하다.

ES6에는 심벌 몇 개가 미리 정의되어 있는데 Symbol.create, Symbol.iterator 식으로 Symbol 함수 객체의 정적 프로퍼티로 접근한다.

```js
obj[Symbol.iterator] = function(){ /*..*/ };
```

심벌을 직접 정의하려면 Symbol() 네이티브를 사용한다. Symbol()은 앞에 new를 붙이면 에러가 나는, 유일한 네이티브 '생성자'다.

```js
var mysym = Symbol( "my own symbol" );
mysym;				// Symbol(my own symbol)
mysym.toString();	// "Symbol(my own symbol)"
typeof mysym; 		// "symbol"

var a = { };
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

Symbol은 Private 프로퍼티는 아니지만, 본래의 사용 목적에 맞게 대부분 전용 혹은 특별한 프로퍼티로 사용한다.

심벌은 객체가 아니다. 단순한 스칼라 원시 값이다.

### 3-4-5 네이티브 프로토타입

내장 네이티브 생성자는 각자의 .prototype 개게를 가진다(예: Array, prototype, String, prototype 등).

* prototype 객체에는 해당 객체의 하위 타입별로 고유한 로직이 담겨 있다.

이를테면 문자열 원시 값을 (박싱으로) 확장한 것까지 포함하여 모든 String 객체는 기본적으로 String.prototype 객체에 정의된 메서드에 접근할 수 있다.

문서화 관례에 따라 String.prototype.XYZ는 String#XYZ로 줄여 쓴다. 다른 .prototype도 마찬가지다.

* String#indexOf() 문자열에서 특정 문자의 위치를 검색
* String#charAt() 문자열에서 특정 위치의 문자를 반환
* String#substr(), String#substring(), and String#slice() 문자열의 일부를 새로운 문자열로 추출
* String#toUpperCase() and String#toLowerCase() 대문자/소문자로 변환된 새로운 문자열 생성
* String#trim() 앞/뒤의 공란이 제거된 새로운 문자열 생성

이 중 문자열 값을 변경하는 메서드는 없다. 수정이 일어나면 늘 기존 값으로부터 새로운 값을 생성한다.

프로토타입 위임 덕분에 모든 문자열이 같이 쓸 수 있다.

```js
var a = " abc ";

a.indexOf( "c" ); // 3
a.toUpperCase(); // " ABC "
a.trim(); // "abc"
```

각 생성자 프로토타입마다 자신의 타입에 적합한 기능이 구현되어 있다. Number#toFixed(), Array#concat() 그리고 모든 함수는 Function.prototype에 정의된 apply(), call(), bind() 메서드를 사용할 수 있다.

그러나 모든 네이티브 프로토타입이 모두 그렇게 평번한 것은 아니다.

```js
typeof Function.prototype;			// "function"
Function.prototype();				// 빈 함수!

RegExp.prototype.toString();		// "/(?:)/" -- 빈 regex
"abc".match( RegExp.prototype );	// [""]
```

이렇게 네이티브 프로토타입을 변경할 수도 있지만 이는 결코 바람직하지 않다.

```js
Array.isArray( Array.prototype );	// true
Array.prototype.push( 1, 2, 3 );	// 3
Array.prototype;					// [1,2,3]

// 이런 식으로 놔두면 이상하게 작동 할 수 있다
// 다음 코드는 'Array.prototype'을 비워버린다
Array.prototype.length = 0;
```

#### 프로토타입은 디폴트다

변수에 적절한 타입의 값이 할당되지 않은 상태에서 아래와 같은 예시는 모두 멋진 '디폴트 값'이다.

```js
function isThisCool(vals,fn,rx) {
	vals = vals || Array.prototype;
	fn = fn || Function.prototype;
	rx = rx || RegExp.prototype;

	return rx.test(
		vals.map( fn ).join( "" )
	);
}

isThisCool();		// true

isThisCool(
	["a","b","c"],
	function(v){ return v.toUpperCase(); },
	/D/
);	// false
```

Note : ES6 부터는 'vals = vals || 디폴트 값' 식의 구문 트릭은 더 이상 필요 없다. 왜냐하면 함수 선언부에서 네이티브 구문을 통해 인자의 디폴트 값을 설정할 수 있기 때문이다.

프로토타입으로 디폴트 값을 세팅하면 사소하지만 추가적인 이점이 있다. .prototypes는 이미 생성되어 내장된 상태이므로 단 한번만 생성된다. 그러나 [], fucntion(){}, /(?:)/ 를 디폴트 값으로 사용하면 isThisCool()를 호출 할 때마다 디폴트 값을 다시 생성하므로 그만 큼 메모리/CPU가 낭비 된다.

그리고 이후에 변경될 디폴트 값으로 Array.prototype을 사용하는 일이 없도록 하자. 변수가 수정되면 Array.prototype 자체도 수정되어 이미 앞에서 언급한 함정에 빠지게 될 것.

## 3-5 정리하기

자바스크립트는 원시 값을 감싸는 객체 래퍼, 즉 네이티브를 제공한다. 객체 래퍼에는 타입별로 쓸 만한 기능이 구현되어 있어 편리하게 사용할 수 있다.

"abc" 같은 단순 스칼라 원시 값이 있을 때, 이 값의 length 프로퍼티나 String.prototype에 정의된 메서드를 호출하면 자바스크립트는 자동으로 원시 값을 '박싱'하여 필요한 프로퍼티와 메서드를 쓸 수 있게 도와준다.
