>[!question]
>GQ. Combine의 체이닝 연산자(map, filter, debounce 등)는 어떻게 동작하며, 언제 어떤 것을 사용하는가?

## Description
- Apple의 Combine 프레임워크는 비동기 이벤트를 선언적으로 처리하기 위한 강력한 API를 제공한다. 시간에 따라 변화하는 값을 처리하는 이 패러다임의 중심에는 Publisher, Subscriber, 그리고 이 둘을 연결하는 Operator가 있다.
- Combine의 핵심인 연산자 체이닝(Operator Chaining)의 동작 원리를 심층적으로 분석하고, `map`, `filter`, `debounce` 등 주요 연산자들을 언제, 어떻게 사용해야 하는지를 알아본다.

## 주요 기능과 코드 예시
+ Combine에서 연산자 체이닝은 단순히 문법적 편의성을 제공하는 것을 넘어, 프레임워크의 근본적인 아키텍처를 반영한다.
+ `map`, `filter`와 같은 연산자들은 독립된 함수가 아니라, 실제로는 상위(upstream) Publisher를 구독하고, 수신한 값을 변환한 뒤 하위(downstream)로 다시 방출하는 새로운 Publisher이다.
	+ 예를 들어, `Just(5).map { $0 * 2 }`라는 코드는 `Just<Int>` 타입의 Publisher를 `Publishers.Map<Just<Int>, Int>` 타입의 Publisher가 구독하여 새로운 스트림을 생성하는 구조이다.
- 이러한 구조는 각 연산자가 이전 단계의 출력을 입력으로 받아 처리하고 다음 단계로 넘기는 데이터 처리 파이프라인을 구축한다.
	- 이는 마치 UNIX 셸에서 `command1 | command2 | command3`와 같이 파이프를 통해 명령어들을 연결하는 것과 매우 유사한 원리이다.
	- 각 연산자는 데이터 스트림의 한 단계를 책임지는 독립적인 처리 단위로 작동하며, 이들을 연결함으로써 복잡한 비동기 로직을 선언적이고 읽기 쉽게 표현할 수 있다.   
- 이러한 연산자 체인이 길어지면 `Publishers.Map<Publishers.Filter<Just<Int>>, String>`과 같이 반환 타입이 매우 복잡해질 수 있다. 이때 `eraseToAnyPublisher()` 연산자를 사용하면, 복잡한 구체 타입을 숨기고 `AnyPublisher<Output, Failure>`라는 단순한 타입으로 통일하여 API 경계를 깔끔하게 유지하고 구현 세부사항을 효과적으로 감출 수 있다.

Combine의 연산자들은 그 기능에 따라 값 변환, 스트림 필터링, Publisher 결합, 오류 처리 등 다양한 카테고리로 나눌 수 있다. 각 연산자의 역할을 정확히 이해하고 상황에 맞게 사용하는 것이 중요하다.

##### 값 변환 (Transforming Values)
Upstream Publisher가 방출하는 값을 다른 형태의 값이나 새로운 Publisher로 변환한다.
`map`, `tryMap`, `flatMap`, `switchToLatest`, `scan`
- **`map`**, **`tryMap`**
    - **기능**: Upstream의 각 값을 클로저를 통해 1:1로 다른 값으로 변환한다. `tryMap`은 변환 과정에서 오류를 던질(throw) 수 있는 경우에 사용된다.
    - **실제 활용**: 네트워크 응답으로 받은 `Data` 객체를 특정 모델 객체로 디코딩하기 전 중간 단계에서 사용하거나, 숫자 데이터를 UI에 표시하기 위한 문자열로 포맷팅하는 데 유용하다.
    - **코드 예시**:
```Swift
import Combine
import Foundation
        
// URLSession.dataTaskPublisher는 (data: Data, response: URLResponse) 튜플을 방출합니다.
// map 연산자를 사용해 data 프로퍼티만 추출합니다.
let url = URL(string: "https://api.example.com/data")!
let dataPublisher = URLSession.shared.dataTaskPublisher(for: url)
	.map(\.data) // (data, response) 튜플에서 data만 하위 스트림으로 전달
	.eraseToAnyPublisher()
```
    
