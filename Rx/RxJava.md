# Reactive Programming(리액티브 프로그래밍)

데이터 흐름과 변화의 전달에 관한 프로그래밍 패러다임.
기존 명령형 프로그래밍은 작성한 코드가 정해진 순서대로 실행되는 것이라면, 리액티브 프로그래밍은 데이터 흐름을 먼저 정의하고 데이터가 변경되면 연관된 함수나 수식이 업데이트되는 방식.
기존 자바 프로그래밍은 사용자의 인터렉션(명령) 등이 발생하였을 때 데이터를 가져와 계산하는 Pull 방식이라면, 리액티브 프로그래밍은 데이터의 변화가 발생할 경우 이벤트를 받아 동작하는 Push 방식.

## 함수형 프로그래밍과 콜백, 옵서버 패턴의 차이점.

콜백과 옵서버 패턴은 수십수백 개의 스레드가 동시에 단일 자원에 접근하면 예측할 수 없는 잘못 된 결과(side effect) 가 발생할 수 있음.
함수형 프로그래밍은 이러한 side effect가 없는 pure function을 지향함.

## RxJava의 탄생 배경.

RxJava는 2013년 2월 넷플릭스의 기술 블로그에서 처음으로 소개됨. RxJava를 만들게 된 핵심적인 이유와 해결 방안은 아래와 같음.
* 동시성을 적극적으로 끌어안을 필요가 있음.
  * 자바가 동시성 처리 하는데 번거로움이 있어 다수의 비동기 실행 흐름을 생성하고 결과를 취합하여 최종 리턴하는 방식으로 설계

* 자바 Future를 조합하기 어렵다는 점을 해결해야 함.
  * 비동기 흐름을 조합할 수 있는 방법을 제공

* 콜백 방식의 문제점을 개선해야 함.
  * 코드의 가독성을 위해 콜백을 사용하지 않는 방향으로 설계

## 1. Observable

RxJava 프로그래밍은 Observable에서 시작해 Observable로 끝난다고 해도 과언이 아닐 정도로 중요한 개념이다.
Observable은 객체의 상태 변화가 있을 때마다 등록된 옵서버(관찰자)에게 알림을 보내 옵서버(관찰자)가 데이터를 처리할 수 있도록 옵서버 패턴을 구현한 클래스이다.
라이프 사이클은 존재하지 않으며 보통 단일 함수를 통해 변화만을 알린다. Android의 OnClickLisener도 옵서버 패턴의 대표적인 예이다.

RxJava의 Observable 클래스는 세 가지의 알림을 구독자에게 발행한다.

* onNext: Observable의 데이터 발행을 알림. 기존 옵서버 패턴과 동일. Observable이 배출하는 항목을 파라미터로 전달 받음.

* onError: Observable에서 어떤 이유로 에러가 발생한경우 알림. onError가 발생하면 Observable이 실행을 종료하며, onNext나 onCompleted 이벤트는 발생하지 않음. 오류 정보를 저장하고 있는 Throwalbe 객체를 파라미터로 전달 받음.

* onComplete: 오류가 발생하지 않았다면 Observable은 마지막 onNext를 호출후 모든 데이터가 발행을 완료했음을 알림.

코드로 작성하면 아래와 같다.

<pre><code>fun exampleObservable() {
    val observable = Observable.just(1 + 3)
    val onNext = { item: Int -> 필요한 연산 처리  }
    val onError = { throwable: Throwable -> 오류 처리  }
    val onCompleted = { 발행히 완료된후 처리  }
    observable.subscribe(onNext, onError, onCompleted)
  }
</code></pre>

### 1.1 just() 함수

just() 함수는 인자로 넣은 데이터를 차례로 발행. 인자는 최소 한 개의 값부터 최대 10개 값까지 넣을수 있음. 단 타입은 모두 같아야 함. 실제 데이터 발행은 subscribe() 함수를 호출해야 시작함.

* 인자가 1개인 just() 함수의 마블 다이어 그램
<img src="https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/just.item.png"></img><br/>
just() 함수를 거치면 입력한 원(인자의 데이터 타입과 형태) 그대로 발행. 파이프(|) 표시는 모든 데이터 발행이 완료(onCompleted)를 의미한다.

* 인자가 N개인 just() 함수의 마블 다이어 그램

<img src="https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/just.5.png"></img><br/>
