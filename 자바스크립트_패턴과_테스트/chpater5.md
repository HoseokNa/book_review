# 5장 콜백 패턴

**콜백**(callback)은 나중에 실행할 부차 함수(second function)에 인자로 넣는 함수다. 여기서 콜백이 실행될 '나중' 시점이 부차 함수의 실행 완료 이전이면 **동기**(synchronous), 반대로 실행 완료 이후면 **비동기**(asynchronous)라고 본다.

- [5-1 단위 테스트](#5-1-단위-테스트)
- [5-2 문제 예방](#5-2-문제-예방)

## 5-1-단위-테스트

순차적인 단위 테스트를 통해 콜백 패턴을 소개한다.

### 5-1-1 콜백 함수를 사용한 코드의 작성과 테스팅

다시 예를 들어 곧 개최될 자바스크립트 콘퍼런스를 보자. 이제 콘퍼런스 자원봉사자가 사용할 참가자 체크인 화면을 구현해야 한다.

이 신규 화면은 참가자 목록을 보여주고 그중 한 사람 또는 여러 사람을 선택(체크인한 것으로 표시)한 뒤 외부 시스템과 연동하여 체크인을 완료한다. UI 배후의 체크인 기능은 checkInService 함수에서 구현하기로 함.

참가자 체크인 여부를 비롯한 각종 정보를 Conference.attendee 함수가 생성한 객체에 담아두면 참가자 체크인 뒤처리는 checkInService의 몫이다.

업무 요건상, 하나 또는 둘 이상의 attendee 객체에 대해 checkIn 함수를 실행해야 함.

attendeeCollection 객체를 두는 것이 타당.

```js
var Conference = Conference || {};
Conference.attendeeCollection = function () {
  var attendees = [];

  return {
    contains: function (attendee) {
      return attendees.indexOf(attendee) > -1;
    },
    add: function (attendee) {
      if (!this.contains(attendee)) {
        attendees.push(attendee);
      }
    },
    remove: function (attendee) {
      var index = attendees.indexOf(attendee);
      if (index > -1) {
        attendees.splice(index, 1);
      }
    },
    getCount: function () {
      return attendees.length;
    },

    iterate: function (callback) {
      // attendees의 각 attendee에 대해 콜백을 실행한다.
    },
  };
};
```

iterate 구현하기 전에 단위 테스트를 만들어 기능 점검하자.

```js
describe("Conference.attendeeCollection", function () {
  "use strict";

  describe("contains(attendee)", function () {
    // contains 테스트
  });
  describe("add(attendee)", function () {
    // add 테스트
  });
  describe("remove(attendee)", function () {
    // remove 테스트
  });

  describe("iterate(callback)", function () {
    var collection, callbackSpy;

    // 도우미 함수
    function addAttendeesToCollection(attendeeArray) {
      attendeeArray.forEach(function (attendee) {
        collection.add(attendee);
      });
    }

    function verifyCallbackWasExecutedForEachAttendee(attendeeArray) {
      // 각 원소마다 한번씩 스파이가 호출되었는지 확인한다
      expect(callbackSpy.calls.count()).toBe(attendeeArray.length);

      // 각 호출마다 spy에 전달한 첫 번째 인자가 해당 attendee인지 확인한다
      var allCalls = callbackSpy.calls.all();
      for (var i = 0; i < allCalls.length; i++) {
        expect(allCalls[i].args[0]).toBe(attendeeArray[i]);
      }
    }

    beforeEach(function () {
      collection = Conference.attendeeCollection();
      callbackSpy = jasmine.createSpy();
    });

    it("빈 콜렉션에서는 콜백을 실행하지 않는다", function () {
      collection.iterate(callbackSpy);
      expect(callbackSpy).not.toHaveBeenCalled();
    });

    it("원소가 하나뿐인 콜렉션은 콜백을 한번만 실행한다", function () {
      var attendees = [Conference.attendee("윤지", "김")];
      addAttendeesToCollection(attendees);

      collection.iterate(callbackSpy);

      verifyCallbackWasExecutedForEachAttendee(attendees);
    });

    it("콜렉션 원소마다 한번씩 콜백을 실행한다", function () {
      var attendees = [
        Conference.attendee("태희", "김"),
        Conference.attendee("정윤", "최"),
        Conference.attendee("유리", "성"),
      ];
      addAttendeesToCollection(attendees);

      collection.iterate(callbackSpy);

      verifyCallbackWasExecutedForEachAttendee(attendees);
    });
  });
});
```

### 콜백 기능에 초점을 둔 이 테스트의 목적.

- 콜백 실행 횟수가 정확하다.
- 콜백이 실행될 때마다 알맞은 인자가 전달된다.

재스민 스파이 인스턴스 callbackSpy 사용.

> NOTE: createSpy로 만든 스파이는 말 그대로 빈 껍데기(bare spy). 따라서 호출 추적 정도의 '감시 행위' 이상의 활동은 어렵다.

forEach 활용하여 구현

```js
// ...
return {
  // ....

  iterate: function (callback) {
    attendees.forEach(callback);
  },
};
```

테스트 성공.

attendeeCollection 기능 구현 완료.

### 5-1-2 콜백 함수의 작성과 테스팅

참가자를 체크인하는 익명 함수를 attendeeCollection.iterate 함수에 바로 넣어주기만 하면 된다.

```js
var attendees = Conference.attendeeCollection();

// UI에서 선택된 참가들을 추가한다.
attendees.iterate(function (attendee) {
  attendee.checkIn();
  // 외부 서비스에 체크인을 등록한다.
});
```

함수를 정의하자 마자 다른 함수에 콜백으로 넘기는 건 자바스크립트의 강력한 특성.

하지만 문제가 발생할 수 있다.

1. 익명 콜백 함수는 콜백만 따로 떼어낼 방법이 없어서 단위 테스트가 불가.
2. 익명 함수는 디버깅을 매우 어렵게 만든다.(호출 스택에 식별자를 표시할 수 없음)

하지만 정의하자마자 바로 넘기는 콜백 함수에도 이름을 붙일 수 있음. 테스트성이 좋아지는 건 아니지만, 적어도 디버깅 작업은 훨씬 수월해짐.

```js
var attendees = Conference.attendeeCollection();

// UI에서 선택된 참가들을 추가한다.
attendees.iterate(function doCheckIn(attendee) {
  attendee.checkIn();
  // 외부 서비스에 체크인을 등록한다.
});
```

그럼 어떻게?

참가자 체크인은 중요한 기능 요건이므로 checkInService 같은 자체 모듈에 캡슐화하면 어떨까?

이렇게 하면 테스트 가능한 단위로 만들 수 있고 체크인 로직을 attendeeCollection에서 분리하여 코드를 재사용할 수도 있음.

checkInService에 의존성을 주입하는 방법도 있음.

외부 시스템의 체크인 등록 기능을 별도의 책임으로 보고 등록용 객체(checkInRecorder)는 checkInService에 주입.

다음은 checkInService.checkIN 함수의 기본 기능을 점검하는 테스트 꾸러미.

```js
describe("Conference.checkInService", function () {
  var checkInService, checkInRecorder, attendee;

  beforeEach(function () {
    checkInRecorder = Conference.checkInRecorder();
    spyOn(checkInRecorder, "recordCheckIn");

    // checkInRecorder를 주입하면서
    // 이 함수의 recordCheckIn 함수에 스파이를 심는다
    checkInService = Conference.checkInService(checkInRecorder);

    attendee = Conference.attendee("형철", "서");
  });

  describe("checkInService.checkIn(attendee)", function () {
    it("참가자를 체크인 처리한 것으로 표시한다", function () {
      checkInService.checkIn(attendee);
      expect(attendee.isCheckedIn()).toBe(true);
    });
    it("체크인을 등록한다", function () {
      checkInService.checkIn(attendee);
      expect(checkInRecorder.recordCheckIn).toHaveBeenCalledWith(attendee);
    });
  });
});
```

체크인 처리/등록 기능을 별개의 모듈로 추출한 덕분에 아주 간명한 단위 테스트가 만들어짐.

checkInService 구현 역시 간단.

```js
var Conference = Conference || {};

Conference.checkInService = function (checkInRecorder) {
  // 주입된 checkInRecorder의 참조값을 담아둔다
  var recorder = checkInRecorder;

  return {
    checkIn: function (attendee) {
      attendee.checkIn();
      recorder.recordCheckIn(attendee);
    },
  };
};
```

콜백 패턴으로 콘퍼런스 참가자를 체크인하는 기능이 테스트를 통과한 3개의 모듈의 모습이다.

```js
var checkInService = Conference.checkInService(Conference.checkInRecorder()),
  attendee = Confrence.attendeeCollection();

// UI에서 선택된 참가들을 컬렉션에 추가한다.

attendees.iterate(checkInService.checkIn);
```

## 5-2 문제 예방

콜백 패턴의 믿을성을 끌어내리는 원이은 또 있다.

'콜백 화살'과 엉뚱한 값을 가리키는 this 다.

어떻게 예방할까?

### 5-2-1 콜백 화살 눌러 펴기

'콜백 화살(callback arrow)'은 익명 콜백 함수를 남발한 극단적인 케이스다.

```js
CallbackArrow = CallbackArrow || {};

CallbackArrow.rootFunction = function () {
  CallbackArrow.firstFunction(function (arg) {
    // 첫번째 콜백 로직
    CallbackArrow.secondFunction(function (arg) {
      // 두번째 콜백 로직
      CallbackArrow.thirdFunction(function (arg) {
        // 세 번째 콜백 로직
        CallbackArrow.forthFunction(function (arg) {
          // 네 번째 콜백 로직
        });
      });
    });
  });
};
CallbackArrow.firstFunction = function (callback1) {
  callback1(arg);
};
CallbackArrow.secondFunction = function (callback2) {
  callback2(arg);
};
CallbackArrow.thirdFunction = function (callback3) {
  callback3(arg);
};
CallbackArrow.fourthFunction = function (callback4) {
  callback4(arg);
};
```

충격과 공포다.

익명 함수에 이름을 붙여 떼어 놓기만 해도 상황은 훨씬 호전된다.

```js
CallbackArrow = CallbackArrow || {};

CallbackArrow.rootFunction = function () {
  CallbackArrow.firstFunction(CallbackArrow.firstCallback);
};

CallbackArrow.firstFunction = function (callback1) {
  callback1(arg);
};
CallbackArrow.secondFunction = function (callback2) {
  callback2(arg);
};
CallbackArrow.thirdFunction = function (callback3) {
  callback3(arg);
};
CallbackArrow.fourthFunction = function (callback4) {
  callback4(arg);
};

CallbackArrow.firstCallback = function () {
  // 첫 번째 로직
  CallbackArrow.secondFunction(CallbackArrow.secondCallback);
};
CallbackArrow.secondCallback = function () {
  // 두 번째 로직
  CallbackArrow.thirdFunction(CallbackArrow.thirdCallback);
};
CallbackArrow.thirdCallback = function () {
  // 세 번째 로직
  CallbackArrow.fourthFunction(CallbackArrow.fourthCallback);
};
CallbackArrow.fourthCallback = function () {
  // 네 번째 로직
};
```

실행 결과는 똑같고 기능 차이도 없다.

하지만 다윈 테스트를 전적으로 할 수 있다는 장점이 있다. 모든 기능을 그 자체만으로 단위 테스트할 수 있는 CallbackArrow의 함수 프로퍼티로 빼낸 덕분이다. 이름도 있어서 디버깅시에도 더 편함.

### 5-2-2 this를 조심하라

콜백 함수에서 this 변수를 참조할 때는 특별히 조심해야 한다.

```js
describe("Conference.checkedInAttendeeCounter", function () {
  var counter;
  beforeEach(function () {
    counter = Conference.checkedInAttendeeCounter();
  });
  describe("increment()", function () {
    // increment 테스트
  });
  describe("getCount()", function () {
    // getCount 테스트
  });
  describe("countIfCheckedIn(attendee)", function () {
    var attendee;

    beforeEach(function () {
      attendee = Conference.attendee("태영", "김");
    });

    it("참가자가 체크인하지 않으면 인원수를 세지 않는다", function () {
      counter.countIfCheckedIn(attendee);
      expect(counter.getCount()).toBe(0);
    });
    it("참가자가 체크인하면 인원수를 센다", function () {
      attendee.checkIn();
      counter.countIfCheckedIn(attendee);
      expect(counter.getCount()).toBe(1);
    });
  });
});
```

```js
var Conference = Conference || {};

Conference.checkedInAttendeeCounter = function () {
  var checkedInAttendees = 0;
  return {
    increment: function () {
      checkedInAttendees++;
    },
    getCount: function () {
      return checkedInAttendees;
    },
    countIfCheckedIn: function (attendee) {
      if (attendee.isCheckedIn()) {
        this.increment();
      }
    },
  };
};
```

위와 같은 attendee 객체 수를 세는 counter를 구현 후 테스트까지 통과한다.

하지만 다음 결과는 실패 한다.

```js
var checkInService = Conference.checkInService(Conference.checkInRecorder()),
  attendees = Conference.attendeeCollection();
counter = Conference.checkedInAttendeeCounter();

// UI에서 선택한 참가자들을 참가자 컬렉션에 추가.
attendees.add(Conference.attendee("유지", "김"));
attendees.add(Conference.attendee("Nick", "Bradshaw"));

// 참석자들을 체크인한다.
attendees.iterate(checkInService.checkIn);

// 체크인을 마친 참가자 인원수를 세워본다.
attendees.iterate(counter.countIfCheckedIn);

console.log(counter.getCount()); // 2 가 아님
```

checkedInAttendeeCounter에서 this가 실제로 가리킨 값은 전역 window 객체임.

이런 이유로 콜백 함수는 대부분 this를 명시적으로 가리킨다.

forEach는 콜백 내부에서 참조할 객체를 두 번째 인자로 전달할 수 있음.

그런데 만약 attendeeCollection이 외부 업체가 납품한 라이브러리 코드라 수정할 수 없을 경우에는?

방법 있다.

우선 단위 테스트를 추가하자.

```js
// ...
it("this가 꼭 checkedInAttendeeCounter 인스턴스를 가리키는 것은 아니다", function () {
  attendee.checkIn();
  // this에 빈 객체를 넣고
  // counter.countIfCheckedIn를 실행한다
  counter.countIfCheckedIn.call({}, attendee);
  expect(counter.getCount()).toBe(1);
});
```

자신의 참조값을 self라는 변수에 담고 countIfCheckedIn에서 this 대시 self로 참조하면 getCount 함수를 확실히 바라 볼 수 있다.

```js
var Conference = Conference || {};

Conference.checkedInAttendeeCounter = function () {
  var checkedInAttendees = 0,
    self = {
      increment: function () {
        checkedInAttendees++;
      },
      getCount: function () {
        return checkedInAttendees;
      },
      countIfCheckedIn: function (attendee) {
        if (attendee.isCheckedIn()) {
          self.increment();
        }
      },
    };

  return self;
};
```

## 5-3 정리하기

TDD 방식으로 콜백 패턴을 구현해봤다.

테스트를 먼저 작성하고 콜백 함수를 여럿 포함한 코드를 구현하면서 this가 뜻밖에 엉뚱한 참조를 할 수 있다. 또한 익명 함수가 얼마나 테스트하기 어려운지, 여기에 중첨까지 더하면 콜백 화살이 된다는 사실을 배움.
