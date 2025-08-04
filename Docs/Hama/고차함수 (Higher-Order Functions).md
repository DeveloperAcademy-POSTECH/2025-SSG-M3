>[!question]
>[!question]
>GQ1. sorted(by:) 함수란?
>GQ2. 클로저(Closure) 표현식을 사용하여 sorted 함수의 동작을 사용자 정의하는 방법은 무엇인가?

Swift의 sorted 함수는 기본적으로 오름차순으로 정렬하지만, 클로저를 활용하여 내림차순, 혹은 특정 프로퍼티를 기준으로 객체를 정렬하는 등 다양한 정렬 기준을 적용할 수 있습니다. 어떻게 가능한지 설명해주세요.

* GQ1. sorted(by:) 함수란?
: Swift의 배열 타입에서 제공하는 메서드로, 배열의 요소를 **정렬된 배열로 반환**한다.
sorted(by:)는 **정렬 기준을 결정하는 클로저**를 인자로 받는다.

정렬 기준을 정의할 때 사용할 클로저 표현식의 문법은 다음과 같다
```swift
{ (a: Int, b: Int) -> Bool in
    return a < b
}
```

또한, 클로저는 다양한 방식으로 쓸 수 있으며, 점점 더 간결하게 표현할 수 있다.

| 형태        | 예시                                           |
| --------- | -------------------------------------------- |
| 전체 형식     | { (a: Int, b: Int) -> Bool in return a < b } |
| 타입 생략     | { (a, b) in return a < b }                   |
| return 생략 | { a, b in a < b }                            |
| 축약 인자 사용  | { $0 < $1 }                                  |
| 연산자 함수 전달 | sorted(by: <)                                |
* GQ2. 클로저(Closure) 표현식을 사용하여 sorted 함수의 동작을 사용자 정의하는 방법은 무엇인가?
: sorted(by:) 함수에 **클로저를 전달해서**, 정렬 기준을 **직접 정하는 것**을 말한다.

기본 정렬방식
```swift
let numbers = [3, 1, 2]
let sortedDefault = numbers.sorted()
// 결과: [1, 2, 3] (기본정렬방식: 오름차순 정렬)
```
클로저 사용 예시
```swift
let numbers = [3, 1, 2]
let sortedDescending = numbers.sorted(by: { (a, b) in
    return a > b
})
// 결과: [3, 2, 1]
```
여기서 { (a, b) in return a > b }는 정렬 기준을 직접 정의한 클로저 표현식이다.
앞의 값이 뒤의 값보다 크면 true → 내림차순 정렬을 의미!

축약형을 활용하면, 더 간단하게도 표현할 수도 있다.
```swift
let numbers = [3, 1, 2]
let sortedDescending  = numbers.sorted(by: { $0 > $1 })
// 결과: [3, 2, 1]
```
여기서 $0, $1 - $0 첫 번째 값, $1 두 번째 값을 의미한다.

## Keywords
+ [[고차 함수 (Higher-Order Functions)]]

## 작성자
#Hama 