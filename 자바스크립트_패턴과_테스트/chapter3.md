# 3장 객체를 바르게 만들기

* [3-1 원시형](#3-1-원시형)
* [3-2 객체 리터럴](#3-2-객체-리터럴)
* [3-3 모듈 패턴](#3-3-모듈-패턴)
* [3-4 객체 프로토타입과 프로토타입 상속](#3-4-객체-프로토타입과-프로토타입-상속)
* [3-5 new 객체 생성](#3-5-new-객체-생성)
* [3-6 클래스 상속](#3-6-클래스-상속)
* [3-7 함수형 상속](#3-7-함수형-상속)
* [3-8 멍키 패칭](#3-8-멍키-패칭)
* [3-9 정리하기](#3-9-정리하기)

이 장의 주제

* 원시형 객체 리터럴, 모듈 형태로 데이터를 생성한다.
* 데이터를 new 키워드로 생성한다.
* 객체 생성 메서드로 객체를 올바르게 만든다.
* 프로토타입, 고전적, 함수형 패턴을 동원하여 상속을 구사한다.
* 멍키 패치를 바르게 사용한다.

## 3-1 원시형

ECMAScript5 명세 기준으로 자바스크립트의 원시형은 String, Number, Boolean, null, undefined 5개가 전부다.(ECMAScript 6부터 심볼이 추가됐다. 객체는 원시형에 속하지 않는다)

원시형 변수는 값은 있되 프로퍼티가 없어서 다음 코드는 에러가 날 것 같지만, 문제 없이 실행된다.

```js
var str = "abcde";
console.log(str.length); // 실행 결과: 5
```

자바스크립트 엔진이 str에서 String 객체를 만들고 이 객체의 length 프로퍼티 값을 참조한다. 물론 이렇게 만든 String 객체는 곧바로 가비지 컬렉션 대상이 된다.

문자열, 불, 숫자 타입 모두 그들만의 객체 래퍼(wrapper), 즉 String(값), Boolean(값), Number(값)를 지닌다.

동물 체중을 킬로그램(kg) 단위로 표시한 변수가 있고 입력한 값을 다음과 같이 처리한다고 하자.

```js
if (inputMass < 0) {
  // 체중이 음수일 리 없다는 에러 메시지를 낸다.
} else if (inputMass > 150000) {
  // 에러: 흰수염고래도 이렇게 무겁진 않다고요!
}
```

그리고 이 코드를 다른 곳에서 똑같이 복사해 쓴다. DRY와는 거리가 먼 이런 코드를 개선할 수 없을까? 범위 체크 기능을 심을 객체로 변환하면 된다. 원시형을 무조건 객체로 바꾸라는 건 아니고 상황에 맞게 고려해볼 수 있다.

원시형을 자꾸 반복하는 건 좋지 않다. 변수에 값을 넣고 다른 곳에서 참조하느니 아무래도 그냥 값을 한번 더 입력해넣는 게 알기 쉽고 편하다.

드문 예외가 있긴 하지만, 한 번 이상 참조할 상수는 변수에 담아두고 변수를 대신 참조하라. 상수를 의존성으로 주입하는 방법도 있다. 이러헥 해야 코드를 DRY하게 유지하면서 특정 갑을 사용하는 코드를 찾기가 쉽다.

#### 원시형의 SOLID/DRY 요약표

| 원칙 | 결과 |
|-|-|
|단일 책임|O|
|개방/폐쇄|확장에 개방적 X, 변경에 폐쇄적 O, 원시형은 불변값(immutable)|
|리스코프 치환|해당 없음|
|인터페이스 분리|인터페이스 구현 불가능|
|의존성 역전|의존성 같은 건 없음|
|DRY|유혹을 느끼는 부분|

## 3-2 객체 리터럴

객체 리터럴(object literal)은 다음과 같이 선언한 객체를 말한다.

```js
{name: 'Koko', genus: 'gorilla', genius: 'sign language'}
```

객체 리터럴은 두 가지 생성 방법이 있다.

먼저 단순 객체 리터럴(bare object literal)이다.

```js
{name: 'Koko', genus: 'gorilla', genius: 'sign language'};
```

이번에는 객체 리터럴이 함수 반환값인 경우다.

```js
var amazeTheWorld = function() {
  // ...
  return {name: 'Koko', genus: 'gorilla', genius: 'sign language'};
}

var koko = amazeTheWorld();
```

둘 중 하나가 다른 하나보다 더 DRY하다.

같은 프로퍼티를 지닌 객체 리터럴을 여럿 생성할 때 계속 반복되는 프로퍼티명을 입력하다 보면 실수하기 마련이다. 사실 TDD 방식으로 어떤 함수가 원하는 프로퍼티를 지닌 객체를 반환하는지는 충분히 확인할 수 있다. 하지만 단순 객체 리터럴만으로는 직접 테스트할 방법이 마땅치 않다.

객체 리터럴은 함수 프로퍼티를 가질 수 있는데, 단순 객체 리터럴은 마찬가지로 테스트할 도리가 없다.

애스팩트를 적용하려면 포인트컷(예: 변수명)이 있어야 하는데 객체 리터럴은 이름 자체가 없기 때문에 문제가 된다. 하지만 팩토리 함수로 생성하면 반환된 리터럴을 갖고 놀 after 애스팩트에 함수를 래핑할 수 있다.

단순 객체 리터럴에서는 의존성 주입은 아예 시도조차 할 수 없지만, 리터럴을 생성/반환하는 함수는 의존성을 주입하는 과정에 아주 잘 어울린다.

단순 객체 리터러은 생성 시 검증 할 수 없다는 것도 문제다. 그러므로 객체 리터럴은 싱글톤 또는 확실히 테스트를 마친 코드에서 생성된 객체 리터럴이 아니라면 중요한 애플리케이션 부분에 쓰지 않는 편이 좋다.

객체 리터럴은 한 곳에서 다른 곳으로 데이터 뭉치를 옮길 때 쓰기 편하다. 함수 인자가 워낙 많아 그 순서를 정확히 맞추기 쉽지 않을 때 객체 리터럴을 사용하면 유용하다. 리터럴에 프로퍼티가 하나도 없다는 건 기본값을 사용하라는 신호로 받아들일 수 있다.

쓰기 편하지만 한 가지 명심하자! 함수가 어떤 프로퍼티 조합도 대비할 수 있으려면 그만큼 테스트를 많이 해야 하는 부담이 생긴다. 생성 방식을 잘 다스릴 수 있고 테스트를 확실히 마친, 말하자면 isValid 같은 메서드로 검증 가능한 객체를 대용하는 방안을 고민하라.

#### 객체 리터럴의 SOLID/DRY 요약표

| 원칙 | 결과 |
|-|-|
|단일 책임|단순 객체 리터럴은 아주 작은 편이어서 이 평가 항목에 문제가 될 만한 부분은 없다. 모듈 API를 구성하는 덩치 큰 객체 리터럴은 자신의 모듈이 담당한 모든 책임을 진다.|
|개방/폐쇄|객체 리터럴 특성상 제멋대로 확장될지 모르니 조심하라!|
|리스코프 치환|해당 없음|
|인터페이스 분리|모듈 패턴 및 멍키 패칭을 참고|
|의존성 역전|단순 객체 리터럴은 내부에 의존성을 주입할 생성자가 없으니 불가능|
|DRY|싱글톤이 아닌 단순 객체 리터럴은 WET한 코드가 되기 일쑤다. 반드시 유념하라!|

## 3-3 모듈 패턴

모듈 패턴은 자바스크립트에서 가장 명망 높은 패턴 중 하나다. 데이터 감춤이 주목적인 함수가 모듈 API를 이루는 객체를 반환하게 한다. 임의로 함수를 호출하여 생성하는 모듈과 선언과 동시에 실행하는 함수에 기반을 둔 모듈이 있다.

### 3-3-1 임의 모듈 생성

다음 예제는 원하는 시점에 언제라도 생성할 수 있는 모듈로, 모듈 함수를 호출하여 API를 얻는다.

```js
// 해당 애플리케이션에서만 사용할 수 있는 모든 객체(모듈)를 담아 넣은
// 전역 객체를 선언하여 이름공간처럼 활용한다.
var MyApp = MyApp || {};

// 애플리케이션 이름공간에 속한 모듈
// 이 함수는 animalMaker라는 다른 함수에 의존하며 animalMaker는 주입 가능하다.
MyApp.wildlifePreserveSimulator = function(animalMaker) {
  // 프라이빗 변수
  var animals = [];

  // API를 반환
  return {
    addAnimal: function(species, sex) {
      animals.push(animalMaker.make(species, sex));
    },
    getAnimalCount: function() {
      return animals.length;
    }
  };
};
```

모듈은 다음과 같이 사용한다.

```js
var preserve = Myapp.wildlifePreserveSimulator(realAnimalMaker);
preserve.addAnimal(gorilla, female);
```

이 모듈은 객체 리터럴을 반환하나 animalMaker 같은 의존성을 외부 함수에 주입하여 리터럴에서 참조하게 만들 수 있다.

다른 모듈에 주입할 수 있어 확장성이 좋다. 옛 버전 모듈을 새 버전 모듈에 주입하여 원하는 대로 래핑, 표출, 확장 등을 꾀할 수 있다. 개방/폐쇄 원칙이 최우선 관심사라면 모듈만한 것도 또 없다.

MyApp.wildlifePreserveSimulator에 after 어드바이스르 ㄹ넣어 반환된 객체에 애스팩트를 적용하는 방법도 있다. 이 어드바이스는 변환된 리터럴을 쥐고 있다가 필요에 따라 이 리터럴을 다른 애스팩트로 수정하게 될 것이다.

### 3-3-2 즉시 실행 모듈 생성

API를 반환하는 건 임의 모듈과 같지만, 외부 함수를 선언하자마자 실행하는 방법이다. 반환된 API는 이름공간을 가진 전역 변수에 할당된 후 해당 모듈의 싱글톤 인스턴스가 된다.

```js
var MyApp = MyApp || {};

MyApp.WildlifePreserveSimulator = (function() {
  // 프라이빗 변수
  var animals = [];

  // API를 반환
  return {
    addAnimal: function(animalMaker, species, sex) {
      animals.push(animalMaker.make(species, sex));
    },
    getAnimalCount: function() {
      return animals.length;
    }
  };
}()); // 즉시 실행한다
```

싱글톤은 이렇게 사용한다.

```js
MyApp.WildPreserveSimulator.addAnimal(realAnimalMaker, gorilla, female);
```

외부 함수는 애플리케이션 기동 코드의 실행과 상관없이 코드가 장성된 지점에서 즉시 실행된다. 따라서 함수 (즉시) 실행 시 의존성을 가져오지 못하면 외부 함수에 주입할 수 없다. 싱글톤이 꼭 필요하다면 임의 모듈 패턴으로 모듈을 코딩하고 해당 모듈을 요청할 때마다 의존성 주입 프레임워크에서 같은 인스턴스를 제공하는 편이 의존성 주입 측면에서 더 낫다.

### 3-3-3 모듈 생성의 원칙

임의든 즉시든 모듈을 생성할 때는 다음 사항을 유념하기 바란다.

첫째, 단일 책임 원칙을 잊지 말고 한 모듈에 한 가지 일만 시키자. 그래야 결속력이 강하고 다루기 쉬운, 아담한 API를 작성하게 된다.

둘째, 모듈 자신이 쓸 객체가 필요하다면 의존성 주입 형태로 (직접 또는 팩토리 주입 형태로) 이 객체를 제공하는 방안을 고려하라.

셋째, 다른 객체 로직을 확장하는 모듈은 해당 로직의 의도가 바뀌지 않도록 분명히 밝혀라(리스코프 치환 원칙).

#### 모듈 패턴에 관한 SOLID/DRY 요약표

| 원칙 | 결과 |
|-|-|
|단일 책임|모듈은 태생 자체가 의존성 주입과 친화적이고 애스팩트 지향적이라 단일 책임 유지는 어렵지 않다.|
|개방/폐쇄|다른 모듈에 주입하는 형태로 얼마든지 확장할 수 있다. 통제해야 할 모듈은 수정하지 못하게 차단할 수 있다.|
|리스코프 치환|의존성의 의미를 뒤바꾸는 일만 없으면 별문제 없다.|
|인터페이스 분리|결합된 API 모듈 자체가 자바스크립트에서 분리된 인터페이스나 다름없다.|
|의존성 역전|임의 모듈은 의존성으로 주입하기 쉽다. 모듈이 어떤 형태든 다른 모듈에 주입할 수 있다.|
|DRY|제대로만 쓴다면 DRY한 코드를 유지하는데 아주 좋은 방법이다.|

## 3-4 객체 프로토타입과 프로토타입 상속

자바스크립트 객체는 생성 메커니즘과 무관하게 프로토타입 객체로 연결되어 프로퍼티를 상속한다.

### 3-4-1 기본 객체 프로토타입

앞 절의 객체 리터럴은 저절로 내장 객체 Object.prototype에 연결 된다.

```js
var chimp = {
  hasTumbs: true,
  swing: function() {
    return '나무 꼭대기에 대롱대롱 매달림';
  }
};

chimp.toString(); // [object Object]
```

toString() 실행 시 undefined 함수 에러가 발생하지 않음. toString 함수가 없다는 사실을 알고 chimp의 프로토타입인 Object.protorype에 정의된 toString을 실행

### 3-4-2 프로토타입 상속

자바스크립트의 프로토타입 상속은 기본 프로토타입을 맞춤형 프로토 타입으로 대체할 때 그 진가가 드러난다.

ECMAScript 5부터 등장한 Object.create 메서드를 사용하면 기존 객체와 프로토타입이 연결된 객체를 새로 만들 수 있다.

```js
var ape = {
  hasTumbs: true,
  hasTail: false,
  swing : function() {
    return '메달리기';
  }
};

var chimp = Object.create(ape);

var bonobo = Object.create(ape);
bonobo.habitat = '중앙 아프리카';

console.log(bonobo.habitat); // '중앙 아프리카' (bonobo 프로퍼티)
console.log(bonobo.hsTail); // false (ape 프로토타입)
console.log(chimp.swing()); // '매달리기' (ape 프로토타입)
```

bonobo에 직접 추가한 habitat 프로퍼티는 ape, chimp 그 누구와도 공유하지 않는 고유 프로퍼티다.

ape는 공유 프로퍼티라서 수정하면 chimp와 bonobo 모두에 즉시 영향을 미친다.

```js
ape.hasTumbs = false;
console.log(chimp.hasThumbs); // false
console.log(bonobo.hasThumbs);  // false
```

### 3-4-3 프로토타입 체인

프로토타입 체인(prototype chain)이라는 다층 프로토타입을 이용하면 여러 계층의 상속을 구현할 수 있다.

```js
var primate = {
  stereoscopicVision: true;
};

var ape = Object.create(primate);
ape.hasTumbs = true;
ape.hasTail = false;
ape.swing = function() {
  return "매달리기";
};

var chimp = Object.create(ape);

console.log(chimp.hasTail); // false (ape 프로토타입)
console.log(chimp.stereoscopicVision);  // true (primate 프로토타입)
```

자바스크립트 엔진은 객체의 고유 프로퍼티가 없으면 프로토타입 체인을 따라 올라간다. 그래도 발견하지 못하면 undefined를 반환한다.

너무 깊숙이 프로토타입 체인을 찾게 하면 성능상 좋을 게 없으니 너무 깊게 사용하지는 말자.

## 3-5 new 객체 생성

new 객체 생성 패턴과 그 장점 및 유의 사항을 살펴보자.

### 3-5-1 new 객체 생성 패턴

객체를 new로 생성하는 구문 패턴은 C#, C++, 자바 등과 비슷하다.

다음 예제의 Marsupial 함수는 new 객체 생성 패턴으로 자신의 객체 인스턴스를 생성한다.

```js
// marsupial은 유대류 의미(캥거루 코알라 등)
function Marsupial(name, nocturnal) {
  this.name = name;
  this.isNocturnal = nocturnal;
}

var maverick = new Marsupial('매버릭', true);
var slider = new Marsupial('슬라이더', false);

console.log(maverick.isNocturnal); // true
console.log(maverick.name);        // "매버릭"

console.log(slider.isNocturnal);   // false
console.log(slider.name);          // "슬라이더"
```

Marsupial 함수는 주어진 인자를 내부적으로 생서할 인스턴스의 프로퍼티에 할당한다.

#### '나쁜 일'이 벌어질 가능성

자바스크립트 언어는 Marsupial 함수를 **생성자 함수**(new 키워드와 함께 사용하려고 작성한 함수)로 사용하라고 강요하지 않는다.

더글라스 크락포트 같은 사람은 생성자 함수 호출 시 new를 빠뜨리면 '나쁜 일'이 생길 수 있으니 아예 생성자 함수를 쓰지 말라고 권한다.

하지만 생성자 함수는 초기화 코드를 공유할 때 유용하고 new가 꼭 필요하다면 체크할 방법이 없는 것도 아니니 섣불리 생성자 함수에 퇴장을 명하고 싶지 않다.

#### new를 사용하도록 강제

instanceof 연산자를 통해 우회적으로 강제하는 방법이 있다.

```js
function Marsupial(name, nocturnal) {
  if (!(this instanceof Marsupial)) {
    throw new Error("이 객체는 new를 사용하여 생성해야 합니다");
  }
  this.name = name;
  this.isNocturnal = nocturnal;
}

var slider = Marsupial('슬라이더', true); // Uncaught Error: 이 객체는 new를 사용하여 생성해야 합니다
```

>> NOTE: instanceof의 작동 원리는 이렇다. 우변 피연산자의 프로토타입이 좌변 피연산자의 프로토타입 체인에 있는지 찾아본다. 만약 있으면 좌변 피연산자가 우변 피연자의 인스턴스라고 결론 내린다. new 키워드를 앞에 붙여 생성자 함수를 실행하면 일단 빈 객체를 하나 만들어 새 객체의 프로토타입을 생성자 함수의 프로토타입 프로퍼티에 연결한다. 그런 다음 생성자 함수를 this로 실행하여 새 객체를 찍어낸다.

에러를 내지 않고 자동으로 new를 붙여 인스턴스를 만들어 반환하게 할 수도 있다.

```js
function Marsupial(name, nocturnal) {
  if (!(this instanceof Marsupial)) {
    return new Marsupial(name, nocturnal);
  }
  this.name = name;
  this.isNocturnal = nocturnal;
}

var slider = Marsupial('슬라이더', true);

console.log(slider.name);  // '슬라이더'
```

이렇게 처리하면 개발자는 Marsupial 함수 호출부에서 new를 신경 쓸 이유가 없다.

그러나 얼핏 편리해 보이지만 사실 개발자의 실수를 모면하게 해줄 뿐이다.

일관성은 곧 믿음성이라 생각하는 우리는 new가 빠졌을 때 예외가 발생하면 Marsupial 객체 인스턴스가 모두 같은 방식으로 생성되었다고 확신할 수 있다. 결국 보다 일관적이고 분명한 코드베이스를 구축할 수 있다. 또한, 테스트 주도 개발과 접목해 new가 빠져 발생한 예외를 빠짐없이 곧바로 알아볼 수 있는 이점도 있다.

new 객체 생성 패턴을 이용하면 정의부 하나로 여러 인스턴스가 함께 사용할 함수 프로퍼티를 생성할 수 있다. 각 인스턴스는 자신만의 고유한 함수를 가진다.

```js
function Marsupial(name, nocturnal) {
  if (!(this instanceof Marsupial)) {
    throw new Error("이 객체는 new를 사용하여 생성해야 합니다");
  }
  this.name = name;
  this.isNocturnal = nocturnal;

  // 각 객체 인스턴스는 자신만의 isAwake 사본을 가진다
  this.isAwake = function(isNight) {
    return isNight === this.isNocturnal;
  }
}

var maverick = new Marsupial('매버릭', true);
var slider = new Marsupial('슬라이더', false);

var isNightTime = true;

console.log(maverick.isAwake(isNightTime));      // true
console.log(slider.isAwake(isNightTime));         // false

// 각 객체는 자신의 isAwake 함수를 가진다
console.log(maverick.isAwake === slider.isAwake); // false
```

함수 프로퍼티를 생성자 함수의 프로토타입에 붙일 수도 있다. 생성자 함수의 프로토타입에 함수를 정의하면 객체 인스턴스를 대량 생성할 때 함수 사본 개수를 한 개로 제한하여 메모리 점유율을 낮추고 성능까지 높이는 추가 이점이 있다.

```js
function Marsupial(name, nocturnal) {
  if (!(this instanceof Marsupial)) {
    throw new Error("이 객체는 new를 사용하여 생성해야 합니다");
  }
  this.name = name;
  this.isNocturnal = nocturnal;
}
// 각 객체 인스턴스는 자신만의 isAwake 사본을 가진다
Marsupial.prototype.isAwake = function(isNight) {
  return isNight === this.isNocturnal;
}
var maverick = new Marsupial('매버릭', true);
var slider = new Marsupial('슬라이더', false);

var isNightTime = true;

console.log(maverick.isAwake(isNightTime));       // true
console.log(slider.isAwake(isNightTime));         // false

// 객체들은 isAwake의 단일 인스턴스를 공유한다
console.log(maverick.isAwake === slider.isAwake); // true
```

#### new 생성 객체의 SOLID/DRY 요약표

| 원칙 | 결과 |
|-|-|
|단일 책임|가능. 하지만 생성한 객체가 반드시 한 가지 일에만 전념토록 해야 한다. 생성자 함수에 의존성을 주입할 수 있다는 점에서 도움이 될 것이다.|
|개방/폐쇄|그렇다. 다음 절에서 상속을 다룰 때 new로 생성한 객체를 어떻게 확장할 수 있는지 알게 됨.|
|리스코프 치환|상속을 잘 이용하면 가능.|
|인터페이스 분리|상속과 다른 공유 패턴을 이용하면 가능.|
|의존성 역전|의존성은 어렵지 않게 생성자 함수에 주입할 수 있다.|
|DRY|new 객체 생성 패턴을 쓰면 아주 DRY한 코드가 된다. 다만, AOP를 이 패턴과 함께 잘 써먹을 방법이 떠오르지 않아 안타깝다. new는 생성할 객체의 프로토타입을 상속한 객체를 생성하므로 AOP와 new는 친구가 되기 어렵다. 이 객체를 애스팩트로 래핑하면 하댕 객체의 프로토타입이 아닌 애스팩트의 프로토타입을 사용하게 될 것이다. 그러나 new로 만든 객체의 프로토타입에 있는 함수를 AOP로 장식하는 건 얼마든지 가능.|

## 3-6 클래스 상속

프로토타입 상속으로 어느 정도 흉내를 낼 수는 있다.

### 3-6-1 고전적 상속 흉내 내기

앞 절의 Marsupial 함수를 확장해 캥거루에 hop 함수라는 특정 프로퍼티를 넣는다고 하자.

Marsupial 함수 프로토타입에 hop 함수를 추가할 수 있지만, 그렇게 되면 Marsupial 생성자 함수로 만든 객체 인스턴스는 모두 hop 함수를 갖게 된다.

최선의 미더운 해결책은 Marsupial을 상속한 Kangaroo 함수를 생성한 뒤 확장하는 것이다.

```js
function Marsupial(name, nocturnal) {
  if (!(this instanceof Marsupial)) {
    throw new Error("이 객체는 new를 사용하여 생성해야 합니다");
  }
  this.name = name;
  this.isNocturnal = nocturnal;
}
Marsupial.prototype.isAwake = function(isNight) {
  return isNight == this.isNocturnal;
};

function Kangaroo(name) {
  if (!(this instanceof Kangaroo)) {
    throw new Error("이 객체는 new를 사용하여 생성해야 합니다");
  }
  this.name = name;
  this.isNocturnal = false;
}

Kangaroo.prototype = new Marsupial();
Kangaroo.prototype.hop = function() {
  return this.name + "가 껑충 뛰었어요!";
};
var jester = new Kangaroo('제스터');
console.log(jester.name);

var isNightTime = false;
console.log(jester.isAwake(isNightTime)); // true
console.log(jester.hop());  // '제스터가 껑충 뛰었어요!'

console.log(jester instanceof Kangaroo);  // true
console.log(jester instanceof Marsupial); // true
```

결론적으로 Kangaroo의 프로토타입이 된 Marsupial 인스턴스에 hop 함수를 추가하여 확장할 수 있게 됐다. hop 함수는 Marsupial 생성자 함수는 물론 Marsupial 생성자 함수는 물론 Marsupial 생성자 함수의 프로토타입 어느 쪽에도 추가되지 않는다. 개방/폐쇄 원칙을 제대로 실천한 사례다.

### 3-6-2 반복이 캥거루 잡네

고전적 상송을 흉내 내면 코드 반복과 메모리 점유는 피하기 어렵다.

```js
Kangaroo.prototype = new Marsupial();
```

Marsupial 생성자 함수에 인자가 하나도 없다. 프로토타입 지정 시 인자를 알 수 없으므로 Marsupial 함수의 프로퍼티 할당 작업은 Kangaroo 함수에서도 되풀이된다.

명백한 DRY 원칙 위반이다. 불필요한 반복은 부실한 코드를 양산한다.

또한, Kangaroo 프로토타입(isNocturnal: false, name: "Jester")과 Kangaroo 인스턴스(isNocturnal: undefined, name: undefined) 자신까지 프로퍼티를 들고 다니는 꼴이 된다.

| 원칙 | 결과 |
|-|-|
|단일 책임|단일 책임 원칙을 지원하지만 강제하지는 못한다. 의존성을 주입하면 여러 책임을 객체들에 전가하지 않게끔 할 수 있다. new 키워드를 사용하면 생성자 함수를 애스팩트로 장식할 수 없다.|
|개방/폐쇄|이 패턴의 주제가 바로 개방/폐쇄 원칙이다.|
|리스코프 치환|의존성을 수정하는게 아니라 확장하려는 것이다. 따라서 원칙에 충실하다.|
|인터페이스 분리|해당 없음.|
|의존성 역전|상속하는 객체의 생성자 함수에 의존성을 주입하는 형태로 실현.|
|DRY|그다지 관련이 없다. 초기화 로직이 상속을 주고받는 객체 모두의 생성 함수에 걸쳐 반복된다. 하지만 프로토타입을 공유하면 함수 사본 개수를 줄일 수 있다.|

## 3-7 함수형 상속

함수형 상속을 하면 데이터를 숨긴 채 접근을 다스릴 수 있다. 이렇게 할 수만 있다면 퍼블릭/프라이빗 데이터 모두 실수와 오용에 노출된 빈도가 줄어들어 믿음성이 커진다.

marsupial을 상속한 이후 hop 함수를 추가한 kangaroo 객체를 만드는 일이 관건인데, 함수형 상속 패턴과 모듈을 이용하여 구현해보자.

```js
var AnimalKingdom = AnimalKingdom || {};

AnimalKingdom.marsupial = function(name, nocturnal) {

  var instanceName = name,
      instanceIsNocturnal = nocturnal;

  return {
    getName: function() {
      return instanceName;
    },
    getIsNocturnal: function() {
      return instanceIsNocturnal;
    }
  };
};

AnimalKingdom.kangaroo = function(name) {
  var baseMarsupial = AnimalKingdom.marsupial(name, false);

  baseMarsupial.hop = function() {
    return baseMarsupial.getName() + '가 껑충 뛰었어요';
  };

  return baseMarsupial;
};

var jester = AnimalKingdom.kangaroo('제스터');
console.log(jester.getName());         // '제스터'
console.log(jester.getIsNocturnal());  // false
console.log(jester.hop());             // '제스터가 껑충 뛰었어요!'
```

AnimalKingdom.kangaroo 함수는 baseMarsupial 인스턴스를 확장해 hop 함수를 추가한다. 이 과정에서 AnimalKingdom.marsupial은 전혀 달라진 게 없다. 개방/폐쇄 원칙이 충실히 반영된 결과다.

| 원칙 | 결과 |
|-|-|
|단일 책임|함수형 상속은 모듈 패턴을 사용하므로 의존성 주입과 애스팩트 장식에 친화적이다. 상속한 모듈에는 반드시 한 가지 책임만 부여해야 한다.|
|개방/폐쇄|함수형 상속은 모듈 확장에 관한 한 완벽한 메커니즘이다.(따라서 모듈을 수정하지 않아도 된다.)|
|리스코프 치환|함수형 상속은 수정 없이 모듈을 확장할 수 있게 해주므로 상속받은 모듈은 자신이 상속한 모듈로 대체될 수 있다.|
|인터페이스 분리|다시 말하지만, 함수형 상속은 모듈 패턴의 변형이다. 응집된 모듈 API 자체가 분리된 인터페이스다.|
|의존성 역전|임의 모듈 생성 방식으로 만든 모듈을 상속에 사용했다면 의존성은 쉽게 주입할 수 있다.|
|DRY|설계만 잘한다면 모듈을 이용한 함수형 상속은 DRY한 코드로 향하는 이상적인 지름길이다.|

## 3-8 멍키 패칭

멍키 패칭은 추가 프로퍼티를 객체에 붙이는 것이다. 다른 객체의 함수를 붙여 객체의 덩치를 불리은 일은 자바스크립트 언어가 제격이다.

```js
var MyApp = MyApp || {};

MyApp.Hand = function() {
  this.dataAboutHand = {}; // etc.
};
MyApp.Hand.prototype.arrangeAndMove = function(sign) {
  this.dataAboutHand = '새로운 수화 동작';
};

MyApp.Human = function(handFactory) {
  this.hands = [handFactory(), handFactory()];
};
MyApp.Human.prototype.useSignLanguage = function(message) {
  var sign = {};
  // 메시지를 sign에 인코딩한다.
  this.hands.forEach(function(hand) {
    hand.arrangeAndMove(sign);
  });
  return '손을 움직여 수화하고 있어, 무슨 말인지 알겠니?';
};

MyApp.Gorilla = function(handFactory) {
  this.hands = [handFactory(), handFactory()];
};

MyApp.TeachSignLanguageTokoko = (function() {
  var handFactory = function() {
    return new MyApp.Hand();
  }
  // (빈자의 의존성 주입)
  var trainer = new MyApp.Human(handFactory);
  var koko = new MyApp.Gorilla(handFactory);

  koko.useSignLanguage = trainer.useSignLanguage;

  // 실행 결과: '손을 움직여 수화하고 있어. 무슨 말인지 알겠니?';
  console.log(koko.useSignLanguage('Hello!'));
}());
```

다음 줄 끝에서 멍키 패칭이 일어난다.

```js
kokouseSignLanguage = trainer.useSignLanguage;
```

조련사(trainer)의 수화(sign language) 능력을 코코(Koko)에게 패치한다. 코코에게 손(hands)이 있기에 가능한 일이다.(Human.useSignLanguage 함수의 this에 있어야 할, 즉 useSignLanguage 앞에 점(.)을 붙여 연결할 객체의 한 부분인) useSignLanguage는 Human에서 비롯되었지만, 이 함수 앞에 점을 붙여 호출하면(koko.useSignLanguage) 사람이 아닌 코코의 손을 움직인다.

1. koko.useSignLanguage('Hello!')를 호출한다.
2. 멍키 패칭을 했으니 MyApp.Human.prototype.useSignLanguage가 실행된다.
3. 이 함수는 this.hands에 접근한다.
4. 여기서 this는 useSignLanguage를 호출한 객체, 즉 MyApp.Gorilla 객체(koko)다. 따라서 MyApp.Gorilla 객체도 비로수 수화할 수 있는 손을 가지게 된다.

빌린 함수에 다른 요건이 추가될 가능성은 항상 있다. 따라서 패치를 관장하는 **빌려 주는 객체**가 빌리는 객체가 요건을 충족하는지 알아보게 하는 것이 가장 좋은 멍키 패칭 방법이다.

```js
MyApp.Humand.prororype.endowSigning = function(receivingObject) {
  if (typeof receivingObject.getHandCount === 'function'
  && receivingObject.getHandCount() >= 2) {
    receivingObject.useSignLanguage = this.useSignLanguage;
  } else {
    throw new Error("미안하지만 너에게 수화를 가르쳐줄 수는 없겠어.");
  }
};
```

물론 빌리는 객체는 빌려주는 객체의 질문에 반드시 대답해야 한다.

```js
MyApp.Gorilla.prototype.getHandCount = function() {
  return this.hands.length;
}

// 사람이 고릴라에게 수화 능력을 줌
trainer.endowSigning(koko);
```

이런 식으로 한 객체의 기능 전체를 다른 객체로 패치할 수도 있다. 인터페이스뿐만 아니라 코드도 함께 구현한 터라 사실 다중 상속에 더 가깝다.

멍키 패칭은 '메서드 빌림(method borrowing)'이라는 타이틀로 19장에서 더 자세히 설명한다.

| 원칙 | 결과 |
|-|-|
|단일 책임|기증받은 기능 다발이 단일 책임으로 이루어진다 해도 빌림 자체로 빌리는 객체에 책임을 전가하는게 아닌가 하는 점에서 놀란의 여지는 있을 수 있다. 하지만 그런식으로 생각하면 애스팩트 역시 책임을 더하는 건 마찬가지라서 어불성설이다.|
|개방/폐쇄|분별 있게 잘 쓰면 어길 일은 없다.|
|리스코프 치환|빌린 함수가 새 집과 옛집에서 그 의미가 같다면 문제없다.|
|인터페이스 분리|멍키 패칭이 추구하는 그 자체.|
|의존성 역전|의존성은 보통 빌려주는 객체 또는 빌리는 객체 어느 쪽에도 주입될 수 있다.|
|DRY|DRY한 콛를 유지하는데 도움이 많이 될 것이다.|

## 3-9 정리하기

원시형과 객체 리터럴은 사용하기 쉽지만, 코드 중복이 일어나기 쉽다.

모듈 패턴은 이에 관한 확실한 개선책이다. 데이터 캡슐화와 애스팩트 지향 프로그래밍을 동원하여 확장과 단위 테스트를 매끄럽게 한다.

객체 생성 패턴을 쓰지 말라는 사람도 있다. 그러나 생성자 함수로 초기화 코드를 공유할 수 있고 new를 강제해야 한다면 어렵지 않게 구현할 수 있기 때문에 퇴출 의견에 반대한다.

모든 자바스크립트 함수는 protorype 프로퍼티를 통해 객체 인스턴스 간에 코드와 데이터를 효과적으로 공유할 수 있다. 고전적 상속은 프로토타입 상속으로 흉내를 낼 수 있다. 또한, 프로토타입 상속 과정에서 야기되는 코드 반복을 없애고 데이터를 감출 수 있는 함수형 상속을 지원한다.

멍키 패칭을 잘 활용하면 한 객체의 기능을 다른 객체로 기증할 수 있다.