- **`flatMap`**
    - **기능**: Upstream의 각 값을 새로운 Publisher로 변환하고, 생성된 모든 내부 Publisher들의 이벤트를 단일 스트림으로 평탄화(flatten)하여 전달한다. 여러 비동기 작업을 순차적 또는 병렬적으로 연결할 때 핵심적인 역할을 한다.
    - **실제 활용**: 사용자 ID 목록을 받아, 각 ID에 대해 사용자 프로필을 가져오는 네트워크 요청을 병렬로 실행하고 그 결과들을 하나의 스트림으로 받아 처리하는 시나리오에 적합하다.
    
- **`switchToLatest`**
    - **기능**: `flatMap`과 유사하게 동작하지만, 오직 가장 최근에 생성된 내부 Publisher의 이벤트만 구독하고 이전의 모든 내부 Publisher는 자동으로 취소한다. 이는 경쟁 상태(race condition)를 방지하는 데 매우 효과적이다.
    - **실제 활용**: 사용자가 검색창에 텍스트를 입력하는 상황이 대표적이다. 사용자가 "A", "AP", "APP"를 순서대로 빠르게 입력하면, "A"와 "AP"에 대한 네트워크 요청은 취소되고 오직 마지막 "APP"에 대한 요청만 실행되어 그 결과만 받게 된다. `debounce`와 함께 사용될 때 최상의 사용자 경험을 제공한다.
    - **코드 예시**:
```Swift
import Combine
import Foundation

let searchQueries = PassthroughSubject<String, Never>()

let searchSubscription = searchQueries
   .map { query -> AnyPublisher<, Never> in
		// 각 쿼리에 대해 네트워크 요청 Publisher를 생성합니다.
		print("Mapping query: \(query) to a new publisher")
		return Future<, Never> { promise in
			// 시뮬레이션을 위해 딜레이를 줍니다.
			DispatchQueue.global().asyncAfter(deadline:.now() + 0.5) {
				promise(.success())
			}
		}.eraseToAnyPublisher()
	}
   .switchToLatest() // 가장 최근에 map으로 생성된 Publisher로 전환합니다.
   .sink { results in
		print("Received results: \(results)")
	}

searchQueries.send("Combine") // 이 요청은 다음 요청에 의해 취소됩니다.
searchQueries.send("SwiftUI")  // 이 요청만 결과를 방출합니다.

// 출력 (약 0.5초 후):
// Mapping query: Combine to a new publisher
// Mapping query: SwiftUI to a new publisher
// Received results:
```
        
- **`scan`**
    - **기능**: 초기값과 함께 시작하여, 매번 upstream에서 값이 올 때마다 클로저를 실행하고 그 결과(누적값)를 방출한다. `reduce`와 비슷하지만, `reduce`가 스트림이 완료될 때 최종 결과 하나만 방출하는 것과 달리 `scan`은 매 단계의 중간 결과를 방출한다.
    - **실제 활용**: 사용자의 버튼 탭 횟수를 누적하여 실시간으로 화면에 표시하거나, 입력된 숫자들의 합계를 계속해서 업데이트하며 보여줄 때 유용하다.
    - **코드 예시**:
```Swift
import Combine

let numbers = (1...5).publisher
let runningTotal = numbers
   .scan(0, { accumulatedValue, newValue in
		return accumulatedValue + newValue
	})
   .sink { print($0) }
// 출력: 1, 3, 6, 10, 15
```

| 기준     | `flatMap`                                        | `switchToLatest`                                             |
| ------ | ------------------------------------------------ | ------------------------------------------------------------ |
| 동작 방식  | 모든 내부 Publisher를 병렬 실행하고, 모든 결과를 하나의 스트림으로 합친다.  | 가장 최근에 생성된 내부 Publisher만 활성 상태로 유지하고, 이전 Publisher는 모두 취소한다. |
| 주요 사용처 | 여러 독립적이고 병렬적인 비동기 작업의 결과를 모두 수집해야 할 때.           | 사용자 입력에 반응하는 검색 기능과 같이, 최신 요청의 결과만이 의미 있을 때.                 |
| 결과 스트림 | 모든 내부 Publisher의 결과가 실행 완료 순서대로 섞여서 나온다.         | 오직 가장 마지막에 생성된 Publisher의 결과만 나온다.                           |
| 주의점    | 결과의 순서가 보장되지 않아 경쟁 상태(Race Condition)가 발생할 수 있다. | 이전 작업은 결과를 내지 못하고 취소되므로, 모든 작업의 결과가 필요한 경우에는 부적합하다.          |
##### 스트림 필터링 (Filtering Streams)
- **`filter`**
	- **기능**: 주어진 조건을 만족하는(클로저가 `true`를 반환하는) 값만 통과시킨다.   
	- **실제 활용**: 사용자 입력의 유효성 검사(예: 3글자 미만의 검색어 무시), 특정 타입의 이벤트만 처리하기 등 광범위하게 사용된다.
	- **코드 예시**:
