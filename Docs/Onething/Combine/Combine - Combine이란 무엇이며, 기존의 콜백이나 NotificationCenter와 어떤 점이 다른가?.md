
> [!question] **GQ 1**: Combine이란 무엇이며, 기존의 콜백이나 NotificationCenter와 어떤 점이 다른가?

## 개요

Combine은 Apple이 iOS 13부터 도입한 **반응형 프로그래밍 프레임워크**다. 

**비동기 이벤트를 데이터 스트림으로 모델링**하고, 이를 **Publisher → Operator → Subscriber** 체인으로 처리하는 **선언형 프레임워크**다. 

반대로 콜백과 NotificationCenter는 **명령형** 방식이라 흐름이 코드 전반에 흩어지기 쉽다. 
핵심 차이는 “어떻게 할 것인가”가 아니라 **“무엇을 원하는가를 선언하고 나머지는 연산자에게 맡긴다”** 는 관점 전환에 있다.

- 비동기 값을 내보내는 **Publisher**, 값을 가공하는 **Operator**, 값을 소비하는 **Subscriber**.
- **map → filter → debounce → flatMap → receive(on) → sink 같은 체인으로 흐름을 기술한다.
- 타입 안정, **가시성 높은 흐름**, **스케줄러 제어**, **취소/에러/완료**를 체계화한다.

---
## 기존 방식과의 본질적 차이
#### **흐름의 가시성**
- **기존**: 콜백/노티는 이벤트가 파일 곳곳에 흩어진다. 전체 흐름을 한눈에 파악하기 어렵다.
- **Combine**: 입력부터 가공·에러·UI 반영까지 **한 줄 체인**으로 모인다.
```swift
$query
  .debounce(for: .milliseconds(400), scheduler: RunLoop.main)
  .removeDuplicates()
  .flatMap(api.search)          // 비동기 합성
  .receive(on: DispatchQueue.main)
  .sink(receiveCompletion: { _ in }, receiveValue: render)
  .store(in: &bag)
```

###  **타입 안전성**
- **기존**: NotificationCenter는 Any 페이로드 → 다운캐스팅/런타임 에러 위험. 콜백 시그니처도 제각각.
- **Combine**: Publisher<Output, Failure>로 **값/에러 타입을 제네릭**으로 모델링. 컴파일 타임 검증.
```swift
// 타입이 보이는 스트림
let p: AnyPublisher<[User], APIError>
```

### **중간 연산자(조합성)**
- **기존**: 디바운스·중복 제거·재시도·백프레셔 등 타이밍/정책 코드를 매번 수작업.
- **Combine**: map, filter, debounce, throttle, retry, catch, switchToLatest 등으로 **정책을 선언**한다.
```swift
events
  .throttle(for: .seconds(1), scheduler: RunLoop.main, latest: true)
  .retry(2)
  .catch { _ in Just(.empty) }
```

### **취소와 수명 관리**
- **기존**: 각자 만든 토큰/플래그/Task.cancel 등 **취소 방식이 제각각**. 누수 위험.
- **Combine**: AnyCancellable과 store(in:)로 **수명과 취소를 문법적으로 일관**시킨다.
```swift
import Combine

@MainActor
final class SearchViewModel: ObservableObject {
  @Published var query: String = ""
  @Published private(set) var results: [String] = []
  private var bag = Set<AnyCancellable>()
  private let api: (String) -> AnyPublisher<[String], Error>

  init(api: @escaping (String) -> AnyPublisher<[String], Error>) {
    self.api = api

    $query
      .map { $0.trimmingCharacters(in: .whitespacesAndNewlines) }
      .removeDuplicates()
      .debounce(for: .milliseconds(400), scheduler: RunLoop.main)
      .filter { $0.count >= 2 }
      .map { [api] q in api(q) }   // 쿼리를 네트워크 퍼블리셔로
      .switchToLatest()            // 최신 요청만 유지
      .receive(on: DispatchQueue.main)
      .replaceError(with: [])      // 단순화: 에러시 빈 배열
      .assign(to: &$results)
  }
}
```

```swift
publisher.sink { _ in }.store(in: &bag) // 뷰 사라지면 자동 해제
```
> **스케줄러 제어**: 발행과 수신 스레드를 코드로 명시한다. subscribe(on:) / receive(on:) 로 UI 안전성 확보.



---
## 기존 방식의 한계점들
### 콜백(Closure) 방식의 문제점
콜백 기반 비동기 처리는 다음과 같은 구조적 한계를 갖는다:

```swift
// 전형적인 콜백 지옥
searchAPI.fetchUsers { users in
    if let firstUser = users.first {
        self.fetchUserProfile(firstUser.id) { profile in
            self.fetchUserPosts(profile.id) { posts in
                DispatchQueue.main.async {
                    self.updateUI(profile: profile, posts: posts)
                }
            }
        }
    }
}
```

