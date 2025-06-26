>[!question]
>[!question]
>GQ1. 1. MainActor는 Swift에서 어떤 역할을 하며 메인 스레드와 어떻게 연결되는가


## Description
### - Actor 란? : Swift Concurrency 에서 등장한 개념. 
	- 상태를 안전하게 격리(Isolation)”하고 데이터 레이스(Data Race)를 방지하기 위해 도입된 참조 타입(reference type)
```
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }

    func getValue() -> Int {
        return value
    }
}

```
위가 선언이고 아래가 호출하는 예시. 
```
let counter = Counter()

Task {
    await counter.increment()
    let v = await counter.getValue()
    print("현재 카운트: \(v)")
}

```

일종의 클레스 같은 것인데, 인스턴스 내부의 가변 상태에 대해 접근을 한번에 하나의 컨텍스트만 허용하는 것. 

### - MainActor 란? : A singleton actor whose executor is equivalent to the main dispatch queue.

```
@globalActor final actor MainActor
```

Swift의 `MainActor`는 **전역 액터(Global Actor)** 중 하나로, UI 업데이트나 사용자 상호작용 같이 메인 스레드(Main Thread)에서 실행되어야 하는 작업을 안전하게 처리하도록 보장해 줌. 


## 주요 기능
### MainActor의 역할 
- **스레드 안전성 확보**
    - `@MainActor`로 표시된 프로퍼티, 메서드, 타입은 오직 메인 스레드에서만 실행되도록 강제합니다.
    - 이를 통해 UI 관련 리소스(예: `UIView`/`NSView`, SwiftUI의 뷰 바인딩 등)에 대한 동시 접근으로 인한 데이터 레이스를 방지합니다.
        
- **코드 격리(Isolation) 제공**
    - 액터 모델(actor model)에 따라, MainActor는 자체적인 격리 컨텍스트를 가지며, 해당 컨텍스트 외부에서는 비동기 호출(`await`)을 통해서만 접근할 수 있습니다.
        
- **편리한 문법**
    - `@MainActor` 애트리뷰트를 타입, 메서드, 클로저 등에 붙이는 것만으로도 컴파일러가 메인 스레드 실행을 자동으로 관리해 줍니다.


### 2. 메인 스레드와의 연결
- 내부적으로 `MainActor`는 **`DispatchQueue.main`** 과 연동됨.
    - MainActor에 스케줄된 작업은 결국 메인 디스패치 큐에 제출되어 실행됩니다.
        
- 예를 들어, `@MainActor func updateUI()`를 호출하면:
    1. 호출 지점이 이미 메인 액터일 때는 즉시 실행   
    2. 그렇지 않으면 `DispatchQueue.main.async`처럼 메인 큐로 스케줄링 후 실행

### cf. Swift 공식 문서에 MainActor 를 검색하면 나오는 `@GlobalActor` 와 `@MainActor` 는 뭐가 다를까?
- globalActor 는 커스텀 전역 Actor. 특정 실행 컨텍스트(스레드, 큐 등)로 작업을 묶어 격리(isolation)하고 싶을 때, `GlobalActor` 프로토콜을 채택한 타입에 붙임. 
	- 즉, 직접 정의하고 싶은 모든 Actor 에 `@GlobalActor` 를 붙임. 
```
@globalActor
struct MyBackgroundActor: GlobalActor {
    // 실제 격리용 액터 인스턴스
    static let shared = Actor()
}

// 이렇게 만든 전역 액터를 통해서…
@MyBackgroundActor
func doBackgroundWork() {
    // 이 코드는 MyBackgroundActor.shared 안에서 직렬화되어 실행됩니다.
}

```
- MainActor 는 미리 정의된 빌트인 액터. 반드시 메인 스레드에서 실행해야 하는 코드를 격리할때 사용. 
```
@MainActor
func updateUI() {
    label.text = "완료됨"    // 메인 스레드에서 안전하게 실행
}

// 비동기 컨텍스트에서 호출할 때는 항상 await
Task {
    await updateUI()
}

```
- 근데? 이 `MainActor` 도 내부적으로는 `GlobalActor` 로 정의되어 있음 ! 

```
@globalActor
internal enum MainActor: GlobalActor {
    static var shared: Actor { …DispatchQueue.main… }
}

```
이렇게~ 

| 구분          | `@globalActor`                                       | `@MainActor`                       |
| ----------- | ---------------------------------------------------- | ---------------------------------- |
| 목적          | **새로운** 전역 액터를 **정의**                                | **미리 정의된** 메인 스레드 전역 액터를 **사용**    |
| 선언 위치       | `struct`나 `enum` 앞 (커스텀 액터 타입 선언)                    | 함수, 타입, 프로퍼티 앞 (격리할 대상 선언)         |
| 예시          | `@globalActor struct LoggerActor: GlobalActor { … }` | `@MainActor class ViewModel { … }` |
| 연결된 실행 컨텍스트 | 사용자가 직접 지정 (예: 백그라운드 큐, 파일 I/O 액터 등)                 | 항상 `DispatchQueue.main` (메인 스레드)   |




>[!question]
>[!question]
>GQ2. 1. UI 업데이트를 안전하게 보장하기 위해 MainActor를 어떤 방식으로 활용하면 좋을까

## 1. ViewModel·Controller 전체를 `@MainActor`로 격리하기
- UI 로직을 담당하는 클래스나 구조체 전체에 메인 액터 붙이면 자동으로 UI ㅇㅓㅂㄷㅔ이트는 메인 스레드에서 돌아감. 
- 일일히 `await` 안써도 되고, 컴파일러도 체크. (간단함)

```
@MainActor
class MyViewModel: ObservableObject {
    @Published var title: String = ""
    @Published var items: [Item] = []

    func loadData() async {
        let fetched = await fetchFromNetwork()
        // 여기선 이미 메인 스레드 보장
        self.items = fetched
    }
}
```

## 2. 특정 메서드만 `@MainActor` 지정하기
- UI 를 담당하는 메서드에만 지정해도 됨. 로직과 UI 업데이트를 명확하게 분리하는게 좋음. 

```
class DataService {
    func fetchData() async -> [String] { … }

    @MainActor
    func applyToLabel(_ data: String) {
        label.text = data
    }
}

Task {
    let data = await dataService.fetchData()
    await dataService.applyToLabel(data.first ?? "")
}
```

## 3. SwiftUI에서는 뷰 자체를 MainActor에 두기 (개인적으로는 제일 좋다고 생각함 💭)

```
@MainActor
struct ContentView: View {
    @State private var isLoading = false

    var body: some View {
        VStack {
            if isLoading { ProgressView() }
            Button("불러오기") {
                Task {
                    isLoading = true
                    await viewModel.loadData()
                    isLoading = false
                }
            }
        }
    }
}
```
- 아예 View 단과 ViewModel 역할을 분리해서 깔끔하고 명확하게 정리하는 게 좋지 않을까... 요...

## Keywords
+ [[병렬 처리(Swift-Concurrency)]]
+ [[MainActor]]

## References
- https://green1229.tistory.com/341
- https://green1229.tistory.com/343
- https://developer.apple.com/documentation/swift/mainactor
- https://1000ji.tistory.com/6

## 작성자
#Zhen 