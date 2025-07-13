### Any, AnyObject란?
Swift에서 모든 타입을 유연하게 처리할 수 있는 타입 지시자(type-erased types)이다. `Any`는 모든 타입을 담을 수 있고, `AnyObject`는 클래스 타입 인스턴스만 담을 수 있다. 

```swift
let a: Any = 432
let b: AnyObject = MyClass()
```

`Any`는 **다양한 타입을 한 컬렉션에 담고 싶을 때**, **타입을 모를 때**, 혹은 **함수에 어떤 타입이든 받을 수 있도록 하려 할 때** 사용한다.
```swift
let values: [Any] = [42, "Hello", true, 3.14]
```

`AnyObject`는 오직 클래스 인스턴스만 다룰 때 사용한다.
```swift
class Dog {}
class Cat {}
let objects: [AnyObject] = [Dog(), Cat()]
```

### 타입 캐스팅과의 관계
`Any`는 타입 정보가 모호하므로 **캐스팅 없이는 사용 불가**
```swift
let unknown: Any = "Swift"

if let str = unknown as? String {
    print("String 타입: \(str)")
}
```

`AnyObject`도 마찬가지로 캐스팅 필요
```swift
let obj: AnyObject = Dog()

if let dog = obj as? Dog {
    dog.bark()
}
```

`as!`를 쓰면 변환 실패 시 런타임 크래시
```swift
let unknown: Any = 42
let str = unknown as! String
```


#Nika