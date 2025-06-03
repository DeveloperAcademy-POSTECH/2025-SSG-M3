
---

> [!question]
> 비동기 함수의 단위 테스트는 어떤 전략으로 접근해야 할까?

---


## 개요

비동기 함수의 단위 테스트는 타이밍과 비결정성을 제어할 수 있어야 한다.
단순히 결과만 확인하는 것이 아니라, 비동기 흐름에서 의도한 타이밍에 정확히 호출되는지, 
결과가 기대와 일치하는지, 실패 시나리오도 잘 처리하는지까지 확인하는 것이 핵심이다.

Swift의 async/await 구조는 테스트를 깔끔하게 만들 수 있지만,
**단위 테스트에서는 외부 의존성과 동시성의 복잡도** 를 분리해서 제어 가능하게 만들어야 한다.

---

## **1. async 함수 테스트의 핵심 전략**

### **XCTest 에서의** async 테스트 

Swift의 XCTest는 async 테스트 메서드를 기본적으로 지원한다.
즉, 아래처럼 await를 쓸 수 있다.

```swift
func test_fetchData_returnsExpectedResult() async throws {
    let result = try await fetchData()
    XCTAssertEqual(result, expected)
}
```

하지만 이것만으로 충분하지 않다.
**실제 단위 테스트에서는 외부 비동기 작업(fetch, delay, API 호출 등)을 mocking하는 설계가 필요하다.**

---

## **2. 의존성 주입으로 비동기 작업 격리하기**

### **테스트 전략: “비동기 작업을 추상화한다”**

예를 들어, NetworkService가 fetchData()라는 비동기 메서드를 가진다고 하자.
이걸 직접 테스트하는 대신, **프로토콜로 추상화**해서 mock 가능한 구조로 만든다.

```swift
protocol DataFetcher {
    func fetchData() async throws -> String
}

class RealDataFetcher: DataFetcher {
    func fetchData() async throws -> String {
        // 실제 네트워크 호출
    }
}

class MockDataFetcher: DataFetcher {
    var valueToReturn: String = ""
    func fetchData() async throws -> String {
        return valueToReturn
    }
}
```

이런 구조는 테스트 코드에서 외부 요인을 걷어내고,
**순수하게 로직만 테스트할 수 있게 해준다.**

---

## **3. 테스트 시나리오를 분리**

### **주요 시나리오 예시**

- 정상적으로 데이터를 받아오는 경우
- 실패(에러)가 발생하는 경우
- 예상 시간 내에 완료되지 않는 경우 (타임아웃)
- 클로저나 콜백이 중복 호출되지 않는지 확인하는 경우

각 시나리오에 대해 Mock을 설정해 명확한 기대값을 가진다.
```swift
func test_viewModel_receivesSuccess() async throws {
    let mock = MockDataFetcher()
    mock.valueToReturn = "Hello" // 의도한 테스트 환경을 만듦
    
    let viewModel = MyViewModel(dataFetcher: mock)
    await viewModel.loadData()
    
    XCTAssertEqual(viewModel.text, "Hello")
}
```



---

## **4.** **XCTestExpectation을 써야 할 때도 있다**

### **콜백 기반 or Combine 구조와 섞일 때**

Swift의 async/await만 쓰는 구조라면 XCTest의 async 지원만으로 충분하지만,
**Combine이나 클로저 기반 코드**, **타이머 등 비동기 이벤트 발생 시점이 명확하지 않은 경우**는
XCTestExpectation을 쓰는 것이 유리하다.

```swift
func test_asyncOperation_withExpectation() {
    let exp = expectation(description: "Async finished")

    someAsyncMethod {
        exp.fulfill()
    }

    wait(for: [exp], timeout: 1.0)
}
```

단, 이 방식은 async/await보다는 복잡하고, 테스트 코드가 지저분해질 수 있다.
→ **가능하다면 구조 자체를 async/await으로 바꾸는 리팩터링도 고려할 만하다.**

---

## **@Published + Combine을 활용한 상태 검증 전략**

SwiftUI나 MVVM 환경에서는 @Published로 선언된 프로퍼티를 통해 ViewModel의 상태를 외부에서 관찰할 수 있다.
이를 통해 비동기 함수 실행 후의 상태 변화를 테스트 코드에서 직접 구독하고 검증할 수 있다

### **언제 유용할까?**
- loadData()와 같은 비동기 함수 실행 이후, 특정 상태 변화가 발생하는지를 확인하고 싶을 때
- ViewModel 내부 상태가 비동기 흐름에 따라 여러 번 바뀌는 경우

