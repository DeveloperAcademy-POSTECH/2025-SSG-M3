**GQ - 강한 참조와 약한 참조, 비소유 참조의 개념 차이는 무엇이며, 각각 사용 목적은 무엇인가?**

### ARC란?

Automatic Reference Counting(자동 참조 카운팅)은 메모리 사용을 자동으로 추적하고 관리한다. 대부분의 경우에 클래스 인스턴스가 만들어졌을 때 자동으로 메모리를 할당했다가 인스턴스가 더이상 필요하지 않으면 자동으로 할당된 메모리를 해제한다.

```swift
class Person {
	let name: String
	init(name: String) {
		self.name = name
		print("\(name) is being initialized")
	}
	deinit {
		print("\(name) is being deinitialized")
	}
}
```

다음과 같은 `Person` 클래스는 `name` 프로퍼티를 가지고 `init`과 `deinit` 시 메세지를 출력한다. `deinitializer`는 클래스의 인스턴스 할당이 해제되었을 때 실행된다.

```swift
var reference1: Person?
var reference2: Person?
var reference3: Person?

reference1 = Person(name: "John")
// John is being initialized
```

다음과 같이 변수 `reference1`에 `Person` 클래스 인스턴스가 할당되었다. 이때 `init` 메세지가 출력된다. `reference1`은 새로운 `Person` 인스턴스에 **강한 참조**를 가진다. 최소 하나의 강한 참조가 있기 때문에, ARC는 `Person` 메모리 할당을 해제하지 않는다.

```swift
reference2 = reference1
reference3 = reference1
```

이제 총 하나의 `Person` 인스턴스에 **3개의 강한 참조**가 생겼다.

```swift
reference1 = nil
reference2 = nil
```

이와 같이 `reference`, `reference2`의 참조를 해제하더라도 `deinit` 메세지는 출력되지 않는다. 아직 1개의 강한 참조가 남아있기 때문이다.

```swift
reference3 = nil
// John is being deinitialized
```

이때 비로소 메세지가 출력된다.

### Strong Reference Cycle(강한 순환 참조)

강한 참조가 하나라도 남아있으면 인스턴스 할당이 해제가 되지 않다는 점 때문에 memory leak(메모리 누수)가 생기는 경우가 있다.

```swift
class Person {
	let name: String
	init(name: String) { self.name = name }
	var apartment: Apartment?
	deinit() { print("\(name) is being deinitialized") }
}

class Apartment {
	let unit: String
	init(unit: String) { self.unit = unit }
	var tenant: Person?
	deinit() { print("Apartment \(unit) is being deinitialized") }
}

var john: Person?
var unit4A: Apartment?

john = Person(name: "John")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john

// John is being initialized
// Apartment 4A is being initialized
```

john은 unit4A를, unit4A는 john을, 즉 **두 인스턴스가 서로를 강하게 참조한다.** 각 인스턴스는 참조하는 곳이 2개로 카운트된다.


```swift
john = nil
unit4A = nil
```

따라서 이와 같이 nil로 만들어도 여전히 1개의 참조가 남아있기 때문에 `deinit` 메세지가 출력되지 않는다. 이는 메모리 누수(memory leak) 현상을 일으킨다.

### Weak Reference(약한 참조)

이러한 사이클을 해결하기 위해서 weak reference를 사용한다.

```swift
class Person {
	let name: String
	init(name: String) { self.name = name }
	var apartment: Apartment?
	deinit() { print("\(name) is being deinitialized") }
}

class Apartment {
	let unit: String
	init(unit: String) { self.unit = unit }
	weak var tenant: Person?
	deinit() { print("Apartment \(unit) is being deinitialized") }
}
```

다음과 같이 Apartment 클래스의 tenant 변수를 weak var로 만들었다.

 그러면 tenant 변수는 Person 인스턴스를 약하게 참조한다. 이는 Person 인스턴스의 참조 카운트를 증가시키지 않는다.

```swift
john = nil
// John is being deinitialized
```

Person 인스턴스는 john 변수에 의한 강한 참조만 없어지면(-1) ARC에 의해서 자동으로 메모리 할당이 해제된다. Person 인스턴스에서 나온 강한 참조 또한 사라졌기 때문에, Apartment 인스턴스도 unit4A 변수에 의한 강한 참조만 사라지면 할당이 해제될 것이다.

```swift
unit4A = nil
// Apartment 4A is being deinitialized
```

### Weak Reference vs. Unowned Reference(약한 참조와 비소유 참조의 차이)

비소유 참조도 약한 참조와 마찬가지로 인스턴스에 강하게 참조하지 않는다. 다른 점이라면, 비소유 참조는 다른 인스턴스가 더 긴 수명을 가지고 있을 때 사용된다. 또 항상 값을 가지고 있어야 한다.

따라서 객체가 항상 존재한다고 보장되는 경우에만 써야 한다.

```swift
class Person {
	var card: CreditCard?
}

class CreditCard {
	unowned var owner: Person
}
```

이 경우 `CreditCard`는 항상 `Person`보다 먼저 해제되기 때문에 `owner`은 항상 존재한다고 보장된다. 만약 `owner`가 먼저 해제되면, `CreditCard`에서 `owner` 접근 시 crash가 일어난다.

**Reference**

- [공식 스위프트 문서-Automatic Reference Counting](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)
- [(블로그)강한 순환 참조와 해결 방법](https://velog.io/@parkgyurim/Swift-Strong-Reference-Cycle)

#Nika