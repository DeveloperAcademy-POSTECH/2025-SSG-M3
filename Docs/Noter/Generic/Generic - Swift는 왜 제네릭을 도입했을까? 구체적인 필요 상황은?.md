
>[!question]
>GQ1. Swift는 왜 제네릭을 도입했을까? 구체적인 필요 상황은?

### 1. 개요
제네릭(Generic)은 다양한 타입에서 **재사용 가능한 코드를 작성하기 위해** 도입된 개념이다. 스위프트는 타입 안정성을 중시하는 언어인데, 제네릭을 통해 타입 안정성을 해치지 않으면서도 유연한 추상화를 제공하고자 한다.

즉, **한 번의 정의로 다양한 타입에 대응하는 코드를** 작성할 수 있게 해준다. 중복을 줄이고, 형변환 오류 없이 안전한 코드를 만들 수 있다.

### 2. 제네릭이 필요한 대표적인 상황

**2.1 같은 기능인데 타입만 다른 경우**
```swift
func swapInts(_ a: inout Int, _b: inout Int){
	let temp = a
	a = b
	b = temp
}

func swapStrings(_ a: inout String, _ b: inout Stiring){
	let temp = a
	a = b
	b = temp
}
```

이렇게 타입마다 일일이 정의하는 대신

```swift
func swapValues<T>(_ a: inout T, _ b: inout T){
	let temp = a
	a = b
	b = temp
}
```

제네릭 하나로 모든 타입에 대해 재사용할 수 있다.


**2.2 컬렉션 타입 설계 시 스위프트의 Array, Dictionary, Set 전부 제네릭 기반이다.**

```swift
var numbers: [Int] = [1, 2, 3]
var names: [String] = ["A", "B"]
```

Array<T> 처럼 어떤 타입이든 담을 수 있는 컬렉션을 만들기 위해서는 제네릭이 필수다.


**2.3 재사용 가능한 알고리즘 구현**
```swift
func allMatch<T: Equatable>(_ list: [T], _ value: T) -> Bool {
	for item in list {
		if item != value {
			return false
		}
	}
	return true
}
```
어떤 타입이든 Equatable만 채택하고 있으면 동작하는 범용 비교 함수를 만들 수 있다.

### 3. 제네릭의 이점

|이점|설명|
|---|---|
|코드 중복 제거|타입별로 같은 함수를 반복 작성할 필요가 없다|
|타입 안전성|타입이 명확히 고정되므로 런타임 오류가 줄어든다|
|유연성|하나의 코드로 다양한 타입을 다룰 수 있다|
|추상화|타입을 추상화해서 더 일반적인 형태로 표현 가능|

### 4. 예제 코드

**4.1 제네릭 함수 예시**
```swift
func repeatElement<T>(_ element: T, count: Int) -> [T] {
    var result: [T] = []
    for _ in 0.. < count {
        result.append(element)
    }
    return result
}
```
어떤 타입이든 전달받아 `[T]` 형태로 반환 가능

**4.2 제네릭 타입 예시**
```swift
struct Stack<Element> {
    private var items: [Element] = []

    mutating func push(_ item: Element) {
        items.append(item)
    }

    mutating func pop() -> Element? {
        return items.popLast()
    }
}

var intStack = Stack<Int>()
intStack.push(1)
intStack.push(2)
print(intStack.pop()!) // 2

var stringStack = Stack<String>()
stringStack.push("A")
print(stringStack.pop()!) // "A"
```

### 5. 정리

Swift가 제네릭을 도입한 이유는 **타입 안정성과 코드 재사용을 모두 만족시키기 위함**이다. 제네릭 덕분에 Swift는 다양한 타입을 안전하게 처리하면서도 간결하고 확장 가능한 코드를 작성할 수 있다.

> 제네릭은 복잡한 타입의 추상화를 가능하게 하고, 타입 안정성을 깨뜨리지 않으면서 범용적인 코드를 만드는 강력한 도구다.
---

## Keyword

- [[제네릭 (Generic)]]


#Noter 