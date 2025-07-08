
 > [!question]  
> GQ. is 연산자는 어떤 역할을 하며, 어떤 상황에서 유용하게 쓰일 수 있는가?

---

### 0. 개요

Swift의 타입 캐스팅은 런타임 중 객체가 **어떤 타입인지 확인**하거나, **다른 타입으로 변환**하는 기능이다.
그중 is 연산자는 인스턴스가 특정 타입인지 **판별**하는 데 사용된다.

is는 다음과 같은 경우에 유용하게 쓰인다:

- 배열이나 딕셔너리 등 Any 타입이 섞인 컬렉션에서, **요소의 타입을 구분**하고 싶을 때
- 다운캐스팅(as?, as!) 전에 **안전하게 타입을 체크**하고 싶을 때
- 특정 인스턴스가 **프로토콜을 채택하고 있는지 검사**할 때

이러한 역할 덕분에 is는 **다형성(polymorphism)**과 **타입 추상화**가 흔한 Swift 코드에서 자주 등장한다.

→ 즉, is는 Swift에서 **유연한 타입 처리**를 가능하게 해주는 중요한 연산자다.

---
### 1. 키워드 개념 정리
#### 1) Type Casting (타입 캐스팅)

- **실행 중(runtime)에 타입을 검사하거나, 다른 타입으로 변환하는 기능**
- Swift에서는 두 가지 키워드를 사용함:
    - is: 타입 확인
    - as?, as!: 타입 변환

### 2) is 연산자

- 특정 인스턴스가 **어떤 타입인지 검사**할 때 사용
- 반환값은 Bool (true or false)
- 주로 **조건문(if, guard)** 안에서 사용됨
```swift
if object is String {
    print("이건 문자열이야!")
}
```

---
### 2. 주요 활용 상황

#### 1) 타입 분기 처리

```swift
let items: [Any] = ["노터", 123, true]

for item in items {
    if item is String {
        print("문자열 발견!")
    } else if item is Int {
        print("정수 발견!")
    }
}
```

- Any 타입처럼 다양한 타입을 담을 수 있는 컬렉션에서 **타입별로 다른 로직을 적용**할 때 유용

#### 2) 타입 변환 전 확인

```swift
let value: Any = "Hello"

if value is String {
    let str = value as! String // 안전하게 강제 변환 가능
    print(str.uppercased())
}
```

- 강제 다운캐스팅(as!) 전에 is로 타입을 미리 확인해두면 **런타임 오류 방지**

#### 3) 프로토콜 채택 여부 확인
```swift
protocol Flyable {}
class Bird: Flyable {}
let sparrow = Bird()

if sparrow is Flyable {
    print("이 새는 날 수 있어!")
}
```
- is는 **클래스뿐 아니라 프로토콜 채택 여부**도 검사 가능함

---

### 3. 정리
|**항목**|**설명**|
|---|---|
|is|인스턴스가 특정 타입인지 확인 (Bool 반환)|
|쓰임|조건 분기, 다운캐스팅 전 확인, 프로토콜 채택 확인|
|조합|as?, as!와 함께 쓰면 안전한 타입 캐스팅 가능|

### Keyword
[[TypeCasting]]

### 작성자
#Noter 
