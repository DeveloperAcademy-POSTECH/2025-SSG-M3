>[!question]
** Optional 타입과 관련 메서드는 어떻게 구현되어 있고, 실무에서 안전하게 다루기 위한 패턴은?**
 
## Optional 타입, 왜 등장했을까?
Swift에서 Optional은 값이 **있을 수도, 없을 수도 있는** 상태를 표현하기 위해 만들어졌다.
기존의 C 계열 언어에서는 null이 아무 타입에나 들어갈 수 있었고, 이로 인해 수많은 런타임 에러가 발생했다.

> Swift는 `Optional<T>`을 통해 “존재할 수도 있는 값”을 **명시적으로** 코드에 표현한다.

---

##  **Optional은 어떻게 구현되어 있을까?**

  Swift에서 Optional은 단순한 enum이다.
```swift
enum Optional<Wrapped> {
    case none
    case some(Wrapped)
}
```
여기서 핵심은, some이 실제 값을 갖고 있고, none은 nil을 의미한다는 점.
이렇게 **두 가지 상태만** 가질 수 있도록 enum으로 설계되어 있기 때문에,
컴파일 타임에 **nil 가능성**을 강제할 수 있다.

---

## **map / flatMap / compactMap의 차이와 활용**

이 메서드들은 Optional을 안전하게 다루기 위한 **Monadic 연산자**처럼 동작한다.
### **map**
- 내부 값을 꺼내서 변형한 다음, 다시 Optional로 감싼다.
```swift
let name: String? = "Onething"
let length = name.map { $0.count } // Optional(8)
```
### **flatMap**
- 중첩된 Optional을 **한 번만 감싼 Optional**로 평탄화한다.
```swift
let input: String? = "123"
let number = input.flatMap { Int($0) } // Int? = Optional(123)
```
> map은 `Optional<Optional<T>>`을 만들 수 있고, flatMap은 그걸 평탄하게 만들어준다.

###  **compactMap**
- Sequence에서만 쓰인다.
- nil을 제거하고, 실제 값만 모아서 배열로 만든다.
```swift
let inputs = ["1", "a", "2"]
let numbers = inputs.compactMap { Int($0) } // [1, 2]
```
???

CompactMap?
```swift
func compactMap<ElementOfResult>(
    _ transform: (Element) throws -> ElementOfResult?
) rethrows -> [ElementOfResult]
```
1. **transform 함수**를 받아서, 배열의 각 요소에 적용함
2. 그 결과가 Optional인 경우
3. **nil을 제거하고, some 값들만 모아서 새로운 배열**로 반환함

ex.
```swift
let input = ["1", "2", "hello", "4"]
let result = input.compactMap { Int($0) }
// ["1", "2", "4"] → [1, 2, 4]
```
여기서 "hello"는 Int("hello")가 nil이 되기 때문에 제거됨.
즉, **Optional → Non-Optional로 변환하면서 nil을 자동으로 걸러내는** 고마운 함수.

#### **내부 구현을 살펴보면?**
```swift
public func compactMap<ElementOfResult>(
    _ transform: (Element) throws -> ElementOfResult?
) rethrows -> [ElementOfResult] {
    var result: [ElementOfResult] = []
    for element in self {
        if let value = try transform(element) {
            result.append(value)
        }
    }
    return result
}
```
- if let을 내부에서 사용해 nil을 거르고 있음
- 값이 존재할 때만 append

####  **언제 쓸까?**
- **Optional을 다룰 때 가장 흔하게 사용되는 패턴**
- 서버 응답에서 일부 값이 없을 수 있는 경우
- 여러 타입을 변환하면서 nil을 걸러내야 할 때

```swift
let users: [User?] = [User(name: "A"), nil, User(name: "B")]
let validUsers = users.compactMap { $0 }  // [User(name: "A"), User(name: "B")]
```

#### **왜** **map** **과 다를까?**
```swift
let values = ["1", "2", "hi"]
let mapped = values.map { Int($0) } 
// → [Optional(1), Optional(2), nil]

let compactMapped = values.compactMap { Int($0) }
// → [1, 2]
```
- map: 결과가 [Int?] (Optional 포함)
- compactMap: 결과가 [Int] (Optional 제거)


---

##  **실무에서 안전하게 Optional 다루기**

Optional은 강력하지만, 잘못 쓰면 if let 지옥이나 강제 언래핑(!)으로 이어진다.
안전하게 쓰기 위한 몇 가지 실전 패턴을 정리한다.

###  **패턴 1. guard로 초기에 거르기**
```swift
func greet(_ name: String?) {
    guard let name else {
        print("No name")
        return
    }
    print("Hello, \(name)")
}
```
- 가드문은 early-exit을 유도해 **복잡한 중첩을 방지**해준다.

###  **패턴 2. Optional chaining + nil coalescing**
```swift
let length = user?.profile?.bio?.count ?? 0
```
- 중간에 nil이 있으면 0으로 fallback
- 복잡한 구조의 안전한 탐색에 유용

### **패턴 3. flatMap을 활용한 연산 연결**
```swift
let urlString: String? = "https://example.com"
let url = urlString
    .flatMap { URL(string: $0) }
    .flatMap { try? Data(contentsOf: $0) }
```
- Optional을 안전하게 연산 체인으로 연결할 수 있다.
- 조건을 만족하지 않으면 조용히 nil로 흐름이 종료됨

---

### **피해야 할 패턴**
- ! 강제 언래핑 → 크래시 유발
- ?? "" 무작정 디폴트 → 의미 없는 값이 흘러다닐 수 있음
- if let 반복 → 코드 가독성 저하

---

## **Insight**

Optional은 단순히 값이 있을지 없을지를 표현하는 문법이 아니다.
**실패 가능성 자체를 설계의 일부로 포함**시키는 방식이다.
map, flatMap은 Optional을 **값의 컨테이너이자 연산 흐름의 제어 구조**로 확장시켜 준다.
Swift는 이를 통해 **안전하고 선언적인 에러 방어 설계**를 가능하게 한다.

> 결국 중요한 건 **Optional을 없애는 게 아니라**,
> **명확하게 다루는 것**이다.

---

### Keywords
- [[Swift Standard Library]]
- [[옵셔널 (Optionals)]]

### 작성자
- #Onething 