>[!question]
>[!question]
>GQ1. ARC 에서 unowned 참조의 역할은?
>GQ2. unowned 참조는 왜 nil을 허용하지 않는가?
>GQ3. 런타임에서 어떤 위험이 발생할 수 있으며, 이를 방지하기 위한 전략은 무엇인가?

## Description
- GQ1. ARC 에서 unowned 참조의 역할은?
	- GA1. Swift는 ARC(Automatic Reference Counting)를 통해 메모리를 관리한다.
	  ARC는 객체의 참조 카운트(reference count)를 자동으로 추적하여, 더 이상 참조하는 곳이 없으면 메모리를 해제한다.
	  그러나, 강한 참조 순환이 발생하면 문제가 생기는데, 객체 A가 객체 B를 강하게 참조하고, 동시에 B도 A를 강하게 참조하게 되면, 둘 다 상대를 참조하고 있기 때문에 어느 하나를 nil 로 설정해도 상대방이 남아있어 참조 카운트가 0이 되지 않아 메모리 누수가 발생한다.
	  이 문제를 해결하기 위해 swift 에서는 weak 참조와 unowned 참조를 제공한다.

- GQ2. unowned 참조는 왜 nil을 허용하지 않는가?
	- GA2. 어떤 객체 관계에서는 참조 대상이 항상 존재함을 보장해야 의미가 맞는 경우가 있다. 예를 들어, 신용카드 객체는 반드시 고객 객체에 소속되어 발급된다.
	  즉, 신용카드는 고객없이 발급이 될 수 없으므로, CreditCard 가 Customer 를 가리킬 때, nil 이 되어서는 안되는 것이다.
	  만약, weak 를 사용한다면, Customer 이 없을 때, nil 이 표시되는데, 논리적으로 "카드에 고객에 없다"는 상황은 일어나지 않아야하기에 unowned 가 더 적합하다는 것을 알 수 있다.

## 코드 예시
```swift
class Customer {
    let name: String
    var card: CreditCard?   // Customer -> CreditCard strong 참조
    init(name: String) {
        self.name = name
        print("\(name) initialized")
    }
    deinit {
        print("\(name) deinitialized")
    }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer   // CreditCard -> Customer unowned 참조
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
        print("Card #\(number) initialized")
    }
    deinit {
        print("Card #\(number) deinitialized")
    }
}

// 사용 예
var john: Customer? = Customer(name: "John")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)

// `john`을 nil로 설정하여 Customer 인스턴스 해제
john = nil

// Console 출력 예시:
// John deinitialized
// Card #1234567890123456 deinitialized
```

- GQ3. 런타임에서 어떤 위험이 발생할 수 있으며, 이를 방지하기 위한 전략은 무엇인가?
	- GA3. unowned 는 **“참조 대상 객체의 수명이, 반드시 자신보다 길거나 동일하다”** 는 전제가 있을 때만 안전합니다.
	  이 전제가 깨지면 앱 크래시가 발생되어 치명적입니다. 때문에 unowned 는 꼭 필요한 경우에만 신중히 사용해야 합니다.

## Keywords
+ [[ARC (Automatic Reference Counting)]]

## 작성자
#Hama 