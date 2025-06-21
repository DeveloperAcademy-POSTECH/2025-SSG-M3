>[!question]
>GQ1. CaseIterable 가 뭘까?
>GQ2. CaseIterable 은 언제 유용할까?
>GQ3. CaseIterable 은 어떤 한계가 있을까?
## Description
* GQ1. CaseIterable 가 뭘까?
* GA1. CaseIterable은 열거형(enum) 타입이 가지고 있는 모든 case 를 자동으로 모아주는 기능으로, 모든 case 들을 '배열'처럼 .allCases 로 가져올 수 있게 해주는 프로토콜이다.
## 주요 기능
+ GQ2. CaseIterable 은 언제 유용할까?
+ GA2. 예를 들어, 방향을 나타내는 enum 있다고 하면, CaseIterable 을 채택하면 자동으로 쓸 수 있게 된다.
```swift
enum CompassDirection: CaseIterable {
    case north, south, east, west
}

print(CompassDirection.allCases) 
// 출력: [north, south, east, west]

print("There are \(CompassDirection.allCases.count) directions.")
// 출력: There are 4 directions.

let caseList = CompassDirection.allCases
	.map({ "\($0)" }) // 각 case 를 문자열로 바꿈
    .joined(separator: ", ") // 쉼표로 연결

print(caseList)
// 출력: "north, south, east, west"

```
* GQ2-1. case north, south, east, west 를 내가 직접 써야되는데, 왜 '자동 으로 쓸 수 있다' 표현할까?
* GA2-1. case 목록을 직접 쓴건 맞다. 그러나, CaseIterable 을 체택하는 순간, 다음과 같은 코드를 '자동'으로 생성해주어, .allCases 로 접근할 수 있게 한다.
```swift
static var allCases: [CompassDirection] {
    return [.north, .south, .east, .west]
}
```
## 한계
+ GQ3. CaseIterable 은 어떤 한계가 있을까?
+ GA3-A. 연관값(Associated Values)이 있는 enum은 자동 지원하지 않는다.
```swift
enum Animal: CaseIterable {
    case cat(name: String) // Error
}
```
* GA3-B. RawValue(enum에 기본값)이 있어도 .allCases는 rawValue가 아니다.
```swift
enum Color: String, CaseIterable {
    case red = "R"
    case green = "G"
    case blue = "B"
}

print(Color.allCases)
// 출력: [red, green, blue]
```
* GA3-C. 이름, 속성, 분류 등을 enum에 직접 못 넣는다.
```swift
enum Fruit: CaseIterable {
    case apple, banana, grape

    // 이름을 붙이고 싶다면 밑 코드처럼 직접해야한다.
    var displayName: String {
        switch self {
        case .apple: return "사과"
        case .banana: return "바나나"
        case .grape: return "포도"
        }
    }
}

let names = Fruit.allCases.map { $0.displayName }
print(names)
// 출력: ["사과", "바나나", "포도"]
```
## Keywords
* enum
+ caseIterable
* .allCases
* .map
* .joined
* switch

## References
- https://developer.apple.com/documentation/swift/caseiterable/ (Apple의 공식 문서)

## 작성자
* Hama