```Swift
import Combine

let numbers = (1...10).publisher
let evenNumbers = numbers
   .filter { $0 % 2 == 0 }
   .sink { print($0) }
// 출력: 2, 4, 6, 8, 10
```
 - **`removeDuplicates`**
	- **기능**: 연속적으로 중복되는 값만 제거한다. `Equatable` 프로토콜을 따르는 타입에 기본적으로 사용할 수 있으며, `removeDuplicates(by:)`를 사용하면 직접적인 비교 로직을 제공할 수 있다.   
	- **주의사항**: 이 연산자는 스트림 전체에서 중복을 제거하는 것이 아니라, 바로 이전 값과 현재 값을 비교하여 같을 경우에만 현재 값을 제거한다.
	- **실제 활용**: 사용자가 스위치를 반복적으로 껐다 켰다 할 때, 상태가 실제로 변경된 시점에만 이벤트를 처리하고 싶을 때 유용하다.
- **`compactMap`** 
	- **기능**: 각 값을 변환하되, 변환 결과가 `nil`이면 해당 값을 스트림에서 제거하고 `nil`이 아닌 값만 하위로 전달한다.
	- **실제 활용**: 문자열 배열에서 유효한 `Int` 값만 추출하거나, 옵셔널 값을 포함하는 시퀀스에서 `nil`이 아닌 값만 안전하게 추출하여 처리할 때 사용된다.
- **`debounce`**
	- **기능**: 지정된 시간 동안 새로운 값이 들어오지 않을 때만 마지막 값을 방출한다.
	- **실제 활용**: 검색창 자동 완성 기능에서 사용자가 타이핑을 멈춘 후 0.5초가 지나면 네트워크 요청을 보내도록 하여, 키 입력마다 API를 호출하는 비효율을 방지한다.
	- **코드 예시**:
```Swift
import Combine
import Foundation

let typingSubject = PassthroughSubject<String, Never>()
let debouncedSubscription = typingSubject
   .debounce(for:.seconds(0.5), scheduler: DispatchQueue.main)
   .sink { print("API Call with: \($0)") }

typingSubject.send("S")
typingSubject.send("Sw")
typingSubject.send("Swi")
typingSubject.send("Swif")
typingSubject.send("Swift") // 0.5초 후 이 값만 방출됩니다.
// 출력 (마지막 입력 후 0.5초 뒤): API Call with: Swift
```
- **`throttle`**
	- **기능**: 지정된 시간 간격 내에 첫 번째 또는 마지막 값 하나만 방출하여 이벤트의 빈도를 일정하게 조절한다.
	- **실제 활용**: 사용자가 화면을 스크롤하거나 드래그하는 위치를 추적할 때, 0.1초에 한 번씩만 위치를 업데이트하여 과도한 처리 부담을 줄이고 UI 성능을 유지하는 데 사용된다.
	- **코드 예시**:
```Swift
import Combine
import Foundation

let scrollSubject = PassthroughSubject<CGFloat, Never>()
let throttledSubscription = scrollSubject
.throttle(for:.seconds(1.0), scheduler: DispatchQueue.main, latest: true)
.sink { print("Updating UI with scroll position: \($0)") }

// 1초 내에 여러 이벤트가 발생해도 마지막 값만 처리됩니다.
scrollSubject.send(100.0)
scrollSubject.send(120.0)
scrollSubject.send(150.0) // 1초 경과 시점에 이 값이 방출될 가능성이 높습니다.
```

