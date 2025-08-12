
>[!question]
>Publisher와 Subscriber의 역할과 관계는 무엇인가?

### 1. 개요

Combine은 Swift에서 비동기 이벤트 스트림\*을 처리하기 위한 프레임워크이다.
Publisher와 Subscriber는 Combine의 핵심 개념!
- Publisher: 시간이 흐름에 따라 값을 방출하는 존재
- Subsriber: 그 값을 받아 처리하는 존재
이 둘은 관찰자 패턴(Observer Pattern)\* 기반으로 동작하며 데이터의 흐름을 선언적으로 연결한다.

\* 비동기 이벤트 스트림
지금 당장 오는 것이 아닌 미래에 언젠가 순차적으로 도착하는 값의 흐름
예) 버튼 이벤트 발생, 1초마다 갱신되는 위치 정보, 실시간 채팅 메시지 수신 등

\*관찰자 패턴
소프트웨어 디자인 패턴 중 하나로 어떤 객체의 상태 변화가 있을 때 그 변화를 자동으로 알려주는 구조

### 2. Publisher의 역할

Publisher?
Swift의 Combine 프레임워크 안에서 Publisher 프로토콜을 채택한 타입.

예를 들어
```swift
let pub = Just("Hello")
```
여기서 `Just<String>`은 Combine에서 제공하는 Publisher.
`pub`은 Publisher 프로토콜을 따르는 구조체 객체,
따라서 `pub`이 Publisher!

- Publisher는 시간이 지남에 따라 값을 방출(publish) 하는 객체이다.
- 예) 네트워크 응답, 사용자 입력, 타이머 이벤트 등
- 프로토콜 이름도 Publisher이며, 이 프로토콜을 채택한 모든 타입은 데이터를 발행할 수 있다.
- 데이터를 제공하는 Publisher 쪽 → 데이터를 받는 Subscriber 쪽
  이 둘을 연결하려면 반드시 subscribe(\_:)를 호출해야 한다. (sink, assign, receive(on:) 같은 메서드도 사실 내부적으로는 Subscriber를 생성하고 subscribe를 호출한다)
  기본 형식:
```swift
publisher
	.subscribe(subscriber)
```


- 대표적인 Publisher:
	- Just (한 번만 값 발행)
	- PassthroughSubject (외부에서 직접 값 발행)
	- CurrentValueSubject (초기값 유지 + 최신값 유행)
	- URLSession.DataTaskPublisher (네트워크 응답)

### 3. Subscriber의 역할

- Subscriber는 Publisher가 보낸 값, 완료(completion), 실패(failure) 이벤트를 받아서 처리하는 쪽이다.
- Subscriber는 receive(_ input: Input) 등의 메서드를 통해 데이터를 처리한다.
- 대표적인 Subscriber
	- `.sink(receiveValue:)` - 가장 일반적
	- `.assign(to:on:)` - 값 할당용
	- 커스텀 Subscriber 생성도 가능

### 4. 관계 구조

```
Publisher ----> Subscriber
   |                |
 emit value     receive value
 complete        handle done
 fail            handle error
```

- Publisher는 구독자가 연결되기 전까지는 아무 값도 방출하지 않음 (Lazy).
- 구독이 이뤄지면 **데이터 흐름이 시작**됨.

```swift
let pub = Just("Hello")
```
	이렇게 선언만 했을 때는 실제로 아무 일도 일어나지 않고
	아래처럼 구독이 발생해야 흐름이 시작된다는 얘기
```swift
let sub = pub.sink { value in
	print(value) // "Hello"
}
```

### 5. 예제 코드

```swift
import Combine

let publisher = Just("Hello, Combine!") // 일회용 Publisher

let subscription = publisher
	.sink { value in
		print("받은 값: \(value)")	
	}
// 출력: 받은 값 = Hello, Combine!
```

### 6. Publisher-Subscriber 체계의 이점

|**이점**|**설명**|
|---|---|
|선언형 프로그래밍|흐름을 선언적으로 구성 가능|
|데이터 흐름 추적 용이|구독 체계로 인해 명확한 데이터 경로 확보|
|비동기 처리 통합|타이머, 네트워크 등 다양한 비동기 소스 통합 처리 가능|
|메모리 관리 자동화|AnyCancellable을 통한 메모리 관리 지원|

### 7. 정리

Combine에서 Publisher와 Subscriber는 데이터 흐름의 **출발점과 도착점**이다.
이 관계는 SwiftUI에서도 뷰와 데이터 모델 사이의 바인딩에 활용된다.

Combine은 데이터를 “발행하고 구독하는 흐름”으로 추상화해 복잡한 비동기 로직을 간결하고 선언적으로 구성하게 해준다!

---

#### Keyword

- [[Combine]]

#### 작성자

#Noter 