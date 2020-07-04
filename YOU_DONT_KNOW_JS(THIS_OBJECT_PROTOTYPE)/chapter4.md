# 클래스와 객체의 혼합

클래스 지향 개념은 자바스크립트 객체 체계와는 태생부터 잘 맞지 않아 지금껏 수많은 개발자가 이런 한계를 극복하고 기능을 확장(믹스인 등)하고자 노력해 왔다.

- [4-1 클래스 이론](#4-1-클래스-이론)
- [4-2 클래스 체계](#4-2-클래스-체계)
- [4-3 클래스 상속](#4-3-클래스-상속)
- [4-4 믹스인](#4-4-믹스인)
- [4-5 정리하기](#4-5-정리하기)

## 4-1 클래스 이론

클래스와 상속은 특정 형태의 코드와 구조를 형성하며 실생활 영역의 문제를 소프트웨어로 모델링 하기 위한 방법이다.

객체 지향 또는 클래스 지향 프로그래밍에서 데이터는 자신을 기반으로 하는 실행되는 작동(Behavior)과 연관되므로 데이터와 작동을 함께 잘 감싸는(캡슐화) 것이 올바른 설계라고 강조한다. 이름 자료 구조(Data Structure)라고도 표현 한다.

클래스는 특정 자료 구조를 분류(Classify)하는 용도로 쓴다. 즉, 일반적인 기준 정의에서 세부적이고 구체적인 변형으로서의 자료 구조를 도출하는 것이다. 흔한 예로, 차(car)는 이란적인 탈것(vehicle) '클래스'의 구현체 중 하나다.

소프트웨어에서는 이런 관계를 Vehicle 클래스, Car 클래스로 각각 모델링 한다. Vehicle은 추진 기관 같은 것(Thing)과 사람을 운송하는 기능, 즉 작동(Behavior)을 함께 정의한다. 따라서 Vehicle의 범주에는 거의 모든 유형의 탈 것(자동차, 비행기, 기차 등)이 포함된다.

다른 탈것마다 '사람을 운송하는 기능'을 매번 재정의해야 한다면 올바른 소프트웨어 설계가 아닐 것이다. 그래서 Vehicle 한 곳에만 정의해두고 Car는 Vehicle 한 곳에만 정의해두고 Car는 Vehicle에 있는 기반 정의(Base Definition)를 상속(Inherit) or 확장(Extend) 받아 정의한다. Car가 일반적이 Vehicle의 정의를 세분화한 셈이다.

Vehicle과 Car에서 메서드르 통해 집학적으로 작동을 정ㅇ의한다면 인스턴스 데이터는 차량별 고유 시리얼 넘버 같은 것을 생각할 수 있다. 이런 식으로 클래스, 상속, 인스턴스화가 이루어진다.

다형성(Polymorphism)은 부모 클래스에 뭉뚱그려 정의된 작동을 자식 클래스에서 좀 더 구체화하여 오버라이드(재정의)하는 것을 뜻한다.

### 4-1-1 클래스 디자인 패턴

클래스 역시 디자인 패턴의 일종이다.

함수형 프로그래밍을 경험해 봤다면 클래스가 단지 많이 쓰는 서너 개 디자인 패턴 중 하나라는 사실 또한 잘 알고 있을 것이다.

자바 같은 언어는 만물이 다 클래스다. C/C++ 같은 언어는 절차적 구문과 클래스 지향 구문을 함께 제공하므로 개발자가 선택 할 수 있다.

### 4-1-2 자바스크립트 클래스

클래스와 비슷하게 생긴 new, instanceof 등의 구문이 있고 ES6 부터는 class라는 키워드가 추가 됐다.

그럼 자바스크립트는 클래스가 있을까?

**No**

클래스는 디자인 패턴이므로 공을 들이면 고전적인 클래스 기능과 얼추 비슷하게 구현할 수 있다. 그러나 클래스처럼 보이는 구문일 뿐이며 개발자들이 클래스 디자인 패턴으로 코딩할 수 있도록 자바스크립트 체계를 억지로 고친 것에 불과하다.

결론은, 클래스는 디자인 패턴 중 한 가지 옵션일 뿐이니 자바스크립트에서 클래스를 쓸지 말지는 결국 자신이 결정할 문제다.

## 4-2 클래스 체계

대부분 클래스 지향 언어의 '표준 라이브러리'는 '스택 자료 구조를 Stack 클래스에 구현해놨다.

하지만 Stack 클래스에서 실제로 어떤 작업을 직접 수행하는 건 아니다. Stck 클래스는 그저 모든 스택이라면 마땅히 해야할 기능을 추상화한 것이지 그 자체가 스택은 아니다. Stack 클래스를 인스턴스화해야 비로소 작업을 수행할 구체적인 자료 구조가 마련된다.

### 4-2-1 건축

클래스와 인스턴스 중심의 사고방식은 건축 현장에 빗대어 생각할 수 있다.

아키텍처 청사진은 건축을 위한 계획이다. 건축의 모든 특성(폭, 높이 창문 위치와 소요량 등)을 기획한다. 건물 안 내용물을 신경 쓰지 않고 이러한 부품들을 배치할 전체적인 구조를 계획한다.

그리고 시공사에 의뢰해서 청사진에 따라 건물을 짓는다. 사실 청사진을 작성한 아키텍트가 의도했던 특성 그대로 물리적인 건물을 복사하는 셈이다. 완공된 건물은 청사진의 물리적인 인스턴스다. 시공사는 이제 다음 부지로 옮겨 같은 건물을 처음부터 다시 짓는다.

클래스가 바로 청사진에 해당한다. 개발자가 상호 작용할 실제 객체는 클래스라는 붕어빵 틀에서 구어낸다.(인스턴스화) 이 '구워냄'의 최종 결과가 인스턴스라는 객체고 개발자는 객체 메서드를 직접 호출하거나 공용 데이터 프로퍼티에 접근한다. 객체는 클래스에 기술된 모든 특성을 그대로 가진 사본이다.

클래스는 복사 과정을 거쳐 객체 형태로 인스턴스화한다.

### 4-2-2 생성자

인스턴스는 보통 클래스명과 같은 이름의 생성자라는 특별한 메서드로 생성한다. 생성자의 임무는 인스턴스에 필요한 정보를 초기화하는 일이다.

생성자는 클래스에 속한 메서드로, 클래스명과 같게 명명하는 것이 일반적이다. 그리고 새로운 인스턴스를 생성할 거라는 신호를 엔진이 인지할 수 있도록 항상 new 키워드를 앞에 붙여 생성자를 호출한다.

## 4-3 클래스 상속

클래스 지향 언어에서는 자체로 인스턴스화할 수 있는 클래스는 물론이고 첫 번째 클래스를 상속받은 두 번째 클래스를 정의할 수 있다. '부모 클래스', '자식 클래스'라고 통칭한다.

자식 클래스는 부모 클래스에서 완전히 떨어진 별개의 클래스로 정의된다. 부모로부터 복사된 초기 버전의 작동을 고스란히 간직하고 있지만 물려받은 작동을 전혀 새로운 방식으로 오버라이드할 수 있다.

다음은 상속받은 클래스를 의사 코드로 대략 작성한 예제다.

```
class Vehicle {
  engines = 1

  ignition() {
    output("엔진을 켠다.")
  }
  drive() {
    ignition()
    output("방향을 맞추고 앞으로 간다!")
  }
}

class Car inherits Vehicle {
  wheels = 4

  drive() {
    inherited::drive()
    output(wheels, "개의 바퀴로 굴러간다!")
  }
}

class SpeedBoat inherits Vehicle {
  engines = 2

  ignition() {
    output(engines, "개의 엔진을 켠다.")
  }
  pilot() {
    inherited:drive()
    output("물살을 가르며 쾌속으로 질주한다!")
  }
}
```

Vehicle 클래스는 탈것에 불과한 추상적인 개념에 지나지 않는다.

그래서 구체적인 탈 것 차(car), 모터보트(Speedboad)를 정의한다. 둘 다 Vehicle의 일반적인 특성을 물려받아 각자에게 맞는 특성을 세분화한다.

### 4-3-1 다형성

Car는 Vehicle로부터 상속받은 drive() 메서드를 같은 명칭의 자체 메서드로 오버라이드한다. 그러나 이 메서드 안에서 오버라이드 하기 전의 원본을 참조하며 SpeedBoat의 pilot() 메서드 역시 상속받은 원본 drive()를 참조한다.

이런 기법을 다형성(Polymorphism) 또는 가상 다형성(Virtual Polymorphism)이라고 한다. 좀 더 구체적으로는 상대적 다형성(Relative Polymorphism)이다. '상대적'이란 관점에서 한 메서드가 상위 수준의 상속 체계에서 (메서드명은 같거나 다를 수 있지만) 다른 메서드를 참조할 수 있게 해주는 아이디어다.

대부분 언어는 supter하는 키워드를 사용하며, 이는 'superclass'를 현재 클래스의 부모/조상이라고 간주하는 것이다.

같은 이름의 메서드가 상속 연쇄의 수준별로 다르게 구현되어 있고 이 중 어떤 메서드가 적절한 호출 대상인지 자동으로 선택하는 것 또한 다형성의 특징이다.

다형성의 흥미로운 단면은 ignition() 메서드에 구체적으로 나타나 있다.

pilot() 안에 Vehicle에 상속된 drive()를 참조한다. 정작 drive()는 그냥 메서드 이름만 보고 ignition() 메서드를 참조한다.

그럼 자바스크립트 엔진은 Vehicle과 SpeedBoad 중 어느 쪽 ignition()을 실행할까? 정답은 SpeedBoad의 ignition() 이다.

Vehicle 클래스를 인스턴스화하여 drive()를 호출했다면 당연히 엔진은 Vehicle의 ignition()을 실행할 것이다.

즉, 인스턴스가 어느 클래스(상속 수준)를 참조하느냐에 따라 ignition() 메서드의 정의는 다형적이게 된다.(모습이 변한다)

클래스를 상속하면 (클래스로부터 생성된 인스턴스가 아닌) 자식 클래스에는 자신의 부모 클래스를 가리키느 ㄴ상대적 레퍼런스가 주어진다. 이를 보통 super라고 한다.

자식이 부모에게 상속받은 메서드를 '오버라이드'하면 원본 메서드와 오버라이드된 메서드는 각자의 길을 걷게 된다. 그래서 양쪽 다 개별적으로 접근할 수 있다.

자식 클래스가 마치 부모 클래스에 연결된 양 다형성을 혼동하지 않길 바란다. 자식은 그저 부모에게서 자신의 필요한 내용을 베껴왔을 뿐. 클래스 상속은 한 마디로 '복사'다.

### 4-3-2 다중 상속

일부 클래스 지향 언어에서는 복수의 '부모' 클래스에서 '상속' 받을 수 있다. 다중 상속은 부모 클래스 각각의 정의가 자식 클래스로 복사된다는 의미다.

좋아보이지만 복잡한 문제들이 잠재되어 있다. 예를 들어 마름모 문제(Diamond Problem)란 것도 있다.

자바스크립트는 '다중 상속'을 지원하지 않는다.

## 4-4 믹스인

자바스크립트엔 '클래스'란 개념 자체가 없고 오직 객체만 있다. 그리고 객체는 다른 객체에 복사되는 게 아니라 서로 연결된다.(자세한 내용은 5장 프로토타입)

믹스인(Mixin)은 클래스 복사 기능을 흉내 낸 것으로, 명시적 믹스인과 암시적 믹스인 두 타입이 있음.

### 4-4-1 명시적 믹스인

Vehicle/Car 예제를 다시 보자. 자바스크립트 엔진은 Vehicle의 작동을 Car로 알아서 복사하지 않으므로 일일이 수동으로 복사하는 유틸리티를 대신 작성하면 된다.

```js
// mixin() 예제
function mixin(sourceObj, targetObj) {
  for(var key in sourceObj) {
    if(!key in targetObj)) {
      targetObj[key] = sourceObj[key];
    }
  }
  return targetObj;
}

var Vehicle = {
  engines: 1,

  ignition: function() {
    console.log("엔진을 켠다.");
  }

  drive: function() {
    this.ignition();
    console.log("방향을 맞추고 앞으로 간다!");
  }
};

var Car = mixin(Vehicle, {
  wheels: 4,
  drive: function() {
    Vehicle,drive.call(this);
    console.log(
      this.wheels + "개의 바퀴로 굴러간다!"
    );
  }
});
```

이제 Car에는 Vehicle에서 복사한 프로퍼티와 함수 사본이 있다.(엄밀히 말해 원본 함수를 가리키는 레퍼런스만 복사된 것)

### 다형성 재고

Vehicle.drive.call(this)와 같은 코드를 명시적 의사다형성(Explicit Pseudopolymorphism)이라 부른다.

자바스크립트는 (ES6 이전엔) 상대적 다형성을 제공하지 않는다. 따라서 drive()란 이름의 함수가 Vehicle과 Car 양쪽에 모두 있을 때 이 둘을 구별해서 호출하려면 (상대적이 아닌) 절대적인 레퍼런스를 이용할 수 박에 없다.

그리고 Car 객체와 바인딩을 하기 위해 Vehicle.drive.call(this)로 호출하게 함.

자바스크립트 언어는 (의사)다혀적 레퍼런스가 필요한 함수마다 명시적 의사다형성 방식의 취약한 연결을 명시적으로 일일이 만들어줄 수 밖에 없다.

결과 적으로 더 복자하고 관리하기 어려운 코드가 된다. 명시적 의사다형성은 가능한 쓰지 말자.

### 사본 혼합

사실 자바스크립트 함수는 (표준적인 방법으로 확실하게) 복사할 수 없다. 복사되는 건 같은 공휴 함수 객체(함수도 객체)를 가리키는 사본 레퍼런스다. (ignition() 같은) 공휴 함수 객체에 프로퍼티를 추가하는 등의 변경을 하면 공유 레퍼런스를 통해 Vehicle/Car 모두에게 영향을 끼친다.

명시적 믹스인은 쓸만한 장치이긴 하지만 강력하지는 않다. 차라리 그냥 무식하게 똑같은 프로퍼티를 각각 두 번씩 정의하는 편이 나을 수도 있다.

명시적 믹스인은 코드 가독성에 도움이 될 때만 조심하여 사용하되 점점 코드가 추적하기 어려워지거나 불필요한 객체 간 의존 관계가 양산될 때 사용을 중단하자.

### 기생 상속

'기생 상속(Pararsitic Inheritance)'은 더글러스 크록포드가 작성한 명시적 믹스인 패턴의 변형. 명시적/암시적 특징을 모두 갖고 있다.

```js
// "전통적인 자바스크립트 클래스" 'Vehicle'
function Vehicle() {
  this.engines = 1;
}
Vehicle.prototype.ignition = function () {
  console.log("엔진을 켠다.");
};
Vehicle.prototype.drive = function () {
  this.ignition();
  console.log("방향을 맞추고 앞으로 간다!");
};

// "기생 클래스" 'Car'
function Car() {
  // 자동차는 탈것의 하나다.
  var car = new Vehicle();

  // 자동차에만 해당되는 내용은 수정한다.
  car.whells = 4;

  // 'Vehicle::drive()'를 가리키는 내부 레퍼런스를 저장한다.
  var vehDrive = car.drive;

  // 'Vehicle::drive()'를 오버라이드한다.
  car.drive = function () {
    vehDrive.call(this);
    console.log(this.whells + "개의 바퀴로 굴러간다!");
  };
  return car;
}

var myCar = new Car();

myCar.drive();
// 엔진을 켠다.
// 방향을 맞추고 앞으로 간다!
// 4개의 바퀴로 굴러간다!
```

부모 클래스인 Vehicle(객체)의 정의를 복사한다. 그리고 자식 클래스(객체) 정의에 믹스인 한 뒤 조합된 객체 car를 자식 인스턴스로 넘긴다.

### 4-4-2 암시적 믹스인

암시적 믹스인은 명시적 의사다형성과 밀접한 관계가 있으므로 사용할 때 주의해야 한다.

```js
var Something = {
  cool: function () {
    this.greeting = "Hello World";
    this.count = this.count ? this.count + 1 : 1;
  },
};

Something.cool();
Something.greeting; // "Hello World"
Something.count;

var Another = {
  cool: function () {
    // 'Something'을 암시적으로 'Another'로 믹스인한다.
    Something.cool.call(this);
  },
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 ('Something'과 상태가 공유되지 않는다.)
```

this 재바인딩을 십분 활용한 이런 유형의 테크닉은 Something.cool.call(this) 같은 호출이 상대적 레퍼런스가 되지 않아 (그래서 더 유연하긴 함) 불안정하므로 신중히 처리해야 한다.

## 4-5 정리하기

클래스는 디자인 패턴의 일종이다. 자바스크립트에서 클래스의 의미는 다른 언어들과 다르다.

클래스는 복사를 의미한다. 전통적인 클래스는 인스턴스화하면 [클래스->인스턴스]로 복사가 일어난다. 클래스를 상속하면 역시 [부모->자식] 방향으로 복사된다. 다형성은 얼핏 보면 [자식->부모] 방향의 상대적 레퍼런스가 아닐까 싶지만 그냥 복사 작업의 결과물일 뿐이다.

자바스크립트는 객체 간 사본을 자동으로 생성하지 않는다. 믹스인 패턴(명시적/암시적)은 클래스의 복사 기능을 모방하기 위해 종종 쓰이지만 대부분 명시적 의사다형성(OtherObj.methodName.call(this, ...)) 처럼 보기 싫고 취약한 구문이 되어 유지 보수도 어려운 코드가 된다.

명시적 믹스인은 클래스의 복사 기능과 같지 않다. 이는 객체(그리고 함수!) 그 자체가 아니라 단지 공유된 레퍼런스만 복사하기 때문이다. 이런 부분에 유의하지 않으면 고생할 수 있다.

일반적으로 자바스크립트에서 클래스를 모방하는 건 당장 닥친 문제를 해결할 수 있어도 앞으로 터질 시한폭탄을 심어놓는 것과 다름 없다.

```

```