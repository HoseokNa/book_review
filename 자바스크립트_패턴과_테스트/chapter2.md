# 2장 도구 다루기

[2-1 테스팅 프레임 워크](#2-1-테스팅-프레임-워크)
[2-2 의존성 주입 프레임워크](#2-2-의존성-주입-프레임워크)

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

## 2-2 의존성 주입 프레임워크

의존성 **역전**이 5대 SOLID 개발 요소 중 하나고, 의존성 **주입**은 이를 실현하기 위한 메커니즘이라고 했다. 이 절에서는 유연하면서도 엄격한 의존성 주입 프레임워크를 개발한다.

### 2-2-1 의존성 주입이란?

승현은 좌석 예약 기능을 갖춘 클라이언트 측 코드 개발을 맡았다.

우선 ConferenceWebSvc 객체에 서비스를 캡슐화하고 멋집 팝업 메시지를 화면에 표시할 자바스크립트 객체 Messenger를 작성한다.

참가자는 1인당 세션을 10개까지 등록할 수 있다. 참가자가 한 세션을 등록하면 그 결과를 성공/실패 메시지로 화면에 표시하는 함수를 개발해야 한다.

```js
Attendee = function(attendeeId) {

  // 'new'로 생성하도록 강제한다.
  if (!(this instanceof Attendee)) {
    return new Attendee(attendeeId);
  }

  this.attendeeId = attendeeId;

  this.service = new ConferenceWebSvc();
  this.messenger = new Messenger();
};

// 주어진 세션에 좌석을 예약 시도한다.
// 성공/실패 여부를 메시지로 알려준다.
Attendee.prototype.reserve = function(sessionId) {
  if (this.service.reserve(this.attendeeId, sessionId)) {
    this.messenger.success('좌석 예약이 완료되었습니다!' +
      ' 고객님은 ' + this.service.getRemainingReservations() +
      ' 좌석을 추가 예약하실 수 있습니다.');
  }  else {
    this.messenger.failure('죄송합니다, 해당 좌석은 예약하실 수 없습니다.');
  }
};
```

이 코드는 일견 ConferenceWebSvc, messenger, Attendee 객체가 각자 자신만의 임무를 갖고 아름다운 모듈로 조화를 이룬 것처럼 보인다. ConferenceWebSvc 내부에는 HTTP 호출이 있다. 이렇게 HTTP 통신이 필요한 코드는 단위 테스트를 어떻게 할까? 그리고 Messenger는 메시지마다 OK 버튼이 있어야 하는데, 이 또한 이 모듈에서 단위 테스트할 대상은 아니다. 모든 단위가 미처 준비도 되기 전에 시스템 테스트의 늪으로 빠지는 게 싫다.

요는, Attendee 객체가 아니라 이 객체가 의존하는 코드다. 의존성을 주입하는 식으로 바꾸면 해결할 수 있다. 실제 운영 환경에서는 진짜 의존성을 주입하겠지만, 단위 테스트용으로는 모의체(fake)나 재스민 스파이 같은 대체제를 주입하면 된다.

```js

// 운영 환경:
var attendee = new Attendee(new ConferenceWebSvc(), new Messenger(), id);

// 개발(테스트) 환경:
var attendee = new Attendee(fakeService, fakeMessenger, id);
```

이처럼 DI 프레임워크를 사용하지 않고 의존성을 주입하는 것을 두고 '빈자의 의존성 주입(poor man's dependency injection)'이라 한다. 다음은 빈자의 의존성 주입 방식으로 작성한 Attendee 객체다.

```js
Attendee = function(service, messenger, attendeeId) {
   // 'new'로 생성하도록 강제한다.
  if (!(this instanceof Attendee)) {
    return new Attendee(attendeeId);
  }

  this.attendeeId = attendeeId;

  this.service = service;
  this.messenger = messenger;
};
```

### 2-2-2 의존성을 주입하여 믿음직한 코드 만들기

아무래도 테스트를 통과한, 자도화한 테스트 꾸러미로 계속 테스트할 수 있는 코드가 더 믿음직하다.

이뿐만 아니라, DI는 실제 객체보다 주입한 스파이나 모의 객체에 더 많은 제어권을 안겨주므로 다양한 에러 조건과 기이한 상황을 만들어내기 쉽다. 혹시 모를 만약의 사태를 폭넓게 커버할 수 있게 테스트를 작성할 수 있다.

또한, DI는 코드 재사용을 적극적으로 유도한다. 의존성을 품은, 하드 코딩한 모듈은 무거운 짐을 질질 끌고 다니는 터라 보통 재사용하기 어렵다.

### 2-2-3 의존성 주입의 모든 것

어떤 객체를 코딩하든 어떤 객체를 생성하든지 스스로 다음 질문을 해보자. 한 가지라도 답변이 "예"라면 직접 인스턴스화(instantiation)하지 말고 주입하는 방향으로 생각을 전환하라.

* 객체 또는 의존성 중 어느 하나라도 DB, 설정 파일, HTTP, 기타 인프라 등의 외부 자원에 의존하는가?
* 객체 내부에서 발생할지 모를 에러를 테스트에서 고려해야 하나?
* 특정한 방향으로 객체를 작동시켜야 할 테스트가 있는가?
* 서드파티(third-party) 제공 객체가 아니라 온전히 내가 소유한 객체인가?

좋은 의존성 주입 프레임워크를 골라 써야 API랑 친해지기 쉽고 여러모로 도움이 된다. 다음 절을 참고하자.

### 2-2-4 사례 연구: 경량급 의존성 주입 프레임워크 개발

지금까지는 의존성 주입을 하드코딩했다. 전문가다운 의존성 주입 프레임워크는 이렇게 작동한다.

1. 애플리케이션이 시작되자마자 각 인젝터블(injectable) 명을 확인하고 해당 인젝터블이 지닌 의존성을 지칭하며 순서대로 DI 컨테이너에 등록한다.
2. 객체가 필요하면 컨테이너에 요청한다.
3. 컨테이너는 일단 요청받은 객체와 그 의존성을 모두 재귀적으로 인스턴스화한다. 그런 다음, 요건에 따라 필요한 객체에 각각 주입한다.

DI 컨테이너를 직접 만들면서 프레임워크의 작동 원리를 알아보자.

컨테이너는 인젝터블과 의존성을 등록하고, 요청 시 객체를 내어주는 두 가지 일을 한다.

register 함수의 인자는 세 가지다.

* 인젝터블 명
* 의존성 명을 담은 배열
* 인젝터블 객체를 반환하는 함수. 인젝터블 인스턴스를 요청하면 컨테이너는 이 함수를 호출하여 반환값을 다시 그대로 반환한다. 또한, 컨테이너는 요청받은 객체의 의존성 인스턴스 역시 이 함수에 전달하는데, 곧 이어지는 테스트에서 다시 설명한다.

TDD는 단계마다 가급적 조금씩 코딩하는 게 좋다. 빈 register를 먼저 생각해보자. 이 함수는 DIContainer의 인스턴스가 모두 공유하는 자산이므로 프로토타입에 둔다.

```js
DiContainer = function() {
};

DiContainer.prototype.register = function(name,dependencies,func) {
  // 처음 버전이라 하는 일이 없다.
};
```

좋은 코드를 짜려면 인자가 제대로 전달됐는지, 타입은 올바른지 확인해야 한다. 후속 테스트들이 기반을 둘 첫 번째 테스트를 잘 작성해야 확실한 토대를 마련할 수 있다.

```js
describe('DiContainer', function() {
  var container;
  beforeEach(function() {
    container = new DiContainer();
  });
  describe('register(name,dependencies,func)', function() {

    // 01
    it('인자가 하나라도 누락되었거나 타입이 잘못되면 예외를 던진다', function() {
      var badArgs = [
        // 인자가 아예 없는 경우
        [],
        // name만 있는 경우
        ['Name'],
        // name과 dependencies만 있는 경우
        ['Name',['Dependency1','Dependency2']],
        // dependencies가 누락된 경우
        // (상용 프레임워크는 지원하지만 DiContainer는 지원하지 않음)
        ['Name', function() {}],
        // 타입이 잘못된 다양한 사례들
        [1,['a','b'], function() {}],
        ['Name',[1,2], function() {}],
        ['Name',['a','b'], 'should be a function']
      ];
      badArgs.forEach(function(args) {
        expect(function() {
          container.register.apply(container,args);
        }).toThrow();
      });
    });
  });
});
```

이 테스트의 내용을 정리해보자.

* container는 '테스트 대상'으로 beforeEach에서 생성된다. 테스트마다 인스턴스를 갓 구워내면 다른 테스트의 결과를 어지럽히지 않아도 된다.
* 중첩된 describes와 it 함수가 인자로 받은 문자열을 죽 이어서 읽어보면 "DiContainer register (name,dependencies,func)는 인자가 하나라도 누락되었거나 타입이 잘못되면 예외를 던진다."는 문장이 된다.
* TDD 순수주의자는 badArgs 원소마다 테스트를 따로 만들라고 하겠지만, 기대식 한 개와 서술문 한 개로 모든 테스트를 묶어 작성해도 무방하다.

테스트를 실행하면 실패한다.

DiContainer.register에 인자 체크 기능을 넣어야 성공한다.

```js
DiContainer = function() {
    // 생성자에 의해 객체가 생성되도록 강제한다.
  if (!(this instanceof DiContainer)) {
    return new DiContainer();
  }
};

DiContainer.prototype.messages = {
  registerRequiresArgs: '이 생성자 함수는 인자가 3개 있어야 합니다: ' +
    '문자열, 문자열 배열, 함수.'
};

DiContainer.prototype.register = function(name,dependencies,func) {
  var ix;

  if (typeof name !== 'string'
  || !Array.isArray(dependencies)
  || typeof func !== 'function') {
    throw new Error(this.messages.registerRequiresArgs);
  }
  for (ix=0; ix<dependencies.length; ++ix) {
    if (typeof dependencies[ix] !== 'string') {
      throw new Error(this.messages.registerRequiresArgs);
    }
  }
};
```

테스트는 성공한다.

코드를 잘 뜯어보면 메시지가 프로토타입에 있어서 외부에 드러나 있다. 이런 식으로 테스트를 더 견고하게 작성할 수 있다. 단지 함수가 toThrow()할 거라 기대하지 말고, 다음과 같이 바꾸면 더 명확한 테스트가 된다.

```js
.toThrowError(container.messages.registerRequiresArgs);
```

register 함수는 여전히 아무 일도 하지 않지만, 이 함수만으로는 의존성을 다시 끌어낼 방법이 없으므로 컨테이너에 의존성이 잘 들어갔는지 테스트하기 어렵다.

따라서 나머지 반쪽 get 함수에 관심이 쏠리다. 이 함수의 유일한 인자는 조회할 의존성 명이다.

우선 인자부터 확인해보는 게 좋다. 에러 체크는 가능한 한 빨리 해야 바른 코드가 만들어진다. '코드를 다 짤 떄까지' 내버려 뒀다간 그사이 다른 관심사로 넘어가 버리기에 십상이다.

>> TIP : 에러 처리 코드를 제일 먼저 테스트하라. 그다음 다른 업무로 넘어가도 늦지 않다.

```js
describe('get(name)', function() {
    it('성명이 등록되어 있지 않으면 undefined를 반환한다', function() {
      expect(container.get('notDefined')).toBeUndefined();
    });
});
```

get 함수 자체가 없으니 테스트는 실패할 것이다. TDD 사상에 따라 당장 에러를조치할 만큼의 코딩만 한다.

```js
DiContainer.prototype.get = function(name) {
};
```

테스트가 성공한다. 지금 여기서 다른 코드가 들어차 있으면 테스트를 앞질러 가버린 셈이다.

>> TIP : 코드가 전혀 없어도 좋으니 테스트를 성공시킬 최소한의 코드만 작성하라. 애플리케이션 코드가 테스트보다 앞서 나가면 안된다.

```js
// 2-6
it('등록된 함수를 실행한 결과를 반환한다', function() {
  var name = 'MyName',
      returnFromRegisteredFunction = "something";
  container.register(name,[], function() {
    return returnFromRegisteredFunction;
  });
  expect(container.get(name)).toBe(returnFromRegisteredFunction);
});
```

name과 returnFromRegisteredFunction 변수로 테스트를 DRY하게 유지한 채 자기 서술적인(self-documenting) 기대식을 만들었다.

>> TIP : 리터럴 대신 변수명을 잘 정해서 DRY하고 자기 서술적인 테스트를 작성하라.

register로 등록한 데이터를 저장하고 get으로 다시 추출할 수 있으면 테스트는 성공이다. 앞서 나온 코드는 생략

```js
DiContainer = function() {
  // ... 생략
  this.registrations = [];
};

DiContainer.prototype.register = function(name,dependencies,func) {  
  // ... 생략
  this.registrations[name] = { func: func };
};

DiContainer.prototype.get = function(name) {
  return this.registrations[name].func();
};
```

새로 짠 테스트는 성공하지만(2-6), 코드가 전혀 없을 때 성공했던 이전 테스트는 실패하다.

미등록 함수 케이스를 처리하도록 get을 고치면 테스트는 모두 성공한다

```js
DiContainer.prototype.get = function(name) {
  var registration = this.registrations[name];
  if (registration === undefined) {
    return undefined;
  }
  return registration.func();
};
```

이제 get은 자신이 반환하는 객체에 의존성을 제공할 수 있다.

다음 예제는 1개의 메인 객체와 2개의 의존성을 등록하는 테스트로, 메인 개겣는 두 의존성의 반환값을 합한 값을 반환한다.

```js
DiContainer.prototype.register = function(name,dependencies,func) {

  var ix;

  if (typeof name !== 'string' ||
     !Array.isArray(dependencies) ||
     typeof func !== 'function') {
    throw new Error(this.messages.registerRequiresArgs);
  }
  for (ix=0; ix<dependencies.length; ++ix) {
    if (typeof dependencies[ix] !== 'string') {
      throw new Error(this.messages.registerRequiresArgs);
    }
  }

  this.registrations[name] = { dependencies: dependencies, func: func };
};

DiContainer.prototype.get = function(name) {
  var self = this,
      registration = this.registrations[name],
      dependencies = [];
  if (registration === undefined) {
    return undefined;
  }

  registration.dependencies.forEach(function(dependencyName) {
    var dependency = self.get(dependencyName);
    dependencies.push( dependency === undefined ? undefined : dependency);
  });
  return registration.func.apply(undefined, dependencies);
};
```

register 함수에서 registrations[name] 객체에 의존성을 추가한 다음, get 함수에서 registration.dependencies에 접근하는 부분이 달라졌다.

이 예제를 통해 테스트 주도 개발 정신을 교감하고 일반적인 자바스크립트 DI 컨테이너의 작동 원리를 감 잡았기를 바란다.

### 2-2-5 의존성 주입 프레임워크 활용

이제 적합한 DI 컨테이너를 마련했으니 객체를 생성할 때마다 의존성을 하드 코딩해서 넣지 않아도 된다. 대규모 자바스크립트 애플리케이션의 출발점은 보통 설정 루틴인데, 의존성 주입 설정을 하기 적절한 지점이기도 하다.

전역 객체 MyApp 내부에서 작동하는 애플리케이션은 다음과 같은 설정 코드를 생각할 수 있다.

```js
MyApp = {};

MyApp.diContainer = new DiContainer();

MyApp.diContainer.register(
  'Service',  // 웹 서비스를 가리키는 DI 태그
  [], // 의존성 없음
  // 인스턴스를 반환하는 함수
  funtion() {
    return new ConferenceWebSvc();
  }
);

MyApp.diContainer.register(
  'Messenger',
  [],
  function() {
    return new Messenger();
  }
);

MyApp.diContainer.register(
  'AttendeeFactory',
  ['Service', 'Messenger'], // Attendee는 service 및 messenger에 의존한다.
  function(service, messenger){
    return function(attendeeId){
      return new Attendee(service, messenger, attendeeId);
    }
  }
);
```

Attendee를 만드는 함수가 아닌, Attendee를 만들 팩토리를 만드는 함수가 등록을 대신한다.

보시다시피 팩토리가 있으면 애플리케이션 깊숙한 곳에서도 DI 컨테이너로부터 Attendee를 가져올 수 있다.

```js
var attendeeId = 123;
var sessionId = 1;

// DI 컨테이너에서 attendeeId를 넘겨 Attendee를 인스턴스화한다.
var attendee = MyApp.diContainer.get('AttendeeFactory')(attendeeId);
attendee.reserve(sessionId);
```

### 2-2-6 최신 의존성 주입 프레임워크

AngularJS와 RequireJS는 많이 사용되는 오픈 소스 의존성 주입 프레임워크로 각자 독특한 장점이 있다.

#### RequireJS

RequireJS는 이 장에서 개발한 DiContainer와 구문이 흡사하다. DiContainer의 register 함수는 리콰이어JS의 전역 함수 define에, get(모듈명) 함수는 require(모듈 URL)에 각각 대응된다.

#### AngularJS

AngularJS는 단순한 DI 컨테이너 이상의 물건으로, 단일 페이지 애플리케이션(SPA) 제작에 와넞ㄴ히 '독자적으로 특화된' 프레임워크다.

AngularJS의 의존성 주입은 그 형태가 무척 다양하다. 각각 다른 객체 타입에 맞는 DiContainer.register 같은 함수가 여럿 비치되어 있다.

AngularJS가 독자적 프레임워크라고는 하지만, 중요한 기능 대부분은 마음껏 주무를 수 있다. 의존성 주입기가 마음에 안 들면 직접 만들어 사용해도 좋다.
