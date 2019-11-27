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

* onError: Observable에서 어떤 이유로 에러가 발생한경우 알림. onError가 발생하면 Observable이 실행을 종료하며, onNext나 onComplete 이벤트는 발생하지 않음. 오류 정보를 저장하고 있는 Throwalbe 객체를 파라미터로 전달 받음.

* onComplete: 오류가 발생하지 않았다면 Observable은 마지막 onNext를 호출후 모든 데이터가 발행을 완료했음을 알림.

코드로 작성하면 아래와 같다.

<pre><code>fun exampleObservable() {
    val observable = Observable.just(1 + 3)
    val onNext = { item: Int -> 필요한 연산 처리  }
    val onError = { throwable: Throwable -> 오류 처리  }
    val onComplete = { 발행히 완료된후 처리  }
    observable.subscribe(onNext, onError, onComplete)
  }
</code></pre>

### 1.1 just() 함수

just() 함수는 인자로 넣은 데이터를 차례로 발행. 인자는 최소 한 개의 값부터 최대 10개 값까지 넣을수 있음. 단 타입은 모두 같아야 함. 실제 데이터 발행은 subscribe() 함수를 호출해야 시작함.

* 인자가 1개인 just() 함수의 마블 다이어 그램
<img src="https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/just.item.png"></img><br/>
just() 함수를 거치면 입력한 원(인자의 상태와 속성)그대로 발행. 파이프(|) 표시는 모든 데이터 발행이 완료(onComplete)를 의미한다.

* 인자가 N개인 just() 함수의 마블 다이어 그램

<img src="https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/just.5.png"></img><br/>
just() 함수에 인자로 넣은 순서대로 1개씩 인자의 상태와 속성 그대로 발행. 모두 발행되면 onComplete 발생.

코드로 작성하면 아래와 같다.

<pre><code>fun exampleObservable() {
    val observable = Observable.just(1,2,3,4,5)
    observable.subscribe(System.out::println, {t: Throwable -> 오류처리 },{System.out.println("발행완료")})
}
</code></pre>

실행결과
<pre><code>1
2
3
4
5
발행완료
</code></pre>

## subscribe() 함수

RxJava는 내가 동작시키기 원하는 것을 사전에 정의해둔 다음 실제 그것이 실행되는 시점을 조절할 수 있다. 이때 사용 하는것이 subscribe() 함수이다.
RxJava의 데이터를 발행하는 모든 함수들은 subscribe() 함수를 호출해야 실제로 데이터가 발행된다.

subscribe() 함수의 주요 원형은 아래와 같다.

* Disposable subscribe() - onNext와 onComplete 이벤트를 무시하고 onError 이벤트가 발생했을 때만 OnErrorNotImplementedException을 발생시킨다. 주로 테스트용이나 디버깅용으로 활용.

* Disposable subscribe(Consumer<? super T> onNext) - onNext 이벤트를 처리한다. 이 함수도 onError 이벤트가 발생시 OnErrorNotImplementedException을 발생시킨다.

* Disposable subscribe(Consumer<? super T> onNext, Consumer<? super java.lang.Throwalbe> onError) - onNext와 onError 이벤트를 처리.

* Disposable subscribe(Consumer<? super T> onNext, Consumer<? super java.lang.Throwalbe> onError, Action onComplete) - onNext, onError, onComplete 이벤트를 처리.


## Disposable 객체

Disposable은 인터페이스이며 RxJava1.x의 Subscription(구독) 객체에 해당한다. 총 2가지의 함수만 정의되어 있다.

* void dispose() - Observable에게 더 이상 데이터를 발행하지 않도록 구독을 해지하는 함수. Observable이 onComplete 알림을 보냈을 때 자동으로 호출됨. onComplete 이벤트가 정상적으로 발생했다면 구독자가 별도로 호출하지 않아도됨.
* boolean isDisposed() - Observable이 구독을 해지했는지 확인하는 함수.

Disposable 객체 활용 예시 코드와 실행결과

<pre><code>fun testDisposable() {
    val observable = Observable.just(1,2,3,4,5)
    val disposable = observable.subscribe(System.out::println, { println(it.message) },{ println("onComplete()") })
    println("isDisposed() : " + disposable.isDisposed)
}
</code></pre>
<pre><code>1
2
3
4
5
onComplete()
isDisposed() : true
</code></pre>

## create() 함수

just() 함수는 데이터를 인자로 넣으면 자동으로 알림이 발생하지만 create() 함수는 개발자가 직접 onNext, onError, onComplete 알림을 호출해야 함.

*  create() 함수의 마블 다이어 그램

<img src="https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/create.png"></img><br/>

구독자에게 데이터를 발행하려면 onNext() 함수를 호출해야 하며, 모든 데이터를 발행한 후에 반드시 onComplete() 함수를 호출해야 함.

create() 함수 활용 예시 코드와 실행결과

<pre><code>fun testCreate() {
  val create = Observable.create<String> {
    it.onNext("라면")
    it.onNext("스파게티")
    it.onNext("국밥")
    it.onNext("냠냠")
    it.onComplete()
  }
  create.subscribe(System.out::println)
}
</code></pre>

<pre><code>라면
스파게티
국밥
냠냠
</code></pre>

`Observable.create()를 사용시 주의사항!!`

