>[!question]
>GQ1. Sequence와 Collection의 차이점은 무엇이고, 각각의 사용 사례는 어떤 것이 있는가?

## Description
- Swift Standard Library는 Swift 프로그래밍 언어의 핵심 기능들을 제공하는 기본 도구 상자임. 이거 없으면 print문도 사용할 수 없고 하나하나 구현해야함
- Sequence와 Collection 둘 다 Swift에서 반복 가능한 타입들을 정의하는 프로토콜(요소들의 집합을 다룸)
- Sequence는 한번만 순회 가능하고 인덱스 기반 접근이 불가하다. 그에 비해 Collection은 여러 번 순회 가능하고 인덱스 기반 접근이 가능하다.  Sequence를 상속받은게 Collection이다.
- 

## 주요 기능
+ ###  Sequence
- 한 번만 순회하고 끝나는 데이터 흐름이다. 
- Sequence는 반드시 여러 번 순회 가능한 걸 보장하지 않음.
- 예: 파일 읽기, 네트워크 스트림, 입력 스트림처럼 **한 번만 읽고 처리해야 하는 데이터**에 적합함
- `Sequence`는 무한 시퀀스를 만들고 일부만 꺼내 쓸 때 유용함

- ###  Collection
- 요소 개수와 인덱스가 필요한 일반적인 경우에 사용함
-  배열, 딕셔너리, 셋 등 거의 모든 기본 자료형
- Collection은 전체 개수를 알아야 하므로 무한 시퀀스를 만들 수 없음.
- `count`, `isEmpty`는 `Collection`에서만 사용 가능.

|상황|적합한 타입|
|---|---|
|무한하게 생성되는 값 다루기|`Sequence`|
|한 번만 처리되는 데이터 스트림|`Sequence`|
|UI에 나열할 리스트 데이터|`Collection`|
|인덱스를 이용해 값에 접근할 필요가 있을 때|`Collection`|
|데이터의 개수나 범위를 알고 싶을 때|`Collection`|
|`forEach`, `map`, `filter`, `reduce` 사용|둘 다 가능하지만 `Collection`이 더 일반적임|

## 코드 예시
- Sequence 예시 
```swift
let input = sequence(first: 1) { $0 < 10 ? $0 + 1 : nil }
for num in input {
    print(num) // 1~10 출력
}
// 여기서 다시 반복문을 돌리면 동작하지 않음

```
- Collection 예시 
```swift
var names = ["joy", "sera", "mingky"]
names[1] = "gabi"
print(names[1]) // "gabi"

```
## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
#Sera 