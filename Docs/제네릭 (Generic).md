>[!question]
>GQ1. 스위프트에서 제네릭을 사용하는 이유는 무엇일까?
>GQ2. 제네릭 타입은 구조체, 클래스, 열거형에서 어떻게 사용되는가?
>GQ3. 여러 개의 타입을 사용할 때는 어떻게 표시하는가?
>GQ4. 프로토콜 제약, 클래스 제약이 정확히 무엇인가? 예시는?


## Description
스위프트는 타입에 민감한 언어이지만, 제네릭을 사용하면 타입에 의존하지 않고 코드를 유연하게 작성할 수 있다.
```
func addTwoNumbers(_ a: Int, _ b: Int) -> Int {
	return a + b
}
```
이 경우 `addTwoNumbers(3, 2)`로 두 `Int`를 더하는 것은 가능하지만, `addTwoNumbers(3.5, 2.0)`과 같이 `Double`을 넣는 것은 불가능하다. 이를 위해서는 `Double`을 위한 함수를 또 만들어야 한다.

그러나 아래와 같이 제네릭을 쓰면 범용적으로 코드를 짜는 것이 가능하다.
```
func addTwoNumbers<T>(_ a: T, _ b: T) -> T {
	return a + b
}
```
여기서 `<T>`에 적힌 T는 사용자가 임의로 설정하며, 제네릭 함수 내에서만 작동한다.

## 주요 기능
+ 타입에 유연한 범용적인 코드 작성

## 코드 예시
```
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
	let temporaryA = a a = b b = temporaryA
}
```

## Keywords
+ 제네릭 함수
+ 프로토콜 제약
+ 클래스 제약
+ 역제네릭(Inverse Generics)
+ Associated Type
+ Opaque Type(`some`)


## References
- [Apple 공식 문서 - Generic](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/)
- [Swift) 제네릭(Generic) 정복하기](https://babbab2.tistory.com/136)
