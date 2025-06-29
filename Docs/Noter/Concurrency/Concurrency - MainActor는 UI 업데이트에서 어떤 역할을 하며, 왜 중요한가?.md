
> [!question]
> GQ. Concurrency - MainActor는 UI 업데이트에서 어떤 역할을 하며, 왜 중요한가?


### 0. 키워드 개념 정리
1) 메인 스레드
**앱이 시작될 때 실행되는 중심 줄기.**
대부분의 UI 코드, 사용자 이벤트 처리가 이 줄기에서 일어난다. iOS앱은 기본적으로 하나의 메인 스레드에서 작동하며, SwiftUI, UIkit 모두 이 메인 스레드에서 UI가 동작하도록 설계되어 있다.
여기서 규칙은, **UI는 항상 메인 스레드에서만 바꿔야 한다**는 것. 여러 스레드가 동시에 화면을 바꾸려고 하면 충돌이나 화면 깨짐이 생길 수 있다.

2) 비동기 코드(Async Code)
Swift의 async, await 키워드나 Task는 동시에 여러 작업을 처리하려고 등장했다.
예를 들어..
```swift
Task { // 비동기 작업을 시작한다 - 스위프트의 병렬처리 시스템(async/await 체계)에서 새로운 비동기 작업을 별도의 컨텍스트에서 실행하겠다는 의미. 즉 백그라운드에서 뭔가 오래 걸리는 작업을 시도하려고 하는 것.
    let data = try await fetchUserData() // fetchUserData()의 결과가 올 때까지 잠깐 멈추고 대기.(*이 줄만 일시정지, 나머지 동작은 돌아감)
    self.userName = data.name // 위에서 받아온 data 안의 이름을 꺼내서 userName이라는 변수에 할당, UI 업데이트
}
```
이 코드는 백그라운드에서 서버에 요청을 보내고, 응답이 오면 UI에 이름을 표시하는 작업이다.

여기서 문제는
- fetchUserData()는 백그라운드에서 돌아가고
- self.userName = data.name은 화면에 표시하는 코드
-> 즉, 백그라운드 스레드에서 UI를 바꾸게 될 위험이 생긴 것!

그래서 등장한 것이 MainActor

3) `MainActor`
```swift
@MainActor
class MyViewModel: ObservableObject {
    @Published var userName: String = ""

    func loadUser() async {
        let data = await fetchUser()
        userName = data.name // 메인 스레드에서 안전하게 실행됨!
    }
}
```
이 클래스는 "나는 메인 스레드에서 실행할게~"라고 선언한 거고
안의 userName = data.name은 자동으로 메인 스레드에서 실행된다. -> UI 업데이트가 안전해진다!

### 1. 개요

Swift의 `MainActor`는 **UI 관련 작업이 항상 메인 스레드(Main Thread)에서 실행되도록 보장**하는 역할을 한다. 비동기 코드가 늘어나면서 잘못된 스레드에서 UI를 수정하면 **예상치 못한 버그나 앱 크래시**로 이어질 수 있기 때문에, Swift는 `MainActor`를 통해 **안전한 UI 업데이트 환경**을 제공한다.


### 2. 왜 MainActor가 필요할까?

**2.1 UI는 반드시 메인 스레드에서 그려져야 한다**

UIKit이나 SwiftUI는 모두 UI를 메인 스레드에서 그리는 구조다.* 백그라운드 스레드에서 UI를 변경하려고 하면 시스템이 경고를 주거나 앱이 충돌할 수 있다.

\*왜 UI는 메인 스레드에서만 그려야 할까?
1) iOS 시스템이 그렇게 설계되어 있음(단일 렌더링 파이프라인 - 오직 메인 스레드 하나만 사용함)
2) UI 업데이트는 순서가 중요함. 여러 스레드에서 동시에 UI를 수정한다면 충돌, 순서 꼬임, 깜빡임, 레이아웃 깨짐 등의 문제가 생긴다. -> 순차적이고 예측 가능한 UI 처리가 가능하도록 아예 UI는 한 줄로만 처리해! 
3) 성능 최적화

**2.2 async/await의 등장으로 코드 흐름이 바뀌었다**

비동기 코드가 생기면, 작업이 어떤 스레드에서 실행될지 명확하지 않다. `MainActor`를 명시해주면 "**이 작업은 무조건 메인 스레드에서 실행해줘**"라는 약속이 된다.

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var text: String = ""

    func fetch() async {
        let result = await fetchSomeData()
        text = result // MainActor 덕분에 메인 스레드에서 안전하게 UI 상태 변경
    }
}
```

**2.3 자동 전파* 덕분에 일관된 스레드 흐름 유지**

MainActor가 적용된 타입(예: class 전체)에선 내부 메서드나 프로퍼티 접근도 모두 메인 스레드에서 수행된다. 이 덕분에 스레드 간 전환 문제를 걱정하지 않고 코드를 쓸 수 있다.

\*자동 전파
`@MainActor`가 클래스 전체에 붙으면 그 안에 있는 모든 속성과 메서드는 자동으로 메인 스레드에서 실행되는데 이걸 자동 전파라고 부른다.

### 3. MainActor 사용 방법

**3.1 전체 클래스에 적용**

```swift
@MainActor
class MyViewModel: ObservableObject {
    ...
}
```

**3.2 특정 메서드만 메인 스레드에서 실행**

```swift
@MainActor
func updateUI() {
    self.labelText = "Updated!"
}
```

**3.3 명시적으로 MainActor 컨텍스트에서 실행**

```swift
await MainActor.run {
    self.labelText = "Safe update"
}
```


### 4. 왜 중요할까?

|문제 상황|MainActor 사용 시 해결|
|---|---|
|백그라운드에서 UI 업데이트 → 크래시|자동으로 메인 스레드에서 실행되므로 안전|
|스레드 전환 수동 관리 필요|MainActor로 선언만 하면 자동 관리|
|비동기 코드에서 일관된 스레드 흐름 유지 어려움|MainActor가 context를 보존해줌|


### 5. 정리

- SwiftUI에서는 `@MainActor`를 활용해 UI 상태 변화를 **메인 스레드에서 안전하게 수행**할 수 있다.
- ViewModel, 상태 변경 메서드 등 UI에 영향을 주는 코드는 `MainActor`를 붙이면 안정성과 가독성을 함께 확보할 수 있다.
- 비동기 환경에서도 일관된 흐름을 유지할 수 있게 해주는 **스레드 안전성의 핵심 수단**이다.

---
## Keyword

- [[병렬 처리(Swift-Concurrency)]]

#Noter