##### Publisher 결합 (Combining Publishers)
여러 Publisher를 하나의 스트림으로 합치거나 조합하여 더 복잡한 데이터 흐름을 만듭니다.
- **`combineLatest`**
    - **기능**: 여러 Publisher를 결합하여, 그 중 어느 하나라도 새로운 값을 방출할 때마다 각 Publisher의 가장 최근 값들을 모아 튜플로 방출합니다. 단, 모든 Publisher가 최소 한 번 이상 값을 방출해야 첫 번째 튜플이 방출된다는 점에 유의해야 합니다.   
    - **실제 활용**: 로그인 폼에서 아이디 필드(`isValidIDPublisher`)와 비밀번호 필드(`isValidPasswordPublisher`)의 유효성을 나타내는 Publisher들을 결합하여, 두 필드가 모두 유효할 때만 로그인 버튼(`loginButton.isEnabled`)을 활성화하는 로직에 이상적입니다.
```Swift
import Combine

let usernamePublisher = PassthroughSubject<String, Never>()
let passwordPublisher = PassthroughSubject<String, Never>()

let validationSubscription = usernamePublisher
   .combineLatest(passwordPublisher)
   .map { username, password -> Bool in
        return!username.isEmpty && password.count >= 8
    }
   .sink { isValid in
        print("Login form is valid: \(isValid)")
    }

usernamePublisher.send("myUser") // 아직 출력 없음
passwordPublisher.send("1234")   // 출력: Login form is valid: false
passwordPublisher.send("12345678") // 출력: Login form is valid: true
```
- **`zip`**
	- **기능**: 여러 Publisher를 짝지어, 모든 Publisher가 각각 새로운 값을 방출할 때까지 기다렸다가 해당 값들을 튜플로 묶어 방출한다. 방출 횟수는 가장 적게 방출하는 Publisher의 횟수에 맞춰진다.   
	- **실제 활용**: 두 개의 서로 다른 네트워크 요청(예: 사용자 정보, 사용자 게시물 목록)이 모두 성공적으로 완료되었을 때, 두 결과를 조합하여 화면에 한 번에 표시해야 하는 경우에 사용된다.
	- **코드 예시**:
```Swift
import Combine

let publisherA = PassthroughSubject<Int, Never>()
let publisherB = PassthroughSubject<String, Never>()

let zippedSubscription = publisherA
   .zip(publisherB)
   .sink { intValue, stringValue in
        print("Zipped: \(intValue), \(stringValue)")
    }

publisherA.send(1)
publisherA.send(2)
publisherB.send("A") // 출력: Zipped: 1, A
publisherB.send("B") // 출력: Zipped: 2, B
publisherA.send(3)   // 출력 없음 (publisherB가 새 값을 보낼 때까지 대기)
```
- **`merge`**
	- **기능**: 동일한 `Output`과 `Failure` 타입을 가진 여러 Publisher를 하나의 스트림으로 합칩니다. 각 Publisher에서 값이 도착하는 순서대로 그대로 방출하여 이벤트를 인터리빙(interleaving)합니다.   
	- **실제 활용**: '새로고침' 버튼 탭 이벤트와 '화면 진입' 이벤트를 하나의 스트림으로 합쳐서 데이터 로딩 로직을 한 번만 구현하고 싶을 때 유용합니다.
	- **코드 예시**:
```Swift
import Combine

let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<Int, Never>()

let mergedSubscription = publisher1
   .merge(with: publisher2)
   .sink { print("Merged value: \($0)") }

publisher1.send(10) // 출력: Merged value: 10
publisher2.send(20) // 출력: Merged value: 20
publisher1.send(11) // 출력: Merged value: 11
```

| 기준                  | `merge`                            | `zip`                                                 | `combineLatest`                                                                  |
| ------------------- | ---------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------- |
| **방출 조건**           | 결합된 Publisher 중 어느 것이든 값이 오면 즉시 방출 | 모든 Publisher가 한 쌍을 이룰 수 있는 새로운 값을 각각 방출할 때까지 기다렸다가 방출 | 모든 Publisher가 최소 한 번 값을 방출한 이후, 그 중 어느 것이든 새로운 값이 오면 '모든 Publisher의 최신 값 조합'을 방출 |
| **방출 값**            | 단일 값 (`Output`)                    | 튜플 (`(Output1, Output2,...)`)                         | 튜플 (`(Output1, Output2,...)`)                                                    |
| **Publisher 타입 제약** | `Output`과 `Failure` 타입이 모두 동일해야 함  | 타입이 달라도 됨                                             | 타입이 달라도 됨                                                                        |
| **핵심 개념**           | Interleaving (끼워넣기)                | Pairing (동기화)                                         | Latest Values (최신 값 조합)                                                          |
| **주요 사용처**          | 여러 이벤트 소스를 하나의 로직으로 처리할 때.         | 여러 비동기 작업의 결과가 모두 필요하며, 순서가 중요할 때.                    | 여러 상태 값을 조합하여 파생된 하나의 상태를 실시간으로 만들어낼 때.                                          |

