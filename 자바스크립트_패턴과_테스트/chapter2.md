# 2장 도구 다루기

[2-1 테스팅 프레임 워크](#2-1-테스팅-프레임-워크)

## 2-1 테스팅 프레임 워크

예를 들어 항공 예약 데이터 생성 모듈을 맡게 되었는데, 그 중에는 다음과 같은 모듈 함수가 있었다.

'passenger 객체, flight 객체를 입력 받은 createReservation은 passengerInformation 프로퍼티가 passenger 객체, flightInformation 프로퍼티가 flight 객체인 새로운 객체를 반환한다.' 

간단하니 함수를 바로 구현한다.

```js
function createReservation(passenger, flight) {
  return {
    passengerInfo: passenger,
    flightInfo: flight
  }
}
```

단위 테스트를 작성한 후 성공까지 확인하고 다음 개발을 진행했다. 그러나 통합 작업을 하는 다른 팀에서 createReservation 함수 테스트가 모두 실패했다는 연락을 받는다.

자세히 보니 문제를 발견했다. 개발을 서두르다 보니 속성명을 명세와 다르게 코딩을 했다.

명세가 아니라 함수 코드의 개발에 따라 테스트를 작성한 탓에 테스트는 기대하는 함수 작동이 아닌, 구현된 함수의 (잘못된) 실제 작동을 확인할 수 있다.

> TIP : 단위 테스트가 없는 기존 코드를 작업할 땐 실제 기능을 확인하는 테스트를 작성해야 한다. 그래야 밖으로 표출되는 기능을 변경하지 않은 상태에서 코드를 리팩토링할 수 있다.

### 2-1-1 잘못된 코드 발견하기

TDD는 코드 결함을 최대한 빨리, 코드 생성 직후 감지하며, 작은 기능 하나라도 테스트를 먼저 작성한 뒤, 최소한의 코드만으로 기능을 구현한다.

createReservation 함수를 구현한 코드에 반환 객체의 속성명을 잘못 쓴 실수가 잠복해 있지만, 테스트를 먼저 작성한 뒤 명세에 따라 테스트를 했으므로 에러를 즉시 확인하여 조치할 수 있다.

사소한 함수마다 일일이 테스트를 만들어 기능을 붙이라는 건 너무하지 않나 여기는 독자도 있을 것이다. 하지만 TDD의 반복 과정이 디버깅 시간을 엄청나게 단축한 사례는 절대 드물지 않다.

### 2-1-2 테스트성을 감안하여 설계하기

테스트를 먼저 작성하란 건 코드의 테스트성(testability)을 차후에 두고 볼 문제가 아니라 우선적인 주요 관심사로 생각하는 것이다. 1장에서 일렀듯이 SOLID 개발 원칙을 준수하면 코드를 테스트할 때 도움이 많이 된다. 결국 테스트성을 설계 목표로 정하면 SOLID한 코드를 작성할 수 있다.

예를 들어 createReservation 함수로 생성한 모든 예약 데이터는 웹 서비스를 거쳐 데이터베이스에 저장해야 한다고 하자.

TDD에 충실한 사라이면 무조건 createReservation을 고치기보다는 일단 새 기능을 확인하는 테스트를 작성한다. 첫 번째 테스트에서는 예약 데이터가 제대로 웹 서비스 종단점까지 보내졌는지를 확인한다.

그런데 여기서 "createReservation이 웹 서비스 통신까지 맡아야 하나?" 라는 의문을 품는 건 자기 계발에 상당히 도움이 된다.

정답은 '그럴 일 없다.'이다. 웹 서비스 통신 전담 객체가 없으면 하나 만드는(그리고 테스트하는) 편이 좋겠다.

코드 테스트성을 극대화하면 SOLID 원칙을 어긴 코드를 쉽게 솎아낼 수 있다.

### 2-1-3 꼭 필요한 코드만 작성하기

TDD 작업 절차를 정리해보자.

작은 기능 하나를 검증하려면 실패하는 테스트를 먼저 작성한 뒤, 테스트를 성공시킬 만큼만 최소한으로 코딩한다. 그 후 내부적으로 구현 세부를 변경하는 **리팩토링** 과정을 거쳐 개발 중인 코드에서 중복 코드를 들어낸다.

결국 마지막에는 꼭 필요한 코드만 살아남게 된다.

### 2-1-4 안전한 유지보수와 리팩토링

TDD를 실천하는 프로젝트 제품 코드를 대상으로 확실한 단위 테스트 꾸러미를 구축할 수 있다.

예전에 잘 돌아가던 코드가 지금은 제대로 작동하지 않는 회귀 결함은 코드 품질과 믿음성을 떨어뜨리는 요인이다.

종합적인 단위 테스트 꾸러미가 마련된 제품 코드를 확장 또는 보수할 때 안도감을 느낄 수 있다. 실수로 다른 코드를 건드맂 않았다는 확신을 하고 코드 일부를 변경할 수 있기 때문이다.

### 2-1-5 실행 가능한 명세

TDD 실천 결과, 탄탄하게 구축된 단위 테스트 꾸러미는 테스트 대상 코드의 실행 가능한 명세(runnable specification) 역할도 한다. 단위 테스팅 프레임워크인 재스민은 행위 기반(behavior-based)으로 테스트를 구성한다. 재스민에서 **스펙**이라 부르는 개별 테스트는, 테스트하여 검증할 작동 로직을 일상 문장으로 표현하면서 시작한다. 각 테스트를 실행할 때마다 이 문장들은 재스민의 테스트 결과 기본 포매터로 화면에 표시된다.

재스민으로 단위 테스트한 결과 메시지를 보고 이 함수가 무슨 일을 하는지 큰 그림을 그려볼 수 있다. 굳이 코드를 읽고 분석하지 않아도 단위 테스트가 죄다 알려주는 셈이다!

### 2-1-6 최신 오픈 소스 및 상용 프레임워크

재스민 말고 다른 테스트 프레임워크도 있다.

#### QUnit

QUnit은 오픈 소스 프레임워크로 제이쿼리, 제이쿼리 UI, 제이쿼리 모바일 자바스크립트 프레임워크 개발자가 만들었다. QUnit은 노드JS나 리노 자바스크립트 엔진 같이 브라우저가 아닌 환경에서 실행할 수 있다. 또한, 브라우저에서 필요한 자바스크립트/CSS 파일을 HTML 파일 안에 포함해 사용하는 방법도 있다.

#### D.O.H.

D.O.H.(Dojo Objective Harness)는 도조(Dojo) 자바스크립트 프레임워크 개발자/운영자를 위해 개발 됐지만, D.O.H. 자체는 도조와 의존 관계가 없어서 범용 자바스크립트 테스팅 프레임워크로도 사용될 수 있다.

재스민, QUnit처럼 D.O.H.도 브라우저 기반 테스트와 브라우저 아닌 환경의 테스트(노드JS 또는 리노 자바스크립트 엔진 안에서) 모두 지원한다.

### 2-1-7 재스민 들어가기

재스민은 행위 주도 개발(Behavior-Driven Development, BDD) 방식으로 자바스크립트 단위 테스트를 작성하기 위한 라이브러리다.

> NOTE : BDD는 단위 테스트로 확인할 기능 또는 작동 로직을 일상 언어로 서술하는데, 이로써 개발자는 자신이 작성 중인 코드가 '어떻게'가 아니라 '무엇'을 해야 하는지 테스트 코드에 표현할 수 있다. 그리고 행위 주도 스타일로 정의/구성한 테스트는 쉬운 문장으로 서술한 기능 명세서로 삼을 만하다는 이점도 있다.

#### 테스트 꾸러미와 스펙

재스민 테스트 꾸러미는 전역 함수 describe로 정의되며, 이 함수는 두 인자를 받는다.

* 문자열 : 무엇을 테스트할지 서술한다.
* 함수 : 테스트 꾸러미의 구현부(implementation)다.

테스트 꾸러미는 스펙, 즉 개별 테스트로 구현되며, 각 스펙은 전역 함수 it으로 정의된다. it 함수도 desribe 함수처럼 인자를 2개 받는다.

* 문자열 : 무엇을 테스트할지 서술한다.
* 적어도 한 개의 기대식(expectation)을 가진 함수 : 코드 상태의 true/false를 확인하는 단언

테스트 꾸러미 구현부에 beforeEach/afterEach를 쓰면 각 꾸러미 테스트가 실행되기 이전에 beforeEach 함수를, 그 이후에는 afterEach 함수를 호출한다. 전체 테스트가 공유할 설정(setup)과 정리(teardown) 코드를 두 함수에 담아두면 코드 중복을 피할 수 있어 좋다.

> NOTE : 테스트 꾸러미 역시 SOLID하고 DRY해야 한다!

#### 기대식과 매처

expect 문은 테스트마다 있다. 다음은 첫 번째 단위 테스트 createReservation의 expect 문이다.

```js
expect(testReservation.passengerInformation).toBe(testPassenger);
```

expect 함수는 대상 코드가 낸 실제값(여기서는 testReservation.passengerInformation)을 인자로 받아 기댓값과 견주어본다. 이 테스트가 기대하는 값은 testPassenger다.

실제값과 기댓값을 비교하는 일은 매처(matcher) 함수의 몫이다. 매처는 비교 결과 성공하면 true를, 실패하면 false를 반환한다. 하나 이상의 기대식이 포함된 스펙에서 매처가 하나라도 실패하면 모조리 실패한거로 간주한다. 반대로, 모든 매처가 성공하면 스펙은 성공한다.

toBe 매처는 이름에서 짐작할 수 있듯이 testReservation.passengerInformation이 testPassenger와 같은 객체여야(to be)한다는 의미다.

재스민이 제공하는 내장 매처는 여럿 있는데, 용도에 맞는 매처가 없으면 재스민이 지원하는 커스텀 매처를 만들어 쓴다. 커스텀 매처를 잘 활용하면 훌륭하고 DRY한 테스트 코드를 작성할 수 있다.

#### 스파이

재스민 스파이(spy)는 **테스트 더블**(test double) 역할을 하는 자바스크립트 함수다. 테스트 더블은 어떤 함수/객체의 본래 구현부를 테스트 도중 다른 (보통은 더 간단한) 코드로 대체한 것을 말하며, 웹 서비스 같은 외부 자원과의 의존 관계를 없애고 단위 테스트의 복잡도를 낮출 목적으로 사용된다.

다음 다섯 가지를 통칭하여 테스트 더블이라고 한다.

* 더미(dummy) : 보통 인자 리스트를 채우기 위해 사용되며, 전달은 하지만 실제로 사용되지는 않는다.
* 틀(stub) : 더미를 조금 더 구현하여 아직 개발되지 않은 클래스나 메서드가 실제 작동하는 것처럼 보이게 만든 객체로 보통 리턴값은 하드 코딩한다.
* 스파이(spy) : 틀과 비슷하지만 내부적으로 기록을 남긴다는 점이 다르다. 특정 객체가 사용되었는지, 예상되는 메서드가 특정한 인자로 호출되었는지 등의 상황을 감시(spying)하고 이러한 정보를 제공하기도 한다.
* 모의체(fake) : 틀에서 조금 더 발전하여 실제로 간단히 구현된 코드를 갖고는 있지만, 운영 환경에서 사용할 수는 없는 객체다.
* 모형(mock) : 더미, 틀, 스파이를 혼합한 형태와 비슷하나 행위를 검증하는 용도로 주로 사용된다. 

생성된 예약 데이터를 데이터베이스에 저장하는 웹 서비스와 직접 통신할 수 있게 createReservation 함수를 확장하는 문제를 앞서 간략히 살펴봤다.

ReservationSaver라는 자바스크립트 객체를 만들어 이 객체의 saveReservation 함수로 웹 서비스에 예약 데이터를 전송하는 기능을 캡슐화 했다. createResevation 함수를 확장하여 이 함수가 ReservationSaver 인스턴스를 인자로 받아 이 인스턴스의 saveReservation 함수를 실행하는지 확인하고자 한다.

saveReservation 함수는 웹서비스와 통신하므로 지금부터 작성할 테스트는 예약 데이터 저장 후 DB를 질의하고 예약 데이터가 분명히 추가됐는지 확인하는 과정이 모두 들어가야 할 듯싶다. 하지만 그럴 필요도 없고 그래서는 안 된다. 자칫 단위 테스트가 웹 서비스, DB 같은 외부 시스템 유무와 작동 여부에 의존하게 될지도 모른다.

>> TIP : 외부 시스템과 연동하는 코드에 이상이 없는지 확인하는 테스트를 통합 테스트(integration test)라 한다. 소프트웨어를 바르게 작성하는 데 중요한 과정이지만, 단위 테스트와는 분명히 구별해야 한다. 여기서는 ResevationSaver 객체에 해당하는 통합 테스트를 잘 마련해두었다고 가정한다.

재스민 스파이를 사용하면 복잡한 saveReservation 구현부를 외부 시스템 의존성을 배제한, 단순한 형태로 바꿀 수 있다. 먼저 ReservationSaver 객체를 보자.

```js
function ReservationSaver() {
  this.saveReservation = function(reservation) {
    // 예약 정보를 젖아하는 웹 서비스와 연동하는 복잡한 코드가 있을 것이다.
  }
}
```

createReservation 함수는 ReservationSaver 인스턴스를 전달받게끔 개선됐다. ReservationSaver를 인자로 받으므로 예약 데이터가 저장되었는지 확인사는 테스트를 다음과 같이 작성할 수 있다.

```js
describe('createReservation', function() {
  it('예약 정보를 저장한다.', function() {
    var saver = new ReservationSaver();
    // testPassenger와 testFlight는 테스트 꾸러미의 beforeEach로 이미 설정 됐다고 가정하자
    createReservation(testPassenger, testFlight, saver);

    // saver.saveReservation(...) 이 정말 호출되는지 어떻게 알 수 있을까?
  });
});
```

이 테스트 코드에 씌어있는 대로 복잡한 ReservationSaver의 기본 구현부를 createReservation 함수에 전달하고 있다. 이렇게 하면 결국 외부 시스템에 의존하게 되고 함수를 테스트하기가 어려워지므로 별로 내키지 않는다. 바로 이럴 때 재스민 스파이가 제격이다.

createReservation을 호출하기 전에 saveReservation 함수에 스파이를 심는다. 스파이로 함수 실행 여부를 알 수 있다.

재스민에서 전역 함수 spyOn을 쓰면 특정 함수를 몰래 들여다볼 수 있다. 이 함수의 첫 번째 인자는 객체 인스턴스, 두 번째 인자는 감시할 함수명이다. 다음과 같이 스파이를 심은 코드로 바꿔보자.

```js
describe('createReservation', function() {
  it('예약 정보를 저장한다.', function() {
    var saver = new ReservationSaver();
    spyOn(saver, 'saveReservation');
    // testPassenger와 testFlight는 테스트 꾸러미의 beforeEach로 이미 설정 됐다고 가정하자
    createReservation(testPassenger, testFlight, saver);

    // saver.saveReservation(...) 이 정말 호출되는지 어떻게 알 수 있을까?
  });
});
```

스파이를 써서 saver 객체의 saveReservation 구현부를 예약 데이터 저장 기능과 무관한 함수로 대체했다. 스파이는 함수를 호출한 시점과 호출 시 전달한 인자까지 정확히 포착하고, 무엇보다 재스민은 어떤 스파이가 한 번 이상 실행됐는지 확인하는 기대식을 지닌 스파이 전용 매처를 지원한다. 다음과 같이 기대식까지 추가하면 완벽한 테스트다.

```js
describe('createReservation', function() {
  it('예약 정보를 저장한다.', function() {
    var saver = new ReservationSaver();
    spyOn(saver, 'saveReservation');
    // testPassenger와 testFlight는 테스트 꾸러미의 beforeEach로 이미 설정 됐다고 가정하자
    createReservation(testPassenger, testFlight, saver);

    expect(saver.saveReservation).toHaveBeenCalled();
  });
});
```

createReservation 함수의 인자가 늘었으니 기존 두 테스트 역시 수정할 수밖에 없다. 하지만 saveReservation 함수 구현부를 직접 실행할 테스트는 없을 테니 ReservationSaver 생성 코드와 스파이 관련 코드를 전체 꾸러미의 beforeEach 함수로 옮겨 리팩토링한다.

```js
describe('createReservation(passenger, flight, saver)', function() {
  var testPassenger = null,
    testFlight = null,
    testReservation = null,
    testSaver = null;

  beforeEach(function() {
    testPassenger = {
      firstName: '윤지',
      lastName: '김'
    };

    testFlight = {
      number: '3443',
      carrier: '대한항공',
      destination: '울산'
    };

    testSaver = new ReservationSaver();
    spyOn(testSaver, 'saveReservation');

    testReservation = createReservation(testPassenger, testFlight, testSaver);
  });

  it('passenger를 passengerInformantion 프로퍼티에 할당한다', function() {
    expect(testReservation.passengerInformation).toBe(testPassenger);
  });

  it('주어진 flight를 flightInformation 프로퍼티에 할당한다', function() {
    expect(testReservation.flightInformation).toBe(testFlight);
  });

  it('예약 정보를 저장한다', function() {
    expect(testSaver.saveReservation).toHaveBeenCalled();
  });
});
```

createReservation 함수도 수정한다.

```js
function createReservation(passenger, flight, saver) {
   var reservation = {
    passengerInformation: passenger,
    flightInformation: flight
  };

  saver.saveReservation(reservation);
  return reservation;
}

function ReservationSaver() {
  this.saveReservation = function(reservation) {
    // 예약 정보를 저장하는 웹서비스와 연동하는 복잡한 코드가 있을 것이다.
  };
}
```
