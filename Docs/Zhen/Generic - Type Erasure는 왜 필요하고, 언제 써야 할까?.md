>[!question]
>GQ1. Type Erasure는 왜 필요하고, 언제 써야 할까?

## Description
-**Type Erasure(타입 소거)**는 Swift에서 **제네릭 타입이나 associatedtype을 사용하는 프로토콜**이  
일반 타입처럼 다뤄질 수 없을 때, 이를 **우회해서 추상화하는 기법**이다.  
즉, 다양한 타입을 같은 인터페이스로 **묶어서 처리하고 싶을 때 사용하는 기법**이다.

- `associatedtype` 또는 `Self`가 포함된 프로토콜은 그 자체로 변수의 타입이 될 수 없다.
- 이런 제한을 해결하고 **다형성**을 구현하기 위해 Type Erasure를 사용함. 

## 주요 기능
### ### Type Erasure란?

- "타입을 숨긴다(지운다)"는 의미로, 실제 제네릭 타입이나 구체 타입을 감추고,  
    인터페이스(프로토콜)의 기능만 노출한다.
- Swift의 `some` 키워드와 `AnyView`, `AnySequence`, `AnyPublisher` 등이 대표적 예시.

---

## 왜 필요할까?

|상황|이유|
|---|---|
|`associatedtype`이 있는 프로토콜을 변수 타입으로 쓰고 싶을 때|컴파일러는 구체 타입이 뭔지 알아야 하기 때문|
|서로 다른 제네릭 타입을 한 컨테이너에 담고 싶을 때|`let data: [SomeGeneric]`처럼 직접 못 담는다|
|SwiftUI 등에서 `some View`의 다양한 구현을 하나로 추상화할 때|View마다 타입이 다르므로 통일 필요|

---

## 언제 사용할까?

- `associatedtype` 또는 `Self`가 포함된 **프로토콜을 변수, 배열, 리턴 타입 등으로 사용할 때**
- **다형성(polymorphism)**이 필요할 때: 서로 다른 타입을 하나로 다루고 싶을 때
- SwiftUI에서 `body` 리턴이 `some View`인데 조건문마다 다른 View를 쓸 때 (`AnyView` 사용)
- Combine에서 `AnyPublisher`처럼 추상화된 스트림으로 다루고 싶을 때


## 코드 예시

```
protocol Animal {
    func sound() -> String
}

struct Dog: Animal {
    func sound() -> String { "멍멍" }
}

struct Cat: Animal {
    func sound() -> String { "야옹" }
}

// Type Erasure 구조체
struct AnyAnimal: Animal {
    private let _sound: () -> String

    init<A: Animal>(_ base: A) {
        _sound = base.sound
    }

    func sound() -> String {
        _sound()
    }
}

let animals: [AnyAnimal] = [AnyAnimal(Dog()), AnyAnimal(Cat())]
for animal in animals {
    print(animal.sound()) // 멍멍, 야옹
}

```


## Keywords
+ [[Docs/Common/Keyword/프로토콜 지향 프로그래밍 (POP)]]
- [[제네릭 (Generic)]]
## References
- Apple 공식 문서: Protocols - Swift Language Guide
- [WWDC 2015 - Protocol-Oriented Programming in Swift](https://developer.apple.com/videos/play/wwdc2015/408/)
- [Swift Evolution Proposal: Type Erasure](https://github.com/apple/swift-evolution/blob/main/proposals/0309-unseal-anyhashable.md)
- Swift Forums: Type Erasure Patterns

#Zhen