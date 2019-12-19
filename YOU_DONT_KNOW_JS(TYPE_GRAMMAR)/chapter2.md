# CHAPTER 2 값

자바스크립트에 내장된 값 타입과 작동 방식을 살펴보고 정확하게 사용할 수 있도록 이해하자.

* [2-1 배열](#2-1-배열)
* [2-2 문자열](#2-2-문자열)
* [2-3 숫자](#2-3-숫자)

## 2-1 배열

자바스크립트 배열은 문자열, 숫자, 객체 심지어 다른 배열이나(이런식으로 다차원 배열을 만든다) 어떤 타입의 값이라도 담을 수 있는 그릇이다.

```js
var a = [ 1, "2", [3] ];

a.length;		// 3
a[0] === 1;		// true
a[2][0] === 3;	// true
```

배열 크기는 미리 정하지 않고도 선언할 수 있으며 원하는 값을 추가하면 된다.

```js
var a = [ ];

a.length;	// 0

a[0] = 1;
a[1] = "2";
a[2] = [ 3 ];

a.length;	// 3
```

(빈/빠진 슬로잇 있는) '구멍 난(spare)' 배열을 다룰 때는 조심해야 한다.

```js
var a = [ ];

a[0] = 1;
// no `a[1]` slot set here
a[2] = [ 3 ];

a[1];		// undefined

a.length;	// 3
```

실행은 되지만 이런 코드에서 중간에서 건너뛴 '빈 슬롯'은 혼란을 부추길 수 있다. a[1] 슬롯 값은 응당 undefined가 될 것 같지만, 명시적으로 a[1] = undefined 세팅한 것과 똑같지는 않다.

> 차이가 궁금해서 찾아봤따. 배열에서 명시적으로 undefined로 지정했을 경우 for in, forEach 구문에서 index가 존재해서 실행된다. 하지만 건너 띈 '빈 슬롯'은 index가 없어 넘어간다. 정확한 내용과 원리는 더 찾아봐야 될 것 같다. [참고링크](https://junhobaik.github.io/js-array/)

배열 인덱스는 숫자인데, 배열 자체도 하나의 객체여서 key/property 문자열을 추가할 수 있다. 하지만 배열 length가 증가하지 않는다.

```js
var a = [ ];

a[0] = 1;
a["foobar"] = 2;

a.length;		// 1
a["foobar"];	// 2
a.foobar;		// 2
```

그런데 키로 넣은 문자열 값이 표준 10진수 숫자로 타입이 바뀌면, 마치 문자열 키가 아닌 숫자 키를 사용한 것 같은 결과가 초래된다는 점은 정말 주의해야 할 함정이다!

```js
var a = [ ];

a["13"] = 42;

a.length; // 14
```

일반적으로 배열에 문자열 타입의 key/property 를 두는 건 추천하고 싶지 않다. 그렇게 해야 한다면 객체를 다용하고 배열 원소의 인덱스는 확실히 숫자만 쓰자.

### 2-1-1 유사 배열

유사 배열 값[숫자 인덱스가 가리키는 값들의 집합]을 진짜 배열로 바꿀 때는 배열 유틸리티 함수(indexof(), concat(), forEach() 등)를 사용하여 해결하는 것이 일반적이다.
예를 들면 DOM 쿼리 작업을 수행하면 유사 배열 형태의 DOM 원소 리스트가 반환된다. 다른 예로 함수에서 arguments 객체를 사용하여 인자를 리스트로 가져오는 것(ES6부터 비 권장)도 마찬가지다.

이런 변환은 slice() 함수의 기능을 차용하는 방법을 가장 많이 쓴다.

```js
function foo() {
	var arr = Array.prototype.slice.call( arguments );
	arr.push( "bam" );
	console.log( arr );
}

foo( "bar", "baz" ); // ["bar","baz","bam"]
```

ES6부터는 기본 내장 함수 Array.from()이 이 일을 대신한다.

```js
...
var arr = Array.from( arguments );
...
```

## 2-2 문자열

자바스크립트 문자열은 실제로 생김새만 비슷할 뿐 문자 배열과 같지 않다.
문자열은 배열과 겉모습이 닮았다(유사 배열이다). 이를테면 둘 다 length 프로퍼티, indexOf() 메서드(ES5 배열에만), concat() 메서드를 가진다.

```js
var a = "foo";
var b = ["f","o","o"];

a.length; // 3
b.length; // 3

a.indexOf( "o" ); // 1
b.indexOf( "o" ); // 1

var c = a.concat( "bar" );  // "foobar"
var d = b.concat( ["b","a","r"] );  // ["f","o","o","b","a","r"]

a === c;  // false
b === d;  // false

a;  // "foo"
b;  // ["f","o","o"]
```

그렇다면 기본적으로는 둘 다 '문자의 배열'이라고 할 수 있을까? 그렇지 않다.

```js
a[1] = "O";
b[1] = "O";

a; // "foo"
b; // ["f","O","o"]
```

문자열은 불변 값(Immutable)이지만 배열은 가변 값(Mutable)이다.
a[1] 처럼 문자열의 특정 문자를 접근하는 형태가 모든 자바스크립트 엔진에서 유효한 것은 아니다. 실제로 인터넷 익스플로러 구버전은 이를 문법 에러로 인식한다(IE 7 까지 a[1] 값은 undefined로 나옴. IE8 부터 a[1]에 o가 할당됨). a.charAt(1)로 접근해야 맞다.

그리고 문자열은 불변 값이므로 문자열 메서드는 그 내용을 바로 변경하지 않고 항상 새로운 문자열을 생성한 후 반환한다. 반면에 대부분의 배열 메서드는 그 자리에서 곧바로 원소를 수정한다.

```js
c = a.toUpperCase();
a === c;	// false
a;			// "foo"
c;			// "FOO"

b.push( "!" );
b;			// ["f","O","o","!"]
```

그리고 문자열을 다룰 때 유용한 대부분의 배열 메서드는 사실상 문자열에 쓸 수 없지만, 문자열에 대해 불변 배열 메서드를 빌려 쓸 수는 있다.

```js
a.join;			// undefined
a.map;			// undefined

var c = Array.prototype.join.call( a, "-" );
var d = Array.prototype.map.call( a, function(v){
	return v.toUpperCase() + ".";
} ).join( "" );

c;				// "f-o-o"
d;				// "F.O.O."
```

다음은 문자열의 순서를 거꾸로 뒤집는 코드다. 배열에는 reverse()라는 가변 메서드가 준비되 있다. 가변 메서드기 때문에 문자열이 빌려 쓸 수 없다.

```js
a.reverse;		// undefined

b.reverse();	// ["!","o","O","f"]
b;				// ["!","o","O","f"]

Array.prototype.reverse.call( a );
// 여전히 String 객체 래퍼를 반환한다 (3장 네이티브 참고)
// for "foo" :(
```

일단 문자열을 배열로 바꾸고 원하는 작업을 수행한 후 다시 문자열로 되돌릴 것이 또 다른 꼼수(전문 용어로는 Hack이라고 함)다.

```js
var c = a
	// 'a'를 문자의 배열로 분할
	.split( "" )
	// 문자 배열의 순서를 거꾸로 뒤집는다
	.reverse()
	// 문자 배열을 합쳐 다시 문자열로 되돌린다
	.join( "" );

c; // "oof"
```

'문자열'자체에 어떤 작업을 빈번하게 수행하는 경우라면 관점을 달리하여 문자열을 문자 단위로 저장하는 배열로 취급하는 것이 더 나을 수도 있다. 배열 메서드를 사용하고 문자열로 나타내야 할 때는 언제나 문자 배열에 join("") 메서드를 호출하면 된다.

## 2-3 숫자

자바스크립트의 숫자 타입은 number가 유일하며 '정수', '부동 소수점 숫자'를 모두 아우른다. 따라서 자바스크립트의 '정수'는 부동 소수점 값이 없는 값이다. (예:42.0은 '정수' 42와 같다).
자바스크립트의 number도 IEEE 754 표준을 따르며, 그중에서도 정확히는 'Double Precision' 표준 포맷(64비트 바이너리)을 사용한다.

### 2-3-1 숫자 구문

자바스크립트 숫자 리터럴은 다음과 같이 10진수 리터럴로 표시한다.

```js
var a = 42;
var b = 42.3;
```

소수점 앞 정수가 0이면 생략 가능하다.

```js
var a = 0.42;
var b = .42;
```

소수점 이하가 0일 때도 생략 가능하다.

```js
var a = 42.0;
var b = 42.; // 혼동을 일으킬 수 있지만 틀린 코드는 아니다.
```

대부분의 숫자는 10진수가 디폴트고 소수점 이하 0은 뗀다.

```js
var a = 42.300;
var b = 42.0;

a; // 42.3
b; //42
```

아주 크거나 아주 작은 숫자는 지수형으로 표시하며, toExponential() 메서드의 결괏값과 같다.

```js
var a = 5E10;
a;					// 50000000000
a.toExponential();	// "5e+10"

var b = a * a;
b;					// 2.5e+21

var c = 1 / a;
c;					// 2e-11
```

숫자 값은 Number 객체 래퍼(Wrapper)로 박싱(Boxing) 할 수 있기 때문에 Number.prototype 메서드로 접근할 수도 있다(3장 네이티브 참고). 예를 들면 toFixed() 메서드는 지정한 소수점 이하 자릿수까지 숫자를 나타낸다. 실제로는 숫자 값을 문자열 형태로 반환한다.

```js
var a = 42.59;

a.toFixed( 0 ); // "43"
a.toFixed( 1 ); // "42.6"
a.toFixed( 2 ); // "42.59"
a.toFixed( 3 ); // "42.590"
a.toFixed( 4 ); // "42.5900"
```

toPrecision()도 기능은 비슷하지만 유효 숫자 개수를 지정할 수 있다.

```js
var a = 42.59;

a.toPrecision( 1 ); // "4e+1"
a.toPrecision( 2 ); // "43"
a.toPrecision( 3 ); // "42.6"
a.toPrecision( 4 ); // "42.59"
a.toPrecision( 5 ); // "42.590"
a.toPrecision( 6 ); // "42.5900"
```

두 메서드는 숫자 리터럴에서 바로 접근할 수 있으므로 굳이 변수를 만들어 할당하지 않아도 된다. 하지만 .이 소수점일 경우엔 프로퍼티 접그나가 아닌 숫자 리터럴의 일부로 해석되므로 . 연사자를 사용할 때는 조심하자.

```js
// 잘못된 구문
42.toFixed( 3 );	// SyntaxError

// 모두 올바른 구문
(42).toFixed( 3 );	// "42.000"
0.42.toFixed( 3 );	// "0.420"
42..toFixed( 3 );	// "42.000"
42 .toFixed( 3 )l // "42.000" .앞에 공란이 있다.

// 숫자 리터럴에 이런 코딩은 가능하지만 안구테러니 하지말자
```

큰 숫자는 보통 다음과 같이 지수형으로 표시한다.

```js
var onethousand = 1E3;  // 1 * 10^3
var onemilliononehundredthousand = 1.1E6;	// 1.1 * 10^6
```

숫자 리터럴은 2진, 8진, 16진 등 다른 진법으로도 나타낼 수 있다.

```js
0xf3; // 243의 16진수
0Xf3; // 위와 같음
0363; // 243의 8진수
```
NOTE: ES6 + Strict Mode에서는 0363처럼 0을 앞에 붙여 8진수를 표시하지 못한다. Non-Strict Mode에서는 과거 형식을 계속 쓸 수는 있지만 미래를 생각해서 쓰지 않는게 좋다.

ES6부터는 다음과 같이 쓸 수 도 있다.

```js
0o363;  // 243의 8진수
0O363;	// 위와 같음

0b11110011;	// 243 이진수
0B11110011; // 위와 같음
```

헷갈리니까 언제나 소문자로 0x, 0b 0o와 같이 표기하자.

### 2-3-2 작은 소수 값

다음은 널리 알려진 이진 부동 솟수점 숫자의 부작용을 알아보자

```js
0.1 + 0.2 === 0.3; // false
```

이진 부동 소수점으로 나타낸 0.1과 0.2는 원래의 숫자와 일치하지 않는다. 그래서 둘을 더한 결과 역시 정확히 0.3이 아니다. 실제로는 0.000000000000004에 가깝지만, '같은' 것은 아니다.

대부분의 애플리케이션이 Whole Number(0 + 양의 정수)만을, 그것도 기껏해야 백만이나 조 단위 규모의 숫자를 다룬다. 이런 상황이라면 언제나 안심하고 자바스크립트의 숫자 연산 기능을 빋고 써도 된다.

그럼 0.1 + 0.2와 0.3 두숫자는 어떻게 비교해야 할까?

가장 일반적으로는 미세한 '반올림 오차'를 허용 공차(Tolerance)로 처리하는 방법이 있다. 이렇게 미세한 오차를 '머신 입실론(Machine Epsilon')이라고 하는데, 자바 스크립트 숫자의 머신 입실론은 2^-52(2.220446049250313e^-16)다.

ES6부터는 이 값이 Number.EPSILON 으로 미리 정의되어 있으므로 필요시 사용하면 되고, ES6 이전 부라우저는 다음과 같이 폴리필을 대신 사용한다.

```js
if (!Number.EPSILON) {
	Number.EPSILON = Math.pow(2,-52);
}
```

Number.EPSILON 으로 두 숫자의 (반올림 허용 오차 이내의) '동등함(equality)'을 비교할 수 있다.

```js
function numbersCloseEnoughToEqual(n1,n2) {
	return Math.abs( n1 - n2 ) < Number.EPSILON;
}

var a = 0.1 + 0.2;
var b = 0.3;

numbersCloseEnoughToEqual( a, b );					// true
numbersCloseEnoughToEqual( 0.0000001, 0.0000002 );	// false
```

부동 소수점 숫자의 최갯값은 대략 1.798e+308이고 Number.MAX_VALUE로 정의하며, 최솟값은 5e-324로 음수는 아니지만 거의 0에 가까운 숫자고 Number.MIN_VALUE로 정의한다.

### 2-3-3 안전한 정수 범위

정수는 Number.MAX_VALUE 보다 훨씬 작은 수준에서 '안전(Safe)' 값의 범위가 정해져 있다.

'안전하게' 표현할 수 있는 정수는 최대 2^53 - 1 (9007199254740991)이다. 9천조가 넘는다.

이 값은 ES6에서 Number.MAX_SAFE_INTEGER로 정의한다. 최솟값은 Number.MIN_SAFE_INTEGER로 정의하며 -9007199254740991이다.

자바스크립트 프로그램에서 이처럼 아주 큰 숫자에 맞닥뜨리는 경우는 데이터베이스 등에서 64비트 ID를 처리할 때가 대부분이다. 64비트 숫자는 숫자 타입으로 정확하게 표시할 수 없으므로(보내고 받을 때) 자바스크립트 string 타입으로 저장해야 한다. 하지만 아주 큰 숫자를 다룰 수 밖에 없는 상황이라면 큰 수(Big Number) 유틸리티 사용을 권하고 싶다.

### 2-3-4 정수인지 확인

ES6부터는 Number.isInteger()로 어떤 값의 정수 여부를 확인한다.

```js
Number.isInteger( 42 );		// true
Number.isInteger( 42.000 );	// true
Number.isInteger( 42.3 );	// false
```

ES6 이전 버전을 위한 폴리필은 다음과 같다.

```js
if (!Number.isInteger) {
	Number.isInteger = function(num) {
		return typeof num == "number" && num % 1 == 0;
	};
}
```

안전한 정수 여부는 ES6부터 Number.isSafeInteger()로 체크한다.

```js
Number.isSafeInteger( Number.MAX_SAFE_INTEGER );	// true
Number.isSafeInteger( Math.pow( 2, 53 ) );			// false
Number.isSafeInteger( Math.pow( 2, 53 ) - 1 );		// true
```

폴리필은 다음과 같다.

```js
if (!Number.isSafeInteger) {
	Number.isSafeInteger = function(num) {
		return Number.isInteger( num ) &&
			Math.abs( num ) <= Number.MAX_SAFE_INTEGER;
	};
}
```

### 2-3-5 32비트 (부호 있는) 정수

정수의 '안전 범위'가 대략 9천 조(53비트)에 이르지만, (비트 연산처럼) 32비트 숫자에만 가능한 연산이 있으므로 실제 범위는 훨씬 줄어든다. 따라서 정수의 안전 범위는 Math.pow(-2,31) 에서 Math.po(2,31) - 1 까지다. (약 -21억 ~ 21억)

a | 0 과 같이 쓰면 '숫자 값 -> 32비트 부호 있는 정수'로 강제 변환됨.