#### 1) **흐름의 분산**: 비즈니스 로직이 여러 콜백으로 흩어져 전체적인 흐름 파악이 어렵다. (=콜백 지옥, 제어권 역전)
여러 단계가 중첩될수록 로직이 “오른쪽으로” 기울며 추론 난이도가 확 치솟는다. 에러/취소/스레드 전환이 섞이면 한층 복잡해진다.


#### 2) **타입 안전성 부족**: 각 콜백의 시그니처가 제각각이고, 에러 처리와 취소 로직을 매번 다르게 구현해야 한다.
- 타입/프로토콜 시그니처의 불일치 → 에러·결과·취소 모델이 제각각
- 콜백 시그니처가 팀/모듈마다 달라진다. 어떤 곳은 (Model?, Error?), 어떤 곳은 Result<Model, Error>, 어떤 곳은 (Model) -> Void + 별도 에러 채널… 취소는 또 토큰/플래그/URLSessionTask.cancel() 등 케이스 별 구현.
```swift
// A 팀: (Model?, Error?)
func fetchUser(id: String, completion: @escaping (User?, Error?) -> Void)

// B 팀: Result
func fetchProfile(_ id: String, completion: @escaping (Result<Profile, APIError>) -> Void)

// C 팀: 성공만, 실패는 Notification으로…
func fetchPosts(_ id: String, completion: @escaping ([Post]) -> Void)
```
- 에러를 한 가지 모델로 **집계**해 상위로 올리기가 어렵다.
- **취소**를 일관성 있게 처리하기 힘들다
	- 최소한 콜백계에서도 **공통 프로토콜**을 강제하면 낫다.
```swift
protocol AsyncTask {
    associatedtype Output
    func start(_ completion: @escaping (Result<Output, Error>) -> Void)
    func cancel()
}
```
> 하지만 이걸 팀 전체에 관철시키기 어렵고, 결국 “구현체 어댑터 지옥”이 생긴다.

- **조합의 어려움**: 디바운싱, 중복 제거, 재시도 같은 고급 패턴을 구현하려면 상당한 boilerplate 코드가 필요하다.


### NotificationCenter의 한계
NotificationCenter는 브로드캐스트 패턴의 특성상 다음과 같은 문제를 갖는다:
```swift
// 타입 안전성이 보장되지 않는 기존 방식
NotificationCenter.default.addObserver(
    forName: .userDidLogin,
    object: nil,
    queue: .main
) { notification in
    // Any 타입의 userInfo를 직접 캐스팅해야 함
    guard let userInfo = notification.userInfo,
          let userId = userInfo["userId"] as? String else { return }
    // 에러 처리나 완료 개념이 없음
}
```

- **타입 안전성 부족**: 페이로드가 `Any` 타입이므로 런타임 캐스팅 오류 위험이 있다.
- **의존성 추적 어려움**: 옵저버가 많아질수록 누가 무엇을 구독하는지 파악하기 힘들다.
- **에러 전파 체계 부재**: 에러나 완료 신호에 대한 표준화된 처리 방식이 없다.

---
## Combine의 핵심 아키텍처
Combine은 다음 세 가지 핵심 구성요소로 이루어진다
### Publisher-Operator-Subscriber 파이프라인
```swift
// 선언형 체인의 예시
publisher
    .map { $0.uppercased() }           // Operator
    .filter { $0.count > 3 }           // Operator  
    .debounce(for: .seconds(0.5), scheduler: RunLoop.main) // Operator
    .sink { value in                   // Subscriber
        print("Received: \(value)")
    }
```

**Publisher**: 값을 방출하는 주체. `Just`, `PassthroughSubject`, `@Published` 등이 있다.
**Operator**: 스트림을 변환하는 중간 처리자. `map`, `filter`, `debounce`, `flatMap` 등이다.
**Subscriber**: 최종적으로 값을 소비하는 주체. `sink`, `assign` 등이다.

### 타입 시스템의 강화

Combine의 가장 큰 혁신 중 하나는 **타입 안정성**이다:
```swift
// Publisher<Output, Failure> 제네릭을 통한 타입 안전성
let stringPublisher: AnyPublisher<String, Never>
let intPublisher: AnyPublisher<Int, URLError>

// 컴파일 타임에 타입 불일치를 잡아낸다
stringPublisher
    .map { Int($0) ?? 0 }  // String -> Int 변환이 명확히 드러남
    .sink { intValue in    // 이제 intValue는 확실히 Int 타입
        print(intValue)
    }
```

### Scheduler를 통한 실행 컨텍스트 제어

기존 방식에서는 스레드 전환이 암묵적이었다면, Combine에서는 이를 명시적으로 선언한다:
```swift
networkPublisher
    .receive(on: DispatchQueue.main)  // UI 업데이트는 메인 스레드에서
    .sink { [weak self] data in
        self?.updateUI(with: data)
    }
```

---

