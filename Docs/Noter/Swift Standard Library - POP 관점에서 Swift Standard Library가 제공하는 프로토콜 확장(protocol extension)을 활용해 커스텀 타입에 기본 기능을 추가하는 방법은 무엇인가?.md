

>[!question]
>GQ. Protocol Oriented Programming 관점에서 Swift Standard Library가 제공하는 프로토콜 확장(protocol extension)을 활용해 커스텀 타입에 기본 기능을 추가하는 방법은 무엇인가?

### 0. 키워드 개념 정리

1. Swift Standard Library
- Swift에 내장된 핵심 기능 모음.
- Array, Dictionary, Set, String 등 자료구조 및 Equatable, Hashable, Codable, Sequence 등의 **기본 프로토콜과 타입들**을 포함한다.

2. Protocol Extension
- Swift는 protocol 자체에 기본 동작을 붙일 수 있는 **extension 문법**을 지원한다.
- 덕분에 모든 conforming 타입\*에 대해 **기본 구현(default implementation)**을 제공 가능.
- 클래스 상속보다 더 가볍고 유연하게 "공통 기능"을 구성할 수 있다.

<span style="color:gray">  \* conforming 타입: 어떤 프로토콜을 따르는(채택한) 타입 <\span>

### 1. 개요

Swift는 객체 지향보다 **프로토콜 지향(POP: Protocol-Oriented Programming)**을 더 강조하는 언어다.  
이 철학은 Swift Standard Library 곳곳에서 잘 드러나는데, 대표적인 방식이 바로 "프로토콜에 기능을 붙이는 것"이다.

즉 프로토콜을 정의하고 extension으로 공통 동작을 미리 붙여두면
커스텀 타입에서 그 프로토콜을 채택함으로써 **기능을 따로 구현하지 않아도 기본 동작을 바로 활용**할 수 있다.

이렇게 하면 공통 코드가 흩어지지 않고, 타입에 따라 필요한 기능만 조합하듯 설계할 수 있다.

### 2. 왜 중요할까?

**2.1 상속보다 더 유연한 기능 조합**

- 클래스를 상속해서 기능을 확장하는 방식은 계층적이고 무거움
    
- 프로토콜 + extension은 "필요한 기능만 조합" 가능 -> **조합 기반 설계**
    

**2.2 공통 로직을 한 곳에 정의 가능**

- 다양한 타입에 적용 가능한 공통 동작을 한 곳에서 정의 → 중복 방지
    

**2.3 커스텀 타입을 더 쉽게 확장할 수 있음**

- struct, enum 등 값 타입도 기능 확장이 가능해짐
    

### 3. 예제 코드

**3.1 Swift Standard Library에서 확장된 기능 예시**

```swift
extension Collection {
    var isNotEmpty: Bool {
        return !isEmpty
    }
}
```

`Array`, `Set`, `Dictionary` 등 `Collection` 프로토콜을 따르는 모든 타입은 이제 `.isNotEmpty`를 바로 사용할 수 있음.

```swift
let arr = [1, 2, 3]
print(arr.isNotEmpty) // true
```

**3.2 직접 커스텀 타입에 적용**

```swift
protocol Describable {
    var title: String { get } // 읽기 전용 속성
    // “이 프로토콜을 따르는 타입은 title이라는 속성(property)을 가져야 한다
	// 그 속성은 반드시 읽을 수 있어야 한다(get 가능)*
	// 쓸 수 있는지는 상관 없음(set은 선택사항)”
}

extension Describable { // 프로토콜 Describable에 기본 기능 추가
    func describe() {
        print("This is \(title)")
    }
    // describe()는 구현하지 않아도 자동으로 사용 가능
}

struct Book: Describable {
    var title: String // 약속 지킴
}

let book = Book(title: "SwiftUI Basics")
book.describe() // 출력: This is SwiftUI Basics
```

→ `Book`은 `describe()`를 직접 구현하지 않았지만, 기본 구현 덕분에 바로 사용할 수 있다.

### 4. 언제 쓰면 좋은가?

- 특정 역할(동작)을 프로토콜로 정의하고 그에 대한 기본 동작을 제공하고 싶을 때
    
- 값 타입(구조체, 열거형)에서 공통 기능을 공유하고 싶을 때
    
- 유연한 기능 조합이 필요할 때
    

### 5. 정리

Swift의 Standard Library는 POP 철학을 바탕으로 설계되어 있으며,  
**프로토콜 + extension** 조합을 통해 공통 기능을 여러 타입에 쉽게 확장할 수 있다.

Swift 프로그래밍 시 커스텀 타입을 설계할 때도 이 방식을 적극 활용하면,

- 더 재사용성 높은 코드
    
- 더 안전한 타입 기반 설계
    
- 더 유연한 기능 조합  
    을 모두 갖출 수 있다.
    

---

## Keyword

- [[Swift Standard Library]]
- [[프로토콜 지향 프로그래밍 (POP)]]
    

#Noter