# CHAPTER 4 강제변환

이 장의 목적은 강제변환의 좋고 나쁨을 충분히 이해하고 자신이 프로그램에 적절한지 스스로 판달할 수 있는 역량을 늘리기다.

* [4-1 값 변환](#4-1-값-변환)
* [4-2 추상 연산](#4-2-추상-연산)

## 4-1 값 변환

어떤 값을 다른 타입의 값으로 바꾸는 과정이 명시적이면 '타입 캐스팅(Type Casting)', (값이 사용되는 규칙에 따라) 암시적이면 '강제변환(Coercion)'이라고 한다.

자바스크립트에서는 대부분 모든 유형의 타입변환을 강제변환으로 뭉뚱그려 일컫는 경향이 많다. 여기서는 '암시적 강제변환(Explicit Coercion')'과 '명시적 강제변환(Implicit Coercion)' 두 가지로 구별하겠다.

* 명시적 강제변환 : 코드만 봐도 의도적으로 타입변환을 일으킨다는 사실이 명백
* 암시적 강제변환 : 다른 작업 도중 불분명한 부수 효과로부터 발생하는 타입변환

```js
var a = 42;

var b = a + "";			// 암시적 강제변환

var c = String( a );	// 명시적 강제변환
```

두 가지 접근 방식 모두 42를 "42"로 바꾸는데, 여기서 논란의 핵심은 변환을 '어떻게' 할 것이냐 하는 문제다.

## 4-2 추상 연산

어떻게 값이 문자열, 숫자, 불리언 등의 타입이 되는지, 그 기본 규칙을 알아보자.

### 4-2-1 ToString

'문자열이 아닌 값 -> 문자열' 변환 작업은 ES5 9.8의 ToString 추상 연산 로직이 담당한다. 내장 원시 값은 본연의 문자열화 방법이 정해져 있다.(null -> "null", undeinfed -> "undefined", true -> "true") 숫자는 그냥 문자열로 바뀌고 너무 작거나 큰 값은 지수 형태로 바뀐다.(2장 값 참고).

```js
// `1.07` 에 `1000`을 7번 곱한다
var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;

// 소수점 이하로 3 x 7 => 21자리까지 내려간다
a.toString(); // "1.07e21"
```

일반 객체는 특별히 지정하지 않으면 기본적으로 (Object.prototype.toString()에 있는) toString() 메서드가 내부 [[Class]]를 반환한다.(예: "[object Object]") 자신의 toString() 메서드를 가진 객체는 문자열처럼 사용하면 자동으로 이 메서드가 호출된다.

배열은 기본적으로 재정의된 toString()이 있다. 문자열 변환 시 모든 원소 값이 (각각 문자열로 바뀌어) 콤마로 분리된 형태로 이어진다.

```js
var a = [1,2,3];
a.toString(); // "1,2,3"
```

또한 toString() 메서드는 명시적으로도 호출 가능하며, 문자열 콘텍스트에서 문자열 아닌 값이 있을 경우에도 자동 호출된다.

#### JSON 문자열화

JSON 문자열화는 강제변환과 똑같지는 않지만, 방금 전 살편본 ToString 규칙과 관련이 있다.

대부분 단순 값들은 직렬화 결과가 반드시 문자열이라는 점을 제외하고는, JSON 문자열화나 toString() 변환이나 기본적으로 같은 로직이다.

```js
JSON.stringify( 42 );	// "42"
JSON.stringify( "42" );	// ""42"" (따옴표가 붙은 문자열 인자를 문자열화한다)
JSON.stringify( null );	// "null"
JSON.stringify( true );	// "true"
```

JSON 안전 값(JSON-Safe Value)은 모두 JSON.stringfy()로 문자열화할 수 있다. (아닌 예: undefined, 함수, Symbol, 환형 참조 객체)

JSON.stringfy() 인자가 undefined, 함수, 심벌 값이면 자동으로 누락시키며 이런 값들이 만약 배열에 포함되어 있으면 (배열 인덱스 정보가 뒤바뀌지 않도록) null로 바꾼다. 객체 프로퍼티에 있으면 간단히 지운다.

```js
JSON.stringify( undefined );					// undefined
JSON.stringify( function(){} );					// undefined

JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } );		// "{"a":2}
```

혹시라도 JSON.stringfy()에 환형 참조 객체를 넘기면 에러가 난다.

객체 자체에 toJSON() 메서드가 정의되어 있다면, 먼저 이 메서드를 호출하여 직렬화한 값을 변환한다.

부적절한 JSON 값이나 직렬화하기 곤란한 객체 값을 문자열화하려면 toJSON() 메서드(해당 객체의 JSON 안전 버전을 반환)를 따로 정의해야 한다.

```js
var o = { };

var a = {
	b: 42,
	c: o,
	d: function(){}
};

// `a`를 환형 참조 객체로 만든다
o.e = a;

// 환형 참조 객체는 JSON 문자열화 시 에러가 난다
// JSON.stringify( a );

// JSON 값으로 직렬화하는 함수를 따로 정의한다
a.toJSON = function() {
	// 직렬화에 프로퍼티 'b'만 포함시킨다.
	return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```

toJSON()은 (어떤 타입이든) 적절히 평범한 실제 값을 반환하고 문자열화 처리는 JSON.stringfy()이 담당한다. 다시 말해 toJSON()의 역할은 '문자열화하기 적당한 JSON 안전 값으로 바꾸는 것'이지, 'JSON 문자열로 바꾸는 것'이 아니다.

```js
var a = {
	val: [1,2,3],

	// 맞다!
	toJSON: function(){
		return this.val.slice( 1 );
	}
};

var b = {
	val: [1,2,3],

	// 틀리다!
	toJSON: function(){
		return "[" +
			this.val.slice( 1 ).join() +
		"]";
	}
};

JSON.stringify( a ); // "[2,3]"

JSON.stringify( b ); // ""[2,3]"
```

b는 배열 자체가 아니라 반환된 문자열을 다시 문자열화한다.

배열 아니면 함수 형태의 대체자(Replace)를 JSON.stringfy()의 두 번째 선택 인자로 지정하여 객체를 재귀적으로 직렬화하면서 (포함할 프로퍼티와 제외할 프로퍼티를 결정하는) 필터링 하는 방법이 있다. toJSON()이 직렬화할 값을 준비하는 방식과 비슷하다.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

JSON.stringfy()는 세 번째 선택 인자는 스페이스라고 하며 사람이 읽기 쉽도록 들여쓰기를 할 수 있다.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, null, 3 );
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify( a, null, "-----" );
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

