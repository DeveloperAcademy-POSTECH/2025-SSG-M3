> [!question]
> POP이 Testability(테스트 용이성)에 어떻게 기여할 수 있을까?

---
## TL;DR

Swift에서 POP을 쓰면 테스트하기가 훨씬 쉬워진다.  
이유는 간단하다. **의존성을 프로토콜로 추상화해서 바꿔 끼우기 쉬워지기 때문.**  
결과적으로 → Mock을 만들기 편하고 → 유닛 테스트도 깔끔하게 설계할 수 있다.

> 의존성 분리를 쉽게 만들어 테스트 가능한 구조로 바꿔줌

---

## 1. 테스트 가능한 구조로 바뀌는 이유

POP을 쓰면 먼저 "프로토콜로 먼저 설계"하게 된다.  
이 구조는 자연스럽게 **구현(implementation)과 기능의 계약(contract)을 분리시킨다.

```swift
protocol PaymentProcessor {
    func pay(amount: Int)
}

class RealPaymentProcessor: PaymentProcessor {
    func pay(amount: Int) {
        // 실제 결제 로직
    }
}
```

이 구조에서는 PaymentProcessor라는 추상화만 알고 있으면,
진짜 결제 로직이 아니더라도 테스트를 진행할 수 있다.

---

## **2. Mock이 정말 간편해진다**

아래처럼 테스트용 객체를 쉽게 만들 수 있다.
```Swift
class MockPaymentProcessor: PaymentProcessor {
    var didCallPay = false
    func pay(amount: Int) {
        didCallPay = true
    }
}
```

테스트 코드
```Swift
func testPayment() {
    let mock = MockPaymentProcessor()
    let viewModel = CheckoutViewModel(paymentProcessor: mock)
    
    viewModel.pay()

    assert(mock.didCallPay == true)
}
```

실제 결제 API를 호출하지 않고도,
결제 함수가 **불렸는지 안 불렸는지** 정확하게 확인 가능하다.
→ 빠르고, 안전하고, 네트워크 끊길 걱정도 없다.

---

## **3. 유닛 테스트의 대상이 명확해진다**

POP은 책임을 쪼개는 데 유리하다.
기능을 작은 단위로 나누고 각각 프로토콜로 추상화하면,
“이 모듈은 이 일만 한다”는 경계가 뚜렷해진다.

→ 테스트할 때도 단위가 작고, 명확하고, 복잡도가 낮아진다.

---

## **정리**
- POP은 구조상 **DI(의존성 주입)과 궁합이 잘 맞는다
- 덕분에 실제 구현을 교체하기 쉬워지고, 테스트 코드 작성도 수월해진다
- 테스트의 목적은 “이 기능이 잘 돌아가는가”를 확인하는 것인데,
  POP을 쓰면 그걸 방해하는 요소(네트워크, DB 등)를 쉽게 떼어낼 수 있다


---

## GQ
-  **POP - 왜 DI(의존성 주입)과 궁합이 좋을까?**

## Keyword
- [[POP - 왜 DI와 궁합이 좋을까?|Dependency Injection(DI)]]