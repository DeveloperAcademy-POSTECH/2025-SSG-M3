
<aside> 📌

**Generic** 은 타입을 매개변수처럼 받는 기능이다. **PAT** 는 프로토콜을 채택하는 타입이, 구체적인 타입을 명시하는 기능이다.

</aside>

### Generic

함수나 타입을 사용할 때, 자유롭게 타입이 정해진다.

- 예시
    
```swift
func wrapGift<T>(_ gift: T) {
	print("🎁 \\(gift) 포장 완료!")
}
    
wrapGift("사과")       // 🎁 사과 포장 완료!
wrapGift(123)         // 🎁 123 포장 완료!
wrapGift(["책", "펜"]) // 🎁 ["책", "펜"] 포장 완료!
```
    
    이렇게 사용하는 입장에서, 타입이 자유롭게 정해지는 것을 볼 수 있다.
    

### PAT(**Protocol with Associated Type)**

해당 프로토콜을 채택할 때, 타입이 정해진다.

- 예시
    
```swift
protocol Container {
    associatedtype Item
    func add(_ item: Item)
}
    
struct IntContainer: Container {
    func add(_ item: Int) { print(item) }
	    // 여기서 Item == Int
}
```
    
    이렇게 프로토콜에서 associatedtype 으로 타입을 ‘미정의’ 해두면, IntContainer 에서 Int 로 타입을 정해 사용하는 것을 볼 수 있다.
    

---

GQ: 만약, asscoiatedtype 를 안쓰면 어떻게되나요?

GA: 타입이 고정이 되어서, 다른 타입으로 사용할 수 없습니다.

```swift
protocol BadContainer {
		//  associatedtype Item
		//  func add(_ item: Item)
    func add(_ item: Int)  // 고정된 타입!
}
```

- GQ: ‘_’ 이 기호는 왜 붙이나요?
    - GA: 이름 없이 바로 값만 넣기 위해서 사용합니다.

```swift
// '_' 있을 때, 외부 이름 생략

func sayHello(_ name: String) {
    print("Hello, \\(name)!")
}

sayHello("Hama")
```

```swift
// '_' 없을 때, 외부 이름 필수

func sayHello(name: String) {
    print("Hello, \\(name)!")
}

sayHello(name: "Hama")
```