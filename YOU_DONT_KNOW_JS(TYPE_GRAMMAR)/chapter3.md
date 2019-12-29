# CHAPTER 3 네이티브


* [3-1 내부 [[Class]]](#3-1-내부-[[Class])
* [3-2 래퍼 박싱하기](#3-2-래퍼-박싱하기)
* [3-3 언박싱](#3-3-언박싱)

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
