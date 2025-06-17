>[!question]
>GQ1.  **Swift Standard Library란?**
>GQ2. .map 은 어떤 기능을 가지고 있는가?
>GQ3. reduce 는 어떤 기능을 가지고 있는가?
>GQ4. 대규모 배열 혹은 딕셔너리 데이터를 처리할 때 `map`, `filter`, `reduce`를 활용하여 성능과 메모리를 최적화하려면 어떻게 설계해야 할까?
## Description

GQ1.  **Swift Standard Library란?**
- GA 1: Swift Standard Library는 Swift 언어에 기본적으로 포함되어 있는 자료형, 연산자, 프로토콜, 함수 등 핵심 요소들의 집합이다.

자료형 / Int, Double, Bool, String / 기본 숫자, 문자 자료형
컬렉션 / Array, Dictionary, Set / 데이터 묶음을 저장하고 조작
제어문 관련 / Optional, if let, guard let / 값이 있을 수도 없을 수도 있는 경우 처리
프로토콜 / Equatable, Comparable, Codable 등 / 공통된 기능을 추상화해서 약속하는 인터페이스
함수형 도구 / map, filter, reduce / 배열 등 컬렉션을 함수로 처리
등

GQ2. .map 은 어떤 기능을 가지고 있는가?
* GA 2: .map 은 컬렉션의 모든 요소를 똑같은 방식으로 처리해서, 그 처리한 결과들을 가지고 새로운 배열을 만드는 기능이다.

```swift
let scores = [90, 80, 100]
let labels = scores.map { "점수는 \($0)점입니다" }

print(labels)
// ["점수는 90점입니다", "점수는 80점입니다", "점수는 100점입니다"]
```

```swift
let strNumbers = ["1", "2", "3"]
let intNumbers = strNumbers.map { Int($0)! }

print(intNumbers)
// [1, 2, 3]
```

```swift
struct Fruit {
    let name: String
    let price: Int
}

let basket = [
    Fruit(name: "사과", price: 1000),
    Fruit(name: "바나나", price: 800),
    Fruit(name: "딸기", price: 2000)
]

let fruitNames = basket.map { $0.name }

print(fruitNames)
// ["사과", "바나나", "딸기"]
```

이때, .map 은 타입을 바꿀 수도 있다.

GQ3. reduce 는 어떤 기능을 가지고 있는가?
* GA 3: reduce 는 컬렉션의 모든 요소를 하나의 값으로 “합쳐주는” 함수

1_ 합계 구하기
```swift
let numbers = [1, 2, 3, 4, 5]

let sum = numbers.reduce(0) { partialSum, next in
    partialSum + next
}

print(sum) // 15
```

1_ 합계 구하기 축약형
```swift
let numbers = [1, 2, 3, 4, 5]

let sum = numbers.reduce(0, +) // 0에서 시작해서 모두 더함

print(sum) // 15
```

2_ 문자열 이어 붙이기
```swift
let words = ["Swift", "는", "재미있어!"]

let sentence = words.reduce("") { $0 + " " + $1 }

print(sentence) // " Swift 는 재미있어!"
```

3_ 가장 큰 숫자 찾기
```swift
let numbers = [3, 7, 2, 9, 4]

let maxNumber = numbers.reduce(Int.min) { max($0, $1) }

print(maxNumber) // 9
```

GQ4. 대규모 배열 혹은 딕셔너리 데이터를 처리할 때 `map`, `filter`, `reduce`를 활용하여 성능과 메모리를 최적화하려면 어떻게 설계해야 할까?
* GA 4: lazy 사용하기, reduce는 계산할 때만 사용하기.

```swift
let numbers = Array(1...1_000_000)

let result = numbers
    .lazy
    .filter { $0 % 2 == 0 }
    .map { $0 * 3 }
    .prefix(5) // 결과 중 5개만 가져오기
```

```swift
let numbers = [1, 2, 3, 4, 5]
let sum = numbers.reduce(0) { $0 + $1 } // 15

let newArray = numbers.reduce([]) { $0 + [$1 * 2] }
//  데이터 축적은 map에게 맡기세요. reduce는 데이터 축적할 때는 느릴뿐 아니라 메모리 낭비.
//  그렇지만, 모든 숫자의 합, 가장 큰 수 이렇게 하나의 값을 뽑아낼때 reduce가 최고입니다.
```