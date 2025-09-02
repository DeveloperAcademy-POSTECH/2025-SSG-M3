>[!question]
>GQ. 팩토리 패턴은 어떤 장점을 가지며 어떻게 적용할 수 있을까?

## Description
- 팩토리 패턴(Factory Pattern)은 객체 생성을 직접 생성하는 대신, 객체 생성을 전담하는 별도의 Factory 클래스를 만들어 객체를 생성하는 디자인 패턴이다.
- 이 패턴을 사용하면 객체를 만드는 복잡한 과정을 숨기고, 생성 코드를 한 곳에서 관리하여 유지보수와 유연성을 높일 수 있다.

## 주요 기능
### 팩토리 패턴을 사용하면 좋은 상황
+ **다양한 타입의 객체 생성**: 비슷한 속성을 공유하지만 구체적인 타입이 다른 여러 객체를 만들어야 할 때
+ **생성 로직의 캡슐화**: 객체 생성 과정이 복잡할 때, 이 로직을 클라이언트 코드로부터 분리하고 싶을 때
+ **결합도 감소**: 객체를 사용하는 코드와 실제 생성되는 구체적인 클래스 간의 의존성을 줄이고 싶을 때

#### 팩토리 패턴 적용 시 유의할 점
- **코드 복잡성 증가**: 단순히 몇 종류의 객체만 생성하는 상황에서는 팩토리 패턴을 도입하는 것이 오히려 불필요한 클래스를 늘리고 코드를 복잡하게 만들 수 있다.


## 코드 예시
### 예시 1: 결제 수단 팩토리
#### 1. 프로토콜과 실제 객체들
모든 결제 수단이 가져야 할 공통 기능(`pay`)을 정의하고, 실제 결제 수간 객체들을 만든다.
```Swift
// 모든 결제 수단이 따라야 할 설계도 (Protocol)
protocol PaymentMethod {
	func pay(amount: Int)
}

// 설계도를 따른 실제 객체들 (Concrete Products)
struct CreditCard: PaymentMethod {
	func pay(amount: Int) {
		print("\(amount)원을 신용카드로 결제합니다.")
	}
}

struct KakaoPay: PaymentMethod {
	func pay(amount: Int) {
		print("\(amount)원을 카카오페이로 결제합니다.")
	}
}
```

#### 2. 결제 수단 공장
결제 타입을 나타내는 열거형(`PaymentType`)을 받아서 그에 맞는 결제 객체를 생성해 반환한다.
```Swift
// 어떤 결제 수단을 만들지 나타내는 타입
enum PaymentType {
    case creditCard
    case kakaoPay
}

// 결제 객체를 찍어내는 공장 (Factory)
enum PaymentFactory {
    static func createPayment(type: PaymentType) -> PaymentMethod {
        switch type {
        case .creditCard:
            return CreditCard()
        case .kakaoPay:
            return KakaoPay()
        }
    }
}
```

#### 3. 사용법
결제 로직에서는 사용자가 어떤 방식을 선택했는지 `PaymentType`으로 공장에 알려주기만 하면 된다.
```Swift
// 사용자가 '신용카드'를 선택한 상황
let userChoice1: PaymentType = .creditCard
let payment1 = PaymentFactory.createPayment(type: userChoice1)
payment1.pay(amount: 10000) // 출력: 10000원을 신용카드로 결제합니다.

// 사용자가 '카카오페이'를 선택한 상황
let userChoice2: PaymentType = .kakaoPay
let payment2 = PaymentFactory.createPayment(type: userChoice2)
payment2.pay(amount: 5000) // 출력: 5000원을 카카오페이로 결제합니다.
```


### 예시 2: 상태 아이콘 뷰 팩토리
메시지의 상태(`success`, `warning`, `error`)에 따라 적절한 아이콘을 보여주는 뷰 팩토리
#### 1. 상태 정의 및 뷰 팩토리
아이콘을 결정할 상태 값(`Status`)을 정의하고, 이 상태에 따라 다른 아이콘(`Image`) 뷰를 반환하는 팩토리를 만든다.
```Swift
import SwiftUI

// 메시지 상태를 나타내는 열거형
enum Status {
    case success
    case warning
    case error
}

// 상태에 맞는 아이콘 뷰를 만들어주는 공장
struct IconFactory {
    @ViewBuilder
    static func createIcon(for status: Status) -> some View {
        switch status {
        case .success:
            Image(systemName: "checkmark.circle.fill")
                .foregroundColor(.green)
        case .warning:
            Image(systemName: "exclamationmark.triangle.fill")
                .foregroundColor(.orange)
        case .error:
            Image(systemName: "xmark.circle.fill")
                .foregroundColor(.red)
        }
    }
}
```

#### 2. 사용법
`ContentView`에서는 현재 상태만 관리하고, 아이콘이 필요한 부분에서 팩토리를 호출하기만 하면 된다.
```Swift
struct ContentView: View {
    // 현재 상태를 @State로 관리
    @State private var currentStatus: Status = .success

    var body: some View {
        VStack(spacing: 20) {
            HStack {
                // 팩토리에 현재 상태를 알려주고 아이콘을 받아옴
                IconFactory.createIcon(for: currentStatus)
                    .font(.largeTitle)
                
                Text("메시지가 여기에 표시됩니다.")
            }
        }
    }
}
```


## Keywords
+ [[디자인 패턴 (Design Pattern)]]
+ [[팩토리 패턴 (Factory Pattern)]]

## References
- [Refactoring Guru - Factory Method Pattern (한국어)]([https://refactoring.guru/ko/design-patterns/factory-method](https://refactoring.guru/ko/design-patterns/factory-method))

## 작성자
- #Air 