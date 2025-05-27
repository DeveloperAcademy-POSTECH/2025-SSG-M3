>[!question]
>GQ1. 제네릭이 코드 재사용성과 타입 안정성에 어떻게 기여할까?

## Description
| 제네릭의 이점 | 설명                                                |
| ------- | ------------------------------------------------- |
| 코드 재사용성 | 타입에 관계없이 동일한 로직을 재사용할 수 있어 코드 중복을 줄인다.            |
| 타입 안정성  | 컴파일 시점에 타입을 검사하므로, 런타임 오류를 방지하고 안전한 코드를 작성할 수 있다. |

## 재사용성 (Reusability)
- 타입마다 같은 로직을 중복해서 작성하면 코드가 불필요하게 길어지고, 유지보수가 어려워진다.
```swift
func printIntArray(_ array: [Int]) {
    for item in array {
        print(item)
    }
}

func printStringArray(_ array: [String]) {
    for item in array {
        print(item)
    }
}
```

```swift
func printArray<T>(_ array: [T]) {
    for item in array {
        print(item)
    }
}
```
- T에는 어떤 타입이든 허용된다.
- `printArray([1, 2, 3])`, `printArray(["A", "B", "C"])`와 같이 호출할 수 있다.

## 타입 안정성 (Type Safety)
- 제네릭이 없을 때 `Any`를 사용해 유연하게 코딩할 수 있지만, 컴파일러가 타입 오류를 잡아내지 못하고 런타임 오류가 날 수 있다.
```swift
func printAnyArray(_ array: [Any]) {
    for item in array {
        if let intValue = item as? Int {
            print(intValue * 2)
        }
    }
}

printAnyArray(["Hello", 3, true]) // 런타임 오류 위험
```
- 제네릭은 컴파일 타임에 타입이 결정되므로, 타입 오류를 미리 잡을 수 있다.
```swift
func doubleValues<T: Numeric>(_ array: [T]) -> [T] {
    return array.map { $0 + $0 }
}
```
- `T: Numeric`으로 숫자 타입만 허용 할 수 있다.
- 만약 `doubleValues(["A", "B"])`와 같이 잘못된 호출을 하면 컴파일 시점에 에러가 발생한다.

## Keywords
+ [[제네릭 (Generic)]]

## References
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/


#Air