##### 오류 처리 (Handling Errors)
비동기 작업에서 필연적으로 발생하는 오류를 우아하게 처리하고 스트림의 흐름을 제어합니다.
- **`catch`**
    - **기능**: Upstream에서 오류가 발생했을 때, 스트림을 실패로 종료하는 대신 다른 Publisher를 반환하여 스트림을 계속 이어가게 한다.   
    - **실제 활용**: 주 API 서버에서 데이터 로딩에 실패했을 때, 백업 서버에 다시 요청하거나 로컬 캐시에서 데이터를 가져오는 Publisher로 대체하는 복구 로직을 구현할 수 있다.
- **`replaceError(with:)`**
	- **기능**: 오류가 발생하면 스트림을 특정 기본값으로 대체하고 정상적으로 완료시킨다. `catch`보다 간단한 오류 처리 방식이다.
	- **실제 활용**: 사용자 프로필 이미지 다운로드에 실패했을 때, 기본 아바타 이미지를 표시하는 경우에 적합하다.
- **`retry`**
	- **기능**: 오류 발생 시, 스트림을 다시 구독하여 지정된 횟수만큼 작업을 재시도한다.
	- **실제 활용**: 일시적인 네트워크 불안정으로 요청이 실패했을 때, 1초 간격으로 2~3회 재시도하는 로직을 간단하게 구현할 수 있다.

##### 멀티캐스팅 (Sharing Subscriptions)
하나의 Publisher에 대한 구독을 여러 Subscriber가 공유하여 불필요한 작업을 방지합니다.
- **`share`**
    - **기능**: 하나의 upstream Publisher에 대한 구독을 여러 downstream Subscriber가 공유하게 한다. 이를 통해 네트워크 요청과 같은 값비싼 작업이 Subscriber 수만큼 중복 실행되는 것을 방지한다. `share()`는 Publisher를 구조체(값 타입)에서 클래스 인스턴스(참조 타입)로 변환하여 이 기능을 구현한다.   
    - **실제 활용**: 하나의 데이터 요청 결과를 받아 여러 UI 컴포넌트(예: 제목 레이블, 내용 레이블, 작성자 이미지뷰)가 동시에 업데이트해야 할 때 필수적이다. `share()`가 없으면 각 UI 컴포넌트가 개별적으로 구독하여 네트워크 요청이 3번 발생하게 된다.
    - **코드 예시**:
```Swift
import Combine
import Foundation

let sharedNetworkPublisher = URLSession.shared.dataTaskPublisher(for: URL(string: "https://api.example.com/user/1")!)
   .map(\.data)
   .share() // 이 연산자로 인해 네트워크 요청은 단 한 번만 발생합니다.

let sub1 = sharedNetworkPublisher.sink { print("Subscriber 1 received data: \($0.count) bytes") }
let sub2 = sharedNetworkPublisher.sink { print("Subscriber 2 received data: \($0.count) bytes") }
```
- **`multicast`**
- **기능**: `share`보다 더 정교한 제어를 제공한다. `Subject`(예: `PassthroughSubject`)를 직접 제공하여 언제 이벤트를 방출할지, 어떻게 연결할지를 제어할 수 있다. `connect()` 메서드를 명시적으로 호출해야 비로소 upstream 구독이 시작된다.   
- **실제 활용**: 여러 Subscriber가 모두 준비될 때까지 기다렸다가, 준비가 완료된 시점에 `connect()`를 호출하여 데이터 스트림을 한 번에 시작하고 싶을 때 사용된다.

## Keywords
+ [[Combine]]

## References
- [Apple Developer Documentation: Combine](https://developer.apple.com/documentation/combine)

## 작성자
- #Air 