## 3) 실제 코드 비교: Before vs After
### Before: 검색 기능
```swift
class SearchViewController: UIViewController {
    @IBOutlet weak var searchBar: UISearchBar!
    private var searchTimer: Timer?
    private var currentTask: URLSessionDataTask?
    private var lastQuery: String = ""
    
    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        // 기존 타이머 무효화
        searchTimer?.invalidate()
        
        // 기존 요청 취소
        currentTask?.cancel()
        
        // 중복 쿼리 체크
        guard searchText != lastQuery else { return }
        lastQuery = searchText
        
        // 디바운싱
        searchTimer = Timer.scheduledTimer(withTimeInterval: 0.5, repeats: false) { _ in
            self.performSearch(query: searchText)
        }
    }
    
    private func performSearch(query: String) {
        guard !query.isEmpty else { return }
        
        currentTask = URLSession.shared.dataTask(with: searchURL(query)) { data, response, error in
            DispatchQueue.main.async {
                if let error = error {
                    self.handleError(error)
                } else if let data = data {
                    self.processResults(data)
                }
            }
        }
        currentTask?.resume()
    }
}
```

### After: Combine을 활용한 선언형 구현
```swift
class SearchViewModel: ObservableObject {
    @Published var query: String = ""
    @Published private(set) var results: [SearchResult] = []
    
    private var cancellables = Set<AnyCancellable>()
    private let searchAPI: SearchAPI
    
    init(searchAPI: SearchAPI) {
        self.searchAPI = searchAPI
        setupSearchPipeline()
    }
    
    private func setupSearchPipeline() {
        $query
            .map { $0.trimmingCharacters(in: .whitespacesAndNewlines) }
            .removeDuplicates()                    // 중복 제거
            .debounce(for: .milliseconds(500), scheduler: RunLoop.main) // 디바운싱
            .filter { $0.count >= 2 }              // 최소 글자 수 필터링
            .flatMap { [searchAPI] query in
                searchAPI.search(query)
                    .catch { _ in Just([]) }       // 에러를 빈 배열로 변환
            }
            .receive(on: DispatchQueue.main)       // UI 업데이트는 메인 스레드
            .assign(to: &$results)                 // 결과를 results에 할당
    }
}
```

차이점이 극명하다. 기존 방식에서는 **상태 관리**, **타이머 제어**, **요청 취소**, **스레드 전환** 등을 모두 수동으로 처리해야 했다. Combine에서는 이 모든 것이 **선언형 체인 하나**로 해결된다.

## 4) Combine의 핵심 이점들

### 가독성과 유지보수성
비동기 로직이 **하나의 파이프라인**으로 표현되어 전체 흐름을 한눈에 파악할 수 있다. 각 단계가 명확히 분리되어 있어 디버깅과 수정이 용이하다.

### 에러 처리의 일관성
```swift
publisher
    .retry(3)                          // 3회 재시도
    .catch { error in                  // 에러를 다른 Publisher로 대체
        Just(defaultValue)
    }
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished:
                print("성공적으로 완료")
            case .failure(let error):
                print("최종 실패: \(error)")
            }
        },
        receiveValue: { value in
            print("값 수신: \(value)")
        }
    )
```

## 주의사항 & Best Practice

### 메모리 관리
```swift
// ❌ 잘못된 방법: 강한 참조로 인한 메모리 누수
publisher
    .sink { value in
        self.handleValue(value) // self가 강하게 캡처됨
    }

// ✅ 올바른 방법: 약한 참조 사용
publisher
    .sink { [weak self] value in
        self?.handleValue(value)
    }
    .store(in: &cancellables)

// ✅ 더 나은 방법: assign 사용으로 누수 위험 제거
publisher
    .assign(to: &$property)
```

### 스레드 안전성

UI 업데이트는 반드시 메인 스레드에서 수행해야 한다:
```swift
networkPublisher
    .map { data in /* 백그라운드에서 파싱 */ }
    .receive(on: DispatchQueue.main)  // 필수!
    .sink { [weak self] result in
        self?.updateUI(with: result)
    }
```

### Publisher 캡슐화
```swift
// ✅ 외부에는 AnyPublisher로 타입 은닉
class DataService {
    private let subject = PassthroughSubject<Data, Error>()
    
    var dataPublisher: AnyPublisher<Data, Error> {
        subject.eraseToAnyPublisher()
    }
}
```

---

Combine은 단순한 프레임워크가 아니라 **사고방식의 전환**을 요구한다. 비동기 작업을 "무엇을 해야 하는가"의 명령 나열이 아닌, "데이터가 어떻게 흘러야 하는가"의 스트림 설계로 접근하는 것이다.

이러한 선언형 접근은 코드의 **가독성**, **테스트 가능성**, **유지보수성**을 크게 향상시킨다. 특히 복잡한 비동기 로직일수록 그 효과가 두드러진다.

## GQ
- 사용자 입력을 실시간으로 감지하고, 일정 시간 멈춘 후 서버에 요청을 보내려면 어떻게 구현할 수 있을까?

## Keywords
- [[Combine]]

#Onething 