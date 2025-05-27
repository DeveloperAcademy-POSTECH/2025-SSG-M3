> [!question] GQ1. where 절을 쓰지 않고도 제네릭 제약을 구현할 수 있을까?

## Description

- **제네릭이란 타입에 의존하지 않는 범용 코드를 작성할 때 사용한다. 제네릭을 사용하면 중복을 피하고, 코드를 유연하게 작성할 수 있음. 다양한 타입을 사용하고싶을 때 제네릭으로 선언하면 됨.**
- where 절은 **제네릭 코드에서 타입에 조건을 걸고 싶을 때** 사용됨. T가 이런 프로토콜을 따를 때만 이 함수나 타입을 쓰게 제약을 거는거임.

## 주요 기능

- 근데 제네릭에서 타입 제약을 걸 때, where만 쓸 수 있는 건 아님. Some 이라는 애를 사용 가능.
- <T: SomeProtocol> : 타입 매개변수가 **하나이거나**, 또는 각각 독립적인 제약일 때 효과적임. 만약 매개변수가 여러개고 제약이 복잡하다면 where을 써야함.

## 코드 예시

```swift
//제네릭 사용 예시

func wrap<T>(_ value: T) {
    print("상자에 담긴 값: \\(value)")
}

wrap(123)         // Int
wrap("hello")     // String
wrap(true)        // Bool
```

```swift
//where절을 사용한 제네릭 예시
func areTheyEqual<T>(_ a: T, _ b: T) -> Bool where T: Equatable {
    return a == b
}
```

```swift
// Some을 이용한 제약
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}
```

## Keywords

- where
- some
- generic

## References

- [https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics#Type-Constraints](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics#Type-Constraints)