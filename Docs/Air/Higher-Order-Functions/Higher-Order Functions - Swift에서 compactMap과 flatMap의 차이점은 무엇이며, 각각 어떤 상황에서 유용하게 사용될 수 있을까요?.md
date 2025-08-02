>[!question]
>GQ. Swift에서 compactMap과 flatMap의 차이점은 무엇이며, 각각 어떤 상황에서 유용하게 사용될 수 있을까요?

## Description
- `compactMap`과 `flatMap`은 Swift의 컬렉션을 다루는 고차 함수이다. 두 함수 모두 컬렉션의 요소를 변환하지만, 핵심 목적과 동작 방식이 다르다. 
- `compactMap`의 주요 역할은 압축(Compacting)이다. 변환 과정에서 `nil`이 발생할 경우, 이를 결과에서 완전히 제거하고 유효한 값만으로 구성된 컬렉션을 만든다.
- `flatMap`의 주요 역할은 평탄화(Flattening)이다. 변환 결과가 또 다른 컬렉션일 때, 중첩된 구조를 한 단계 해제하여 단일 레벨의 컬렉션으로 만든다. 

## 주요 기능
- `flatMap`: 중첩된 컬렉션을 평탄화한다.
    - `[[1, 2], [3, 4]]`와 같은 2차원 배열을 `[1, 2, 3, 4]`와 같은 1차원 배열로 변환하는 데 사용된다. 
    - `s.flatMap(transform)`은 `Array(s.map(transform).joined())`와 동일하게 동작한다. 
- `compactMap`: 변환 결과의 `nil`을 제거한다.
    - `["1", "two", "3"]`을 `Int`로 변환할 때, 실패하는 "two"(`nil` 반환)를 제외하고 `[1, 3]`을 결과로 얻는 데 사용된다. 
    - 타입의 배열에서 `nil`을 제거하여 타입의 배열을 얻을 때도 유용하다.
- `flatMap`은 중첩된 배열을 하나로 합칠 때, `compactMap`은 배열에서 `nil`을 제거하고 싶을 때 유용하다.

## 코드 예시
##### `flatMap`이 유용한 경우
- 여러 개의 배열을 하나의 배열로 합치고 싶을 때 주로 사용된다. 예를 들어, 각 작가(`Author`)가 여러 권의 책(`[Book]`)을 가지고 있을 때, 모든 작가의 책 목록을 하나의 통합된 책 목록(`[Book]`)으로 만들고자 할 경우 `flatMap`을 사용하여 처리할 수 있다.
- 2차원 배열처럼 **중첩된 배열**을 하나의 배열로 평탄화(flatten)하는 데 아주 효과적이다.
```Swift
struct Author {
    let name: String
    let books: [String]
}

let authors = [
    Author(name: "Kim", books: ["Swift Programming", "iOS Apps"]),
    Author(name: "Lee", books: ["Understanding closures"]),
    Author(name: "Park", books: ["Intro to Combine", "Advanced Swift"])
]

// 각 작가의 책 배열들을 하나의 배열로 합친다.
let allBooks = authors.flatMap { $0.books }

// 결과: ["Swift Programming", "iOS Apps", "Understanding closures", "Intro to Combine", "Advanced Swift"]
print(allBooks)
```
##### `compactMap`이 유용한 경우
- 배열의 `nil` 값을 제거하거나, 타입 변환에 실패한 요소를 버리고 싶을 때 아주 유용하다. 예를 들어, 문자열 배열을 정수 배열로 바꾸려고 할 때, 숫자로 변환할 수 없는 문자열은 `nil`이 된다. 이때 `compactMap`을 사용하면 변환에 성공한 값만 남길 수 있다.
- 변환 과정에서 생기는 `nil`을 안전하게 제거하고 유효한 값만 추출하는 역할.
```Swift
let stringInputs = ["10", "20", "hello", "30", "world", "40"]

// 문자열을 Int로 변환하되, 변환에 실패하면(nil) 결과에서 제외한다.
let validNumbers = stringInputs.compactMap { Int($0) }

// 결과: [10, 20, 30, 40]
print(validNumbers)
```

## Keywords
+ [[고차 함수 (Higher-Order Function)]]
+ [[compactMap]]
+ [[flatMap]]

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- 작성한 사람의 닉네임 (예: # Air )