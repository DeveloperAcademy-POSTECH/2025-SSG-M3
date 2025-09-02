>[!question]
>[!question]
>GQ1. Design Pattern 이 뭔가?
>GQ2. 왜 써야하는가?
>GQ3. Design Pattern 의 간단한 예시 코드

## Description
- GQ1. Design Pattern이 뭔가?
	GA. 여러 개발자가 반복해서 맞닥뜨린 문제를 해결하는 과정에서 발견된 **공통적인 모범 답안**을 정리한 것입니다. 각각의 패턴은 구체적인 코드 조각이라기보다는 **설계상의 청사진**에 가깝기 때문에 언어나 구현에 무관하게 응용될 수 있습니다.

* GQ2. 왜 써야하는가?
	GA.
	1. **코드 작성과 유지보수를 더 쉽게하고, 소프트웨어 설계의 품질을 높이기 위함이다.**
		패턴에는 선배 개발자들이 오랜 시행착오 끝에 검증한 해결 방법이 담겨 있으므로, 이를 사용하면 유사한 문제를 일일이 새로 해결하지 않고도 빠르게 개발을 진행할 수 있습니다 . 반복적으로 같은 문제를 해결해야 하는 수고를 덜어주어 개발 시간과 비용을 단축시킵니다.
	2. **의사소통과 협업을 쉽게 하기 위함이다.**
		일정한 설계 규칙을 따르기 때문에 개발자 개인뿐만 아니라 팀 전체가 코드를 더 이해하기 쉬워지고, 몇 년 뒤 코드를 보더라도 일관된 구조 덕분에 파악이 용이합니다. 디자인 패턴 자체가 개발자들 사이의 공통된 어휘로 기능하기도 합니다. 예를 들어 “이 부분에는 옵저버 패턴을 썼다”라고 하면 상세한 설명 없이도 다른 개발자들이 설계 의도를 이해할 수 있습니다.

## 코드 예시
+ GQ3. Design Pattern 의 간단한 예시 코드
	GA.
1. **Singleton 패턴**:
 싱글톤은 공통된 리소스(네트워크, 설정, 캐시 등)를 앱 전체에서 공유할 때 유용.
```swift
class NetworkManager {
    static let shared = NetworkManager()  // 유일한 인스턴스
    private init() {} // 외부에서 새로 생성 불가
    
    func requestData() {
        print("서버에 요청을 보냈습니다.")
    }
}

// 사용 예시
NetworkManager.shared.requestData()
```

2. **Observer 패턴**:
여러 곳에서 같은 알림 이름을 등록하면, post 한 번으로 모든 옵저버가 동시에 반응함.
```swift
// 옵저버 등록(여러 곳에서 가능)
let observer = NotificationCenter.default.addObserver(
    forName: .init("dataUpdated"),
    object: nil,
    queue: .main
) { notification in
    print("데이터가 업데이트되었습니다!")
}

// 옵저버에게 알림 보내기
NotificationCenter.default.post(name: .init("dataUpdated"), object: nil)
```

3. **Delegate 패턴**:
객체 간 1:1 통신.
```swift
protocol TaskDelegate: AnyObject {
    func taskDidFinish()
}

class Worker {
    weak var delegate: TaskDelegate?
    
    func doWork() {
        print("일하는 중")
        delegate?.taskDidFinish()
    }
}

class Manager: TaskDelegate {
    func taskDidFinish() {
        print("작업이 끝났습니다.")
    }
}

// 사용 예시
let worker = Worker()
let manager = Manager()
worker.delegate = manager
worker.doWork()
```

4. **Strategy 패턴**:
실행 중에도 전략(알고리즘)을 쉽게 교체 가능.
```swift
// 전략을 함수(클로저)로 정의
typealias PaymentStrategy = (Int) -> Void

let creditCardPayment: PaymentStrategy = { amount in
    print("\(amount)원을 카드로 결제합니다.")
}

let cashPayment: PaymentStrategy = { amount in
    print("\(amount)원을 현금으로 결제합니다.")
}

// Context
class PaymentProcessor {
    var strategy: PaymentStrategy
    
    init(strategy: @escaping PaymentStrategy) {
        self.strategy = strategy
    }
    
    func pay(amount: Int) {
        strategy(amount)
    }
}

// 사용 예시
let processor = PaymentProcessor(strategy: creditCardPayment)
processor.pay(amount: 10000)  // 카드 결제

processor.strategy = cashPayment
processor.pay(amount: 5000)   // 현금 결제
```
## Keywords
+ [[디자인 패턴 (Design Pattern)]]

## 작성자
- #Hama 