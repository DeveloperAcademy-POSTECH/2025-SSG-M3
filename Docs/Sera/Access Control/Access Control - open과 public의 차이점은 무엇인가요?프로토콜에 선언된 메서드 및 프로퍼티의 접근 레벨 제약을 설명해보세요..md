
>[!question]
>GQ1. open과 public의 차이점은 무엇인가요?  프로토콜에 선언된 메서드 및 프로퍼티의 접근 레벨 제약을 설명해보세요.

## Open
- open은 가장 높은 수준의 접근 제어자로 다른모듈에서도 해다 ㅇ클래스나 메서드를 서브클래싱하거나 오버라이딩할 수 있다. open으로 선언된 클래스나 메서드를 상속하거나 재정의해서 사용할 수 있기 때문에 보통 외부 라이브러리를 만들고 사용할 때 유용하다. 
## Public 
- public은 기본적으로 open이랑 접근제어 정도는 같은데, 다른 모듈에서 서브클래싱하거나 오버라이딩 할 수는 없다. 

open과 public의 공통점은 둘 다 모듈 내/외적으로 사용 할 수 있다는 것인데, open은 완전개방이여서 모듈 밖에서도 자유롭게 상속하고 매서드를 재정의할 수 있지만, public은 오직 해당 클래스가 정의된 모듈 안에서만 상속과 재정의가 가능하다. 

```swift
// MyFramework.swift

import Foundation

// MARK: - Open (개방형)
// 모듈 밖에서도 상속 및 재정의(override)가 가능합니다.
open class Animal {
    public init() {}

    open func makeSound() {
        print("동물이 소리를 냅니다.")
    }
}


// 모듈 밖에서 접근은 가능하지만, 상속 및 재정의는 불가능합니다.
public class Person {
    public init() {}

    public func introduce() {
        print("저는 사람입니다.")
    }
}
```

- **`open class Animal`**: 이 클래스는 모듈 밖(앱)에서 상속받아 새로운 동물을 만들고, `makeSound()` 메서드를 자신만의 소리로 재정의할 수 있습니다.
- **`public class Person`**: 이 클래스는 모듈 밖(앱)에서 `Person()` 객체를 생성하고 `introduce()` 메서드를 호출할 수는 있지만, 새로운 종류의 사람을 상속으로 만들거나 `introduce()`를 재정의할 수는 없습니다.

```swift
// MyApp.swift

import MyFramework // 우리가 만든 프레임워크를 가져옵니다.


// MARK: - Open 클래스 사용
// 'open'으로 선언된 Animal 클래스는 상속 및 재정의가 자유롭습니다.

class Dog: Animal { // 성공: 'open' 클래스이므로 상속 가능
    override func makeSound() { //  성공: 'open' 메서드이므로 재정의 가능
        print("멍멍!")
    }
}

let myDog = Dog()
myDog.makeSound() // 출력: "멍멍!"


// 'public'으로 선언된 Person 클래스는 상속 및 재정의가 불가능합니다.

//  아래 코드는 컴파일 오류를 발생시킴.
class Developer: Person { //  오류: 'public' 클래스는 모듈 밖에서 상속할 수 없습니다.
    override func introduce() { //  오류: 'public' 메서드는 모듈 밖에서 재정의할 수 없습니다.
        print("저는 개발자입니다.")
    }
}


// 'public' 클래스는 이렇게 객체를 생성해서 사용하는 것만 가능합니다.
let aPerson = Person()
aPerson.introduce() // 출력: "저는 사람입니다."
```


## 프로토콜에 선언된 메서드 및 프로퍼티의 접근 레벨 제약
구현체는 약속(프로토콜)과 자신(타입)의 수준 중 더 높은 기준을 따라야 한다.
프로토콜의 요구사항(메서드, 프로퍼티)을 구현하는 최종 접근 수준은, **① 프로토콜 자체의 접근 수준**과 **② 프로토콜을 채택한 타입의 접근 수준** 중 **더 개방적인(높은) 수준 이상**이어야 함

> **구현체의 접근 수준 ≥ max(프로토콜의 접근 수준, 타입의 접근 수준)**

1. `public` 타입이 `public` 프로토콜을 채택 - `max(public, public)`은 `public`이므로, 프로토콜의 요구사항은 반드시 `public` 또는 그 이상인 `open`으로 구현해야함
2. `internal` 타입이 `public` 프로토콜을 채택 - `max(public, internal)`은 `public` 이므로 반드시 `public` 또는 `open` 으로 구현해야함
3. `public` 타입이 `internal` 프로토콜을 채택 - `max(internal, public)`은 `public`이므로 반드시 `public` 또는 `open`으로 구현해야 함


## 주요 기능
+ 실제 활용을 작성

## 코드 예시
+ 실제 코드 예시를 작성

## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- 작성한 사람의 닉네임 (예: # Air )