RxJava의 javadoc에 따르면 create() 함수는 RxJava에 익숙한 사용자만 활용하도록 권고한다. 사실 create()를 사용하지 않고 다른 팩토리 함수를 활용하면 같은 효과를 낼 수 있기 때문이다.
만약 그래도 사용해야 한다면 아래 사항을 확인해야한다.

1. Observable이 구독 해지(dispose)되었을 때 등록된 콜백을 모두 해제해야 함. 그렇지 않으면 메모리 누수(memory leak)가 발생.
1. 구독자가 구독하는 동안에만 onNext와 onComplete 이벤트를 호출해야 함.
1. 에러가 발생했을 때는 오직 onError 이벤트로만 에러 전달을 해야 함.
1. 배압(back pressure)을 직접 처리해야 함. (배압은 추후에 작성)

## fromArray() 함수

위에 다루었던 just() 함수나 create() 함수는 단일 데이터를 발행한다. 단일 데이터가 아닌경우 fromXXX() 종류의 함수를 사용한다.
RxJava 1.x에서는 from() 함수와 fromCallable() 함수만 사용했었다. 하지만 from() 함수를 배열, 반복자, 비동기 계산 등에 모두 사용하다 보니 모호함이 있어
RxJava 2에서 from() 함수를 세분화 하였다. 먼저 fromArray() 함수를 설명한다.

fromArray() 함수 활용 예시 코드와 실행결과

* java 코드

<pre><code>void testFromArray(){
  Integer[] array = {100,1000,10000};
  Observable<Integer> observable = Observable.fromArray(array);
  observable.subscribe(System.out::println);
}
</code></pre>

* kotlin 코드

<pre><code>fun testFromArray() {
    val array = arrayOf(100,1000,10000)
    val observable = Observable.fromArray(*array)
    observable.subscribe(System.out::println)
}
</code></pre>

* 실행결과

<pre><code>100
1000
10000
</code></pre>

숫자뿐만 아니라 사용자 정의 클래스 객체도 넣을 수 있다.

* 만약 Integer[]가 아닌 int[]를 사용한다면 어떤 결과가 나올까?

<pre><code>int[] array = {100,1000,10000};
  Observable.fromArray(array).subscribe(System.out::println);
</code></pre>

실행결과는 I@6bc168e5이다. 100, 1000, 10000이 출력될 것이라는 예상과는 다른 결과 이다.
RxJava에서 int 배열을 인식시키려면 Integer 배열로 변환 해야한다.

## fromIterable() 함수

Observable을 만드는 다른 방법은 Iterable 인터페이스를 구현한 클래스에서 Observable 객체를 생성 하는것이다.
Iterable 인터페이스를 구현하는 대표적인 클래스는 ArrayList, BlockingQueue, HashSet, LinkedList, Stack, TreeSet, Vector등이 있다.

1. List 객체를 Observable로 만드는 방법

formIterable()에 List 사용 예제 코드와 실행 결과

<pre><code>fun fromIterable() {
    val array = arrayListOf("Hello","RxJava","World")
    val observable = Observable.fromIterable(array)
    observable.subscribe(System.out::println)
}
</code></pre>

<pre><code>Hello
RxJava
World
</code></pre>

2. Set 객체를 Observable로 만드는 방법

formIterable()에 Set 사용 예제 코드와 실행 결과

<pre><code>fun fromIterable() {
    val set = hashSetOf<Int>(1,2,3)
    val observable = Observable.fromIterable(set)
    observable.subscribe(System.out::println)
}
</code></pre>

<pre><code>1
2
3
</code></pre>

3. BlockingQueue 객체를 Observable로 만드는 방법

formIterable()에 BlockingQueue 사용 예제 코드와 실행 결과

<pre><code>fun fromIterable() {
    val blocking = ArrayBlockingQueue<String>(200)
    blocking.add("삼성")
    blocking.add("애플")
    blocking.add("화웨이")
    val observable = Observable.fromIterable(blocking)
    observable.subscribe(System.out::println)
}
</code></pre>

<pre><code>삼성
애플
화웨이
</code></pre>

4. Map 객체를 Observable로 만드는 방법

Map 객체에 관한 Observable 클래스의 from() 함수는 존재하지 않는다. Map 인터페이스는 배열도 아니고 Iterable 인터페이스를 구현하지 않았으므로 from() 계열 함수는 존재하지 않는다.
하여 다른방법이 여러가지 있지만 fromIterable 함수를 활용하여 최대한 쉽게 코드를 작성 해보았다.

예제 코드 와 실행 결과

<pre><code>fun fromIterable() {
    val hashMap = hashMapOf("삼성" to "갤럭시", "애플" to "아이폰")
    val observable = Observable.fromIterable(hashMap.keys)
        .map { key -> hashMap[key] }
    
    observable.subscribe(System.out::println)
}
</code></pre>

<pre><code>갤럭시
아이폰
</code></pre>

우선 hashMap 객체의 키들을 fromIterable() 함수로 발행하고 map() 함수로 발행된 키를 가지고 hashMap 객체의 값을 가져오는 코드이다.
여기서 map() 함수는 추후에 설명하겠지만, 먼저 짧게 설명하자면 앞에서 발행된 데이터를 가지고 가공을 하여 새로운 객체로 만들어주는 함수이다.
위에 코드에서는 발행된 키를 가지고 hashMap[key] 코드로 값을 가져와 값이라는 데이터로 가공을 하였다.