JSON.stringfy()는 직접적인 강제변환의 형식은 아니지만 두 가지 이유로 ToString 강제 변환과 연관된다.
1. 문자열, 숫자, 불리언, null 값이 JSON으로 문자열화하는 방식은 ToString 추상 연산의 규칙에 따라 문자열 값으로 강제변환되는 방식과 동일하다.
2. JSON.stringfy()에 전달한 객체가 자체 toJSON() 메서드를 갖고 있다면, 문자열화 전 toJSON()이 자동 호출되어 JSON 안전 값으로 '강제변환' 된다. 

### 4-2-2 ToNumber

'숫자 아닌 값 -> 수식 연산이 가능한 숫자' 변환 로직은 ES5 9.3 ToNumber 추상 연산에 잘 정의되어 있다.

ex) true => 1, false => 0, undefined => NaN, (희한하게도) null => 0

문자열 값에 ToNumber를 적용하면 대부분 숫자 리터럴 규칙/구문과 비슷하게 작동한다. 변환이 실패하면 (숫자 리터럴 구문 에러가 아닌) 결과는 NaN이다. 한가지 다른 점은 0이 앞에 붙은 8진수는 ToNumber에서 올바른 숫자 리터럴이라도 8진수로 처리하지 않는다.(일반 10진수로 처리)

객체(그리고 배열)는 일단 동등한 원시 값으로 변환 후 그 결괏값(아직 숫자가 아닌 원시 값)을 ToNumber 규칙에 의해 강제 변환한다.

동등한 원시 값으로 바꾸기 위해 ToPrimitive 추상 연산 과정에서 해당 객체가 valueOf() 메서드를 구현했는지 확인한다. ValueOf()를 쓸 수 있고 반환 값이 원시 값이면 그대로 강제변환 한다. 그렇지 않을 경우 toString() 메서드가 존재하면 toString()을 이용하여 강제변환 한다.

어찌해도 원시 값으로 바꿀 수 없을 땐 TypeError 오류를 던진다.

ES5 부터는 [[Prototype]]이 null인 경우 대부분 Object.create(null)를 이용하여 강제변환이 불가능한 객체(valueOf(), toString()이 없는 객체)를 생성할 수 있다.

```js
var a = {
	valueOf: function(){
		return "42";
	}
};

var b = {
	toString: function(){
		return "42";
	}
};

var c = [4,2];
c.toString = function(){
	return this.join( "" );	// "42"
};

Number( a );			// 42
Number( b );			// 42
Number( c );			// 42
Number( "" );			// 0
Number( [] );			// 0
Number( [ "abc" ] );	// NaN
```

### 4-2-3 ToBoolean

자바스크립트에는 true와 false라는 키워드가 존재한다. 그리고 1을 true로, 0을 false로 (역방향도 마찬가지) 강제변환할 수는 있지만 그렇다고 두 값이 똑같은 건 아니다.

#### Falsy 값

true/false가 아닌 값을 boolean에 상당한 값으로 강제 변환하면 어떻게 작동할까?

다음 둘 중 하나다.
1. boolean으로 강제변환하면 false가 되는 값
2. 1번을 제외한 나머지(즉 명백히 true인 값)

명세가 정의한 'falsy' 값은 다음과 같다.
* undefined
* null
* false
* +0, -0, NaN
* ""

이게 전부다. 이 'falsy' 값을 boolean으로 강제변환하면 false다.

'truthy' 값 목록은 없지만 'falsy' 값 목록에 없으면 응당 'truthy' 값이 된다.

| 인자타입 | 결괏값 |
| --- | --- |
| `Undefined` | false |
| `Null` | false |
| `Boolean` | 인자 값과 동일(변환 안함) |
| `Number` | +0, -0, NaN이면 false, 그 외에는 true |
| `String` | 인자가 공백 문자열(length가 0)이면 false, 그 외에는 true |
| `Object` | true |

