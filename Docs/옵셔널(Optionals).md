>[!question]
>GQ1. 옵셔널이란 무엇인가?
>GQ2. 옵셔널을 왜 사용할까?
>GQ3. 옵셔널 바인딩과 강제 언래핑의 차이는?
>GQ4. guard let vs. if let
>GQ5. 옵셔널 체이닝과 nil 병합 연산자는 어떻게 동작할까?
>GQ6. 옵셔널과 관련된 런타임 오류를 어떻게 방지할 수 있을까?

## Description

"A type that represents either a wrapped value or the absence of a value."
값이 있을 수도 있고 없을 수도 있음을 명시적으로 표현하는 Swift의 특수한 열거형(Enum) 타입.


## 주요 기능
+ 변수에 값이 없을 수 있음을 안전하게 표현
+ 런타임 오류를 방지하기 위한 안전한 값 처리
+ nil-check로 로직을 간결히 작성
+ 옵셔널 체이닝으로 복잡한 nil 검사 단순화
+ nil 병합 연산자를 통한 기본값 설정

## 코드 예시

```swift
// 1. 옵셔널 선언
var name: String? = "Noter"
var age: Int? = nil

// 2. 옵셔널 바인딩 (if let)
if let unwrappedName = name {
    print("Name is \(unwrappedName)")
} else {
    print("Name is nil")
}

// 3. 옵셔널 바인딩 (guard let)
func greet(_ name: String?) {
    guard let unwrappedName = name else {
        print("No name provided")
        return
    }
    print("Hello, \(unwrappedName)!")
}

// 4. 강제 언래핑 (위험, 값이 nil일 경우 크래시 발생)
print(name!) // "Noter"

// 5. 옵셔널 체이닝
struct Person {
    var address: Address?
}

struct Address {
    var city: String
}

let person = Person(address: Address(city: "Pohang"))
let cityName = person.address?.city  // "Pohang"

// 6. nil 병합 연산자
let finalName = name ?? "Unknown" // "Noter"
```

## Keywords
+ Optional
+ Optional Binding
+ Forced Unwrapping
+ guard let
+ if let
+ Optional chaining
+ Nil-Coalescing Operator (??)
+ Implicitly Unwrapped Optional (! after type)

## References
- SwiftUI | Apple Developer Documentation - Optional (https://developer.apple.com/documentation/swift/optional/)
- 