```swift
func test_loadingState_isUpdated() {
    let viewModel = MyViewModel()
    var states: [LoadingState] = []

    let cancellable = viewModel.$state
        .sink { state in
            states.append(state)
        }

    viewModel.loadData()

    // 비동기 작업 이후에도 일정 시간 기다려줘야 함
    DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) {
        XCTAssertEqual(states, [.idle, .loading, .success])
    }

    cancellable.cancel()
}
```
→ viewModel.loadData()라는 **비동기 로직을 호출한 후**,
ViewModel의 state 값이 .idle → .loading → .success 순서로 잘 바뀌는지를 검증하려는 것이다.

#### 동작 원리
### **1.**  **@Published + Combine = 관찰 가능한 상태**
```swift
let cancellable = viewModel.$state
    .sink { state in
        states.append(state)
    }
```
- @Published var state: LoadingState는 내부 상태가 바뀔 때마다 Combine의 **Publisher** 역할을 한다.
- viewModel.$state는 Combine의 **Published.Publisher**이다.
- 여기에 .sink를 붙이면, 상태가 바뀔 때마다 콜백이 호출된다.
    즉, states 배열에 상태 변화 기록을 하나씩 쌓아가고 있는 것이다.
→ **Combine은 여기서 @Published를 구독하고 상태 흐름을 기록하는 데 사용된다.**


### **2. 상태 흐름 전체를 기록한다**

```swift
var states: [LoadingState] = []
```
- .sink 구독으로 인해 state의 변화를 순차적으로 states에 누적시킨다.
- 예를 들어,
    - 테스트 시작 시 초기값 .idle
    - loadData() 호출 → .loading
    - 네트워크 성공 → .success
        이런 흐름을 **배열로 쌓아서 나중에 통째로 비교**하는 전략이다.

이 방식은 상태 변화가 연속적으로 일어날 때 유용하다. 중간 상태도 모두 확인할 수 있다.

### **3. 검증 타이밍을 맞춰야 한다**
```swift
DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) {
    XCTAssertEqual(states, [.idle, .loading, .success])
}
```
- 비동기 작업은 시간차를 두고 완료되므로, **바로 비교하면 안 된다**.
- asyncAfter를 써서 약간 기다린 후 XCTAssertEqual로 검증한다.
- 이건 테스트 흐름에서 **비동기 타이밍을 수동으로 맞추는 기법**이다.

하지만 이 방식은 테스트 코드가 깔끔하지 않고,
→ 보통은 XCTestExpectation이나 await을 쓰는 구조로 리팩터링하는 것이 더 좋다.


#### 정리
UI와 ViewModel 사이의 반응형 상태 바인딩을 그대로 테스트에서도 가져온 방식이다.
**Combine을 활용하면 state의 변화를 시간순으로 추적 가능**하고,
테스트에서는 이 흐름을 통해 앱의 상태 전이 로직이 **정확히 구현되었는지**를 확인할 수 있다.

다만, DispatchQueue.main.asyncAfter와 같은 임시 대기 전략보다는
XCTestExpectation, 혹은 async/await 기반의 테스트 리팩터링을 통해 **보다 신뢰도 높은 테스트**를 만드는 것이 장기적으로 중요하다.


---

## **5. 단위 테스트 설계 팁**

- 외부 의존성은 모두 Mock 또는 Stub으로 치환하자.
- Task.sleep 등 타이밍이 개입된 코드는 최대한 피하고, 타이밍 자체를 주입 가능한 값으로 바꾸자.
- 테스트 대상이 실제로 수행하는 핵심 로직은 순수 함수처럼 작성하는 것이 이상적이다.
- actor, Sendable, MainActor 등의 병렬성 도구를 썼다면, 관련 특성도 함께 검증해야 한다.

---

##  **Insight**

단위 테스트는 언제나 _속도_, _신뢰도_, _격리성_ 이 중요하다.
비동기 코드가 등장하면 이 셋이 쉽게 깨진다.

그래서 “어떻게 비동기 로직을 주입 가능하게 만들고”,
“타이밍의 흐름을 통제 가능한 구조로 만들 것인가?“가 설계의 핵심이다.

Swift에서 async/await는 테스트 설계에도 강력한 도구다.
하지만 그 기반 위에서 **의존성 추상화와 테스트 전략**이 뒷받침되지 않으면,
여전히 예측 불가능하고 flaky한 테스트가 나올 수 있다.



---

## GQ
- **테스트 대상 함수 내부에서 async 작업이 연속적으로 호출될 때, 상태 검증은 어디서 어떻게 해야 할까?**


## Keywords
- [[병렬 처리(Swift-Concurrency)]]


#Onething 