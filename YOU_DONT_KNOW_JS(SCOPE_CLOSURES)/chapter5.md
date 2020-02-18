# 스코프 클로저

* [5-1 깨달음](#5-1-깨달음)
* [5-2 핵심](#5-2-핵심)
* [5-3 이제 나는 볼 수 있다](#5-3-이제-나는-볼-수-있다)
* [5-4 반복문과 클로저](#5-4-반복문과-클로저)
* [5-5 모듈](#5-5-모듈)
* [5-6 정리하기](#5-6-정리하기)

## 5-1 깨달음

클로저는 자바스크립트의 모든 곳에 존재한다. 그저 인식하고 받아들이면 된다. 클로저는 새롭게 문법과 패턴을 배워야 할 특별한 도구가 아니다.

클로저는 렉시컬 스코프에 의존해 코드를 작성한 결과로 그냥 발생한다. 굳이 의도적으로 클로저를 생성할 필요도 없다. 모든 코드에서 클로저는 생성되고 사용된다. 그러므로 여기서 적절히 클로저의 전반을 파악하면 클로저를 목적에 따라 확인하고, 받아들이고, 이용할 수 있다.

깨달음의 순간이 이럴 것이다. "아, 클로저는 내 코드 전반에서 이미 일어나고 있었구나! 이제 난 클로저를 볼 수 있어.".

## 5-2 핵심

클로저는 함수가 속한 렉시컬 스코프를 기억하여 함수가 렉시컬 스코프 밖에서 실행될 때에도 이 스코프에 접근할 수 있게 하는 기능을 뜻한다.

코드를 보자.

```js
function foo() {
  var a = 2;
  function bar() {
    console.log(a); // 2
  }
  bar();
}

foo();
```

함수 bar()는 렉시컬 스코프 검색 규칙을 통해 바깥 스코프의 변수 a에 접근할 수 있다(이 경우는 RHS 참조 검색이다).

이것이 '클로저'인가? 기술적으로 보자면 그렇게 볼 수 있다. 그러나 앞에서 말한 정의 따르자면 꼭 그렇지는 않다. a를 참조하는 bar()를 설명하는 가장 정확한 방식은 렉시컬 스코프 검색 규칙에 따라 설명하는 것이고, 이 규칙은 *클로저의 일부*일 뿐이다.

순전히 학술적인 관점에서 보면 함수 bar()는 foo() 스코프에 대한 클로저를 가진다(물론 bar()는 그 외의 접근 가능한 (이 경우는 글로벌) 스코프에 대한 클로저도 가진다). 달리 말하면 bar()는 foo() 스코프에서 닫힌다. 왜? bar()는 중첩되어 foo() 안에 존재하기 때문이다. Easy~.

그러나 이런 방식으로 정의된 클로저는 바로 알아보기 힘들고 앞의 코드에서 클로저가 작동하는 방식을 볼 수도 없다. 렉시컬 스코프는 분명하게 볼 수 있지만, 클로저는 여전히 코드 뒤에 숨겨져 있다.

그럼 클로저의 정체를 확인할 코드를 보자.

```js
function foo() {
  var a = 2;
  function bar() {
    console.log(a);
  }
  return bar;
}

var baz = foo();

baz();  // 2 - 후, 클로저가 보이는구먼
```

함수 bar()는 foo()의 렉시컬 스코프에 접근할 수 있고, bar() 함수 자체를 값으로 넘긴다. 이 코드는 bar를 참조하는 함수 객체 자체를 반환한다.

반환 값(bar())을 baz 변수에 대입하고 baz() 함수를 호출 했다. 내부 함수인 bar()가 여지없이 실행 됐다. 그러나 이 경우에 함수 bar는 함수가 선언된 렉시컬 스코프 밖에서 실행됐다.

일반적으로 foo()가 실행된 후에는 foo()의 내부 스코프가 사라졌다고 생각할 것이다. 엔진이 가비지 콜렉터를 고용해 더는 사용하지 않는 메모리를 해제시키기 때문이다. 더는 foo()의 내용을 사용하지 않는 상황이라면 사라졌다고 보는게 자연스럽다.

그러나 클로저가 이를 내벼러두지 않는다. 사실 foo의 내부 스코프는 여전히 '사용 중'이므로 해제되지 않는다. 바로 bar() 자신이 사용중이기 때문이다. bar()는 foo() 스코프에 대한 렉시컬 스코프 클로저를 가지고, foo()는 bar()가 나중에 참조할 수 있도록 스코프를 살려둔다. 즉, bar()는 여전히 해당 스코프에 대한 참조를 가지는데, 그 참조를 바로 클로저라고 부른다.

즉 변수 baz를 호출할 때, 해당 함수는 원래 코드의 렉시컬 스코프에 접근할 수 있고 이는 함수가 변수 a에 접근 할 수 있다는 의미다.

함수는 원래 코드의 렉시컬 스코프에서 완전히 벗어나 호출됐다. 클로저는 호출된 함수가 원래 선언된 렉시컬 스코프에 계속해서 접근할 수 있도록 허용한다. 물론, 어떤 방식이든 함수를 값으로 넘겨 다른 위치에서 호출하는 행위는 모두 클로저가 작용한 예다.

```js
function foo() {
  var a = 2;
  function baz() {
    console.log(a); // 2
  }
  bar(baz);
}

function bar(fn) {
  fn(); // 봐봐, 클로져라구!
}
```

코드에서 함수 baz를 bar에 넘기고, 이제 fn이라 명명된 함수를 호출됐다. 이때 foo()의 내부 스코프에 대한 fn의 클로저는 변수 a에 접근할 때 확인할 수 있다. 이런 함수 넘기기는 간접적인 방식으로도 가능하다.

```js
var fn;

function foo() {
  var a = 2;
  function baz() {
    console.log(a);
  }
  fn = baz; // 글로벌 변수에 baz를 할당
}

function bar(fn) {
  fn(); // 봐봐, 클로져라구!
}

foo();

bar();  // 2
```

어떤 방식으로 내부 함수를 자신이 속한 렉시컬 스코프 밖으로 수송하든 함수는 처음 선언된 곳의 스코프에 대한 참조를 유지한다. 즉, 어디에서 해당 함수를 실행하든 클로저가 작용한다.

## 5-3 이제 나는 볼 수 있다

앞에서 본 코드들은 클로저 사용법을 보여주기 위해 다소 인위적으로 작성했다. 클로저가 모든 코드 안에 존재하는 무언가라고 이야기했다. 이제 왜 그것이 사실인지 살펴보자.

```js
function wait(message) {
  setTimeout(function timer(){
    console.log(message);
  }, 1000);
}

wait("Hello, closure!");
```

내부 함수 timer를 setTimeout()에 인자로 넘겼다. timer 함수는 wait() 함수의 스코프에 대한 소크프 클로저를 가지고 있으므로 변수 message에 대한 참조를 유지하고 사용할 수 있다.

wait() 실행 1초 후, wait의 내부 스코프는 사라져야 하지만 익명의 함수가 여전히 해당 스코프에 대한 클로저를 가지고 있다.

엔진 내부 깊숙한 곳의 내장 함수 setTimeout()에는 아마도 fn이나 func 정도로 불릴 인자의 참조가 존재한다. 엔진은 해당 함수 참조를 호출하여 내장 함수 timer를 호출하므로 timer의 렉시컬 스코프는 여전히 온전하게 남아 있다.

#### 클로저

JQuery(또는 다른 자바스크립트 프레임워크) 신봉자라면 다음을 보자.

```js
function setupBot(name, selector) {
  $(selector).click(function activator(){
    console.log("Activating: " + name);
  });
}

setupBot("Closure Bot 1", "#bot_1");
setupBot("Closure Bot 2", "#bot_2");
```

자체의 렉시컬 스코프에 접근할 수 있는 함수를 인자로 넘길 때 그 함수가 클로저를 사용하는 것을 볼 수 있다. 타이머, 이벤트 처리기, Ajax 요청, 윈도 간 통신, 웹 워커와 같은 비동기적(또는 동기적) 작업을 하며 콜백 함수를 넘기면 클로저를 사용할 준비가 된 것이다!

NOTE : 흔히 IIFE 자체가 드러난 클로저의 예라고도 하는데, 나는 동의하지 않는다. IIFE만으로는 앞에서 정의한 요소를 갖추지 못하기 때문이다.

```js
var a = 2;
(function IIFE(){
  console.log(a);
})();
```

이 코드는 '작동'하지만 엄격히 말해 클로저가 사용된 것은 아니다. 'IIFE' 함수가 자신의 렉시컬 스코프 밖에서 실행된 것이 아니기 때문이다. 변수 a는 클로저가 아니라 일반적인 렉시컬 스코프 검색을 통해 가져왔다.

클로저는 기술적으로 보면 선언할 때 발생하지만, 바로 관찰할 수 있는 것은 아니다. 누군가 말했던 것처럼 아무도 듣는 사람 없이 쓰러진 숲 속의 나무와 같다.

IIFE 자체는 클로저의 사례가 아니지만 IIFE는 틀림없이 스코프를 생성하고 클로저를 사용할 수 있는 스코프를 만드는 가장 흔한 도구의 하나다. 따라서 IIFE 자체가 클로저를 작동시키지는 않아도 확실히 클로저와 연관이 깊다.

## 5-4 반복문과 클로저

클로저를 설명하는 가장 흔하고 표준적인 사례는 for 반복문이다.

```js
for(var i = 1; i <= 5; i++) {
  setTimeout( function timer() {
    console.log(i);
  }, i * 1000);
}
```

이 코드의 목적은 '1', '2', ... , '5'까지 한 번에 하나씩 일 초마다 출력하는 것이다. 그러나 실제로 코드를 돌려보면, 일 초마다 한 번씩 '6'만 5번 출력된다.

왜?

먼저, 6이 어떻게 나오는지 보자. for문의 종료되는 조건은 i의 값이 6일 때다. 즉, 출력된 값은 반복문이 끝났을 때의 i 값을 반영한 것이다.

timeout 함수 콜백은 반복문이 끝나고 나서야 작동한다. setTimeout(... , 0)이었다 해도 해당 함수 콜백은 확실히 반복문이 끝나고 나면 동작해서 결과는 매번 6을 출력한다.

그러면 어떻게 해야 원래의 목적대로 출력할 수 있을까?

그러기 위해서는 반복마다 각각의 i 복제본을 '잡아'두는 것이다. 현재는 반복문 안 총 5개의 함수들은 반복마다 따로 정의됐음에도 모두 같이 글로벌 스코프 클로저를 공유해 해당 스코프 안에는 오직 하나의 i만이 존재한다. 따라서 모든 함수는 당연하게도 같은 i에 대한 참조를 공유한다.

그래서 반복마다 하나의 새로운 닫힌(Closured) 스코프가 필요하다.

3장에서 IIFE가 함수를 정의하고 바로 실행시키면서 스코프를 생성한다고 배웠다. 시도해보자.

```js
for(var i = 1; i <= 5; i++) {
  (function (){
    setTimeout( function timer() {
      console.log(i);
    }, i * 1000);
  })();
}
```

결과는 '6'만 5번 출력된다. 반복마다 각각의 IIFE가 생성한 자신의 스코프를 가진다. 하지만 스코프가 비어있다.

각 스코프는 자체 변수가 필요하다.

```js
for(var i = 1; i <= 5; i++) {
  (function (){
    var j = i;
    setTimeout( function timer() {
      console.log(j);
    }, j * 1000);
  })();
}
```

잘 동작한다.

약간 다른 버전을 선호하는 이들도 있다.

```js
for(var i = 1; i <= 5; i++) {
  (function (j){
    setTimeout( function timer() {
      console.log(j);
    }, j * 1000);
  })(i);
}
```

IIFE를 사용하여 반복마다 새로운 스코프를 생성하는 방식으로 timeout 함수 콜백은 원하는 값이 제대로 저장된 변수를 가진 새 닫힌 스코프를 반복마다 생성해 사용할 수 있다.

### 5-4-1 다시 보는 블록 스코프

이전 해법은 반복마다 IIFE를 사용해 하나의 새로운 스코프를 생성했다. 즉, 실제 필요했던 것은 반복 별 블록 스코프였다. 키워드 let은 본질적으로 하나의 블록을 닫을 수 있는 스코프로 바꾼다.

```js
for(var i = 1; i <= 5; i++) {
  let j = i;  // 클로저를 위한 블록 스코프
  setTimeout( function timer() {
    console.log(j);
  }, j * 1000);
}
```

그러나 그게 다가 아니다! let 선언문이 for 반복문 안에 사용되면 특별한 방식으로 동작한다. 반복문 시작 부분에서 let으로 선언된 변수는 한 번만 선언되는 것이 아니라 반복할 때마다 선언된다.

```js
for(let i = 1; i <= 5; i++) {
  setTimeout( function timer() {
    console.log(i);
  }, i * 1000);
}
```

블록 스코프와 클로저가 함께 활약해서 모든 문제를 해결했다.

## 5-5 모듈

클로저의 능력을 활용하면서 표면적으로는 콜백과 상관없는 코드 패터늘이 있다. 그중 가장 강력한 패턴인 모듈을 살펴보자.

```js
function foo() {
  var something = "cool";
  var another = [1, 2, 3];

  function doSomething() {
    console.log(something);
  }

  function doAnother() {
    console.log(another.join(" ! "));
  }
}
```

이 코드에는 클로저의 흔적이 보이지 않는다. 변수 something, another 그리고 내부 함수 doSomething(), doAnother() 같은 몇 가지 비공개 데이터가 있다. 이들 모두 foo()의 내부 스코프를 렉시컬 스코프(당연히 클로저도 따라온다)로 가진다.

```js
function CoolModule(){
  var something = "cool";
  var another = [1, 2, 3];

  function doSomething() {
    console.log(something);
  }

  function doAnother() {
    console.log(another.join(" ! "));
  }

  return {
    doSomething: doSomething,
    doAnother: doAnother
  };
}

var foo = CoolModule();

foo.doSomething();  // cool
foo.doAnother();  // 1 ! 2 ! 3
```

이 코드와 같은 자바스크립트 패턴을 모듈이라고 부른다. 가장 흔한 모듈 패턴 구현 방법은 모듈 노출(Revealing Module)이고, 앞의 코드는 이것의 변형이다.

앞의 코드를 확인해 보자.

첫째, CoolModule()은 그저의 하나의 함수다. 그러나 모듈 인스턴스를 생성하려면 반드시 호출해야만 한다. 최외곽 함수가 실행되지 않으면 내부 스코프와 클로저는 생성되지 않는다.

둘째, CoolModule() 함수는 객체를 반환한다. 반환되는 객체는 객체-리터럴 문법 { key:value, ...}에 따라 표기된다. 해당 객체는 내장 함수들에 대한 참조를 가진다. 하지만 내장 데이터 변수는 비공개로 숨겨져 있다. 이 객체의 반환 값은 본질적으로 모듈의 공개 API라고 생각할 수 있다.

객체의 반환 값은 최종적으로 외부 변수 foo에 대입되고, foo.doSomething()과 같은 방식으로 API의 속성 메서드에 접근할 수 있다.

함수 doSomething()과 doAnother()는 모듈 인스턴스의 내부 스코프에 포함하는 클로저를 가진다.

이 모듈 패턴을 사용하려면 두 가지 조건이 필요하다.

1. 하나의 최외곽 함수가 존재하고, 이 함수가 최소 한번은 호출되어야 한다(호출할 때마다 새로운 모듈 인스턴스가 생성된다).
2. 최외곽 함수는 최소 한 번은 하나의 내부 함수를 반환해야 한다. 그래야 해당 내부 함수가 비공개 스코프에 대한 클로저를 가져 비공개 상태에 접근하고 수정할 수 있다.

하나의 함수 속성만을 가지는 객체는 진정한 모듈이 아니다. 함수 실행 결과로 반환된 객체에 데이터 속성들은 있지만 닫힌 함수가 없다면, 당연히 그 객체는 진정한 모듈이 아니다.

이번엔 오직 하나의 인스턴스, '싱글톤'만 생성하는 모듈을 살펴보자.

```js
var foo = (function CoolModule(){
  var something = "cool";
  var another = [1, 2, 3];

  function doSomething() {
    console.log(something);
  }

  function doAnother() {
    console.log(another.join(" ! "));
  }

  return {
    doSomething: doSomething,
    doAnother: doAnother
  };
})();

foo.doSomething();  // cool
foo.doAnother();  // 1 ! 2 ! 3
```

모듈 함수를 IIFE로 바꾸고 즉시 실행시켜 반환 값을 직접 하나의 모듈 인스턴스 확인자 foo에 대입시켰다.

효과적인 모듈 패턴 중 하나느 ㄴ다음 코드와 같이 공개 API로 반환하는 객체에 이름을 정하는 방식이다.

```js
var foo = (function CoolModule(id){
  function change() {
    // public API 수정
    publicAPI.identify = identify2;
  }

  function identify1() {
    console.log(id);
  }

  function identify2() {
    console.log(id.toUpperCase());
  }

  var publicAPI = {
    change: change,
    identify: identify1
  };

  return publicAPI;
})("foo module");

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

### 5-5-1 현재의 모듈

많은 모듈 의존성 로더와 관리자는 본질적으로 이 패턴의 모듈 정의를 친숙한 API 형태로 감싸고 있다.

```js
var MyModules = (function Manager(){
  var modules = {};

  function define(name, deps, impl){
    for(var i = 0; i < deps.length; i++){
      deps[i] = modules[deps[i]];
    }
    modules[name] = imp.apply(impl, deps);
  }

  function get(name) {
    return modules[name];
  }

  return {
    define: define,
    get: get
  };
})();
```

이 코드의 핵심부는 "modules[name] = impl.appy(impl, deps)"다. 이 부분은 (의존성을 인자로 넘겨) 모듈에 대한 정의 래퍼 함수를 호출하여 반환 값인 모듈 API를 이름으로 정리된 내부 모듈 리스트에 저장한다.

해당 부분을 이용해 모듈을 정의하는 다음 코드를 보자.

```js
MyModules.define("bar",[], function(){
  function hello(who){
    return "Let me introduce: " + who;
  }

  return {
    hello: hello
  };
});

MyModules.define("foo", ["bar"], function(){
  var hungry = "hippo";
  function awesome(){
    console.log( bar.hello(hungry).toUpperCase());
  }

  return {
    awsome: awesome
  };
});

var bar = MyModules.get("bar");
var foo = MyModules.get("foo");

console.log(
  bar.hello("hippo")
);  // Let me introduce: hippo

foo.awesome();  // LET ME INTRODUCE: HIPPO
```

'foo'와 'bar' 모듈은 모두 공개 API를 반환하는 함수로 정의됐다. 'foo'는 심지어 'bar'의 인스턴스를 의존성 인자로 받아 사용할 수도 있다.

모듈 관리자를 만드는 특별한 마법이란 존재하지 않는다. 모듈 관리자는 앞에서 언급한 모듈 패턴의 특성을 모두 가진다. 이들은 함수 정의 래퍼를 호출하여 해당 모듈의 API인 변환 값을 저장한다. 모듈은 그저 모듈일뿐이다.

### 5-5-2 미래의 모듈

ES6는 모듈 개념을 지원하는 최신 문법을 추가했다. 모듈 시스템을 불러올 때 ES6는 파일을 개별 모듈로 처리한다. 각 모듈은 다른 모듈 또는 특정 API 멤버를 불러오거나 자신의 공개 API 멤버를 내보낼 수 있다.

NOTE: 함수 기반 모듈은 정적으로 알려진(컴파일러가 읽을 수 있는) 패턴이 아니다. 따라서 이들 API의 의미느는 런타임 전까지 해석되지 않는다. 즉, 실제로 모듈의 API를 런타임에 수정할 수 있다는 말이다. 반면 ES6 모듈 API는 정적이다. 따라서 캄파일러는 컴파일레이션 중에  불러온 모듈의 API 멤버 참조가 실제로 존재하는지 확인한다. API 참조가 존재하지 않으면 컴파일러는 컴파일 시 초기 오류를 발생시킨다.

ES6 모듈은 inline 형식을 지원하지 않고, 반드시 개별 파일(모듈당 파일 하나)에 정의되어야 한다. 브라우저와 엔진은 기본 모듈 로더를 가진다. 모듈을 불러올 때 모듈 로더는 동기적으로 모듈 파일을 불러온다.

```js
// bar.js
function hello(who) {
  return "Let me introduce: " + who;
}

export hello;

// foo.js: "bar" 모듈로부터 오직 'hello()'만 import 
import hello from "bar";

var hungry = "hippo";function awesome(){
    console.log( bar.hello(hungry).toUpperCase());
  }

  return {
    awsome: awesome
  };
}

export awesome;

// baz.js: "foo"와 "bar" 전체를 import
module foo from "foo";
module bar from "bar";

console.log(
  bar.hello("rhino");
);  // Let me introduce: rhino

foo.awesome();  // LET ME INTRODUCE: HIPPO
```

키워드 import는 모듈 API에서 하나 이상의 멤버를 불러와 특정 변수(여기서는 hello)에 묶어 현재 스코프에 저장한다. 키워드 module은 모듈 API 전체를 불러와 특정 변수(여기서는 foo와 hello)에 묶는다. 키워드 export는 확인자를 현재 모듈의 공개 API로 내보낸다. 이 연산자들은 모듈의 정의에 따라 필요하면 얼마든지 사용할 수 있다.

앞서 살펴본 함수-클로저 모듈처럼 모듈 파일의 내용은 스코프 클로저에 감싸진 것으로 처리된다.

## 5-6 정리하기

클로저는 사실 표준이고, 함수를 값으로 마음대로 넘길 수 있는 렉시컬 스코프 환경에서 코드를 작성하는 방법이다.

클로저는 함수를 렉시컬 스코프 밖에서 호출해도 함수는 자신의 렉시컬 스코프를 기억하고 접근할 수 있는 특성을 의미한다.

또한, 클로저는 다양한 형태의 모듈 패턴을 가능하게 하는 매우 효과적인 도구다.

모듈은 두 가지 특성을 가져야 한다.

1. 최외곽 래퍼 함수를 호출하여 외곽 스코프를 생성한다.
2. 래핑 함수의 변환 값은 반드시 하나 이상의 내부 함수 참조를 가져야 하고, 그 내부 함수는 래퍼의 비공개 내부 스코프에 대한 클로저를 가져야 한다.