>[!question]
>[!question]
>GQ1. 접근 레벨(Access Level)이란?
>GQ2. 프로토콜에서 말하는 '요구사항'은 무엇을 의미하는가?
>GQ3. subclassing, override 가 뭘까?
>GQ4. 프로토콜 선언 시 접근 레벨과 요구사항(requirement)이 미치는 영향은? subclassing 및 override 관점에서 두 키워드의 차이를 구체적으로 비교해보세요.

## Description

- GQ1. 접근 레벨(Access Level)이란?
: Swift에서 접근 레벨은 **어디서 사용할 수 있는지를 제어**하는 규칙을 의미한다.

**접근 수준**은 다음과 같다.
open - **모든 모듈**에서 사용 가능.
	클래스와 메서드에만 사용됨.
	모든 모듈에서 하위 클래스와 오버라이드 가능.
public - **모든 모듈**에서 사용 가능.
	같은 모듈에서 하위 클래스는 가능하지만, 다른 모듈에서 하위 클래스는 불가능.
	모든 모듈에서 오버라이드는 안됨.
internal (기본값) - **같은 모듈** 내에서만 사용 가능.
fileprivate - **같은 파일** 내에서만 사용 가능.
private - **같은 스코프 또는 블록** 내에서만 사용 가능.

* GQ2. 프로토콜에서 말하는 '요구사항'은 무엇을 의미하는가?
: 이 프로토콜을 따르려면, **반드시 이 기능을 구현해야 한다** 라는 약속 목록이다.
```swift
protocol Greetable {
    func greet()
}
// 여기서 greet()가 바로 요구사항임.

struct Person: Greetable {
    func greet() {
        print("안녕하세요!")
    }
}
// Person이 Greetable을 채택했기 때문에, 요구사항인 greet()를 꼭 구현해야 함.
```

**요구사항과 접근 레벨의 관계**
: 프로토콜의 접근 레벨이 public이라면, 그 안의 요구사항들도 public이어야 해요.
```swift
public protocol Greetable {
    func greet()
}

public struct Person: Greetable {
    public func greet() { // 반드시 public
        print("Hello")
    }
}
```

* GQ3. subclassing, override 가 뭘까?
: 앞서, GQ1 에서 **open**과 **public**의 차이를 언급했었다.
둘 다 모든 모듈에서 사용 가능하다.
open 클래스는 하위 클래스(subclassing)와 오버라이드(override)가 가능하지만,
public 클래스는 같은 모듈에서 하위 클래스(subclassing)는 가능해도, 오버라이드(override)는 막혀있다.

**하위 클래스(subclassing)**
: 기존 클래스(부모 클래스)를 상속받아, 새로운 클래스(자식 클래스)를 만드는 걸 말해요.
```swift
class Animal {
    func sound() {
        print("동물이 소리를 낸다")
    }
}

class Dog: Animal {
}
```
Dog는 Animal의 하위 클래스, Dog는 Animal의 모든 기능을 그대로 사용할 수 있음.

**오버라이드(override)**
: 부모 클래스의 기능을 자식 클래스가 바꿔서 다시 구현
```swift
class Animal {
    func sound() {
        print("동물이 소리를 낸다")
    }
}

class Dog: Animal {
    override func sound() {
        print("멍멍!")
    }
}
```
Dog는 Animal의 sound()를 override(재정의) 함. Dog().sound()를 실행하면 “멍멍!"

* GQ4. 프로토콜 선언 시 접근 레벨과 요구사항(requirement)이 미치는 영향은? subclassing 및 override 관점에서 두 키워드의 차이를 구체적으로 비교해보세요.
: 다음과 같다.

1. 접근 레벨과 요구사항의 관계
internal protocol - 이게 기본값. 같은 모듈 안에서만 사용 가능.
public protocol - 외부 모듈에서도 채택 가능. 단, **요구사항도 모두 public이어야 함**.
open protocol - ❌ Swift에는 **open protocol이 없음**. 프로토콜 자체는 open할 수 없음.

2. 요구사항이 미치는 영향
public protocol 안에 정의된 함수, 변수, associatedtype 등은 전부 public 접근 수준으로 자동 간주됨. 명시 안 해도 됨.
단, 이 프로토콜을 채택하는 타입은 요구사항 구현도 **public이어야 함**.
그렇지 않으면 외부에서 호출 불가함.

## Keywords
+ [[접근 제어 (Access Control)]]

## 작성자
- #Hama 