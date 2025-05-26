
>[!question]
>GQ1. Protocol Extension은 어떻게 코드 중복을 줄이고 있을까?
>GQ2. Protocol을 따르는 것과 그냥 상속 받는 것의 차이는 무엇이지
>
## Description

- **프로토콜이란, 어떤 기능에 적합한** **특정 메서드, 프로퍼티 및 기타 요구 사항의 청사진(Bluprint)을 의미합니다** **프로토콜은 클래스, 구조체, 열거형에 의해 채택되며,** **프로토콜에 정의된 요구사항의 실제 구현을 제공합니다** **프로토콜의 요구 사항을 모두 충족하는 모든 유형(클래스/구조체/열거형)은** **해당 프로토콜에 부합하다고 합니다.** 쉽게 말하면 프로토콜은 메서드나 속성 등에 대한 요구사항(청사진)을 정의하는 **일급객체**(걍 변수나 상수로 사용 가능하다는 뜻)
- 쉽게 설명하자면, 일종의 약속인거임. 만약 어떤 함수가 저 프로토콜을 따를거야 라고 선언한다면, 그 프로토콜내에서 선언해둔 약속들을 다 따라야함.
- 근데 상속이랑 프로토콜이랑 다른점이 뭘까요 -> 상속의 단점 : 상속을 위해 캡슐화(Encapsulation)가 깨질 수 밖에 없다. swift 엔 protected 키워드가 없어요. 그리고 상속을 많이 하게 되면 부모 클래스가 매우 모호해져서 자식클래스에서 코드 중복이 심해짐, 단일 상속만 가능해서 계속 연결됨, 결합도 증가로 유연성과 확장성 떨어짐 → 이러한 단점들로 인해 프로토콜을 사용함
- class는 object 자체이므로 변경될 여지가 많고 실제로 상속을 받았을 때 오버라이딩하는 형태로 코드가 길어지고 복잡해짐, 하지만 프로토콜로 변경하면 새로운 기능을 추가할 땐 또 다른 protocol을 만들어서 그 약속을 지키게하는 형태가 되므로 나중에 추가하기 쉽고, 수정하기 쉬운 코드로 바뀜

## 주요 기능

- 일반적으로 프로토콜을 채택한 커스텀 타입(클래스, 구조체, 열거형)은 프로토콜의 모든 요구사항을 충족하여 사용해야 합니다. **같은 프로토콜을 채택하여 사용하는 커스텀 타입이 많을수록 중복되는 필수 기능 코드 또한 많아짐.**
- **프로토콜 확장 시 프로토콜의 필수 기능을 미리 작성해 놓으면 채택 후 따로 작성할 필요가 없슴. → 이걸로 코드 중복을 줄이는 거임.** 즉, 확장을 통해 **프로토콜의 필수 기능을 기본으로 구현하여 사용하는 방식**.
- 확장을 통해 얻은 디폴트(기본) 기능을 재정의하여 사용할 경우 **재정의된 기능을 우선적으로 사용함**. → 만약에 extension 에서 func a() {print(”a”)} 라 해두고 밑에 프로토콜을 따르는 클래스에서 func a() {print(”aa”)}라고 해두면, func a는 print(”aa”)임.
- 추가로 정의한 기능은 재정의하여 사용할 수 없음. 만약 밑에 코드에서 newMethod를 extension이 아니라 함수에서 재정의 한다면, 무시됨

## 코드 예시

```swift
//Extension 사용하지 않는 예시
protocol AB{
    func a()
    func b()
}

class Test1: AB{
    func a() {      // 중복 코드
        print("a")
    }
    func b() {      // 중복 코드
        print("b")
    }
}

class Test2: AB{
    func a() {      // 중복 코드
        print("a")
    }
    func b() {      // 중복 코드
        print("b")
    }
}

struct Test3: AB{
    func a() {      // 중복 코드
        print("a")
    }
    func b() {      // 중복 코드
        print("b")
    }
}

```

```swift
// Extension 사용 예시
protocol AB{
    func a()
    func b()
}

extension AB{
    func a(){   //확장을 통해 프로토콜의 필수 기능을 미리 구현 
        print("a")
    }
    func b(){   //확장을 통해 프로토콜의 필수 기능을 미리 구현 
        print("b")
    }
   
    func newMethod(){   // 확장을 통해 새로 추가한 메서드
        print("newMethod")
    }
}

class Test1: AB{
}

class Test2: AB{
}

struct Test3: AB{
}

var kim = Test2()
kim.a()          //a
kim.b()          //b
kim.newMethod()  //newMethod
```

Extension을 사용하여, 3번 쓸 코드를 한번만 작성하게 됨. 또한 확장하여 메서드를 추가할 수도 있음.

## Keywords

- 상속

## References

- [https://kirkim.github.io/ios/2022/08/05/inheritance_protocol.html](https://kirkim.github.io/ios/2022/08/05/inheritance_protocol.html)
- [https://ios-development.tistory.com/1377](https://ios-development.tistory.com/1377)
- [](https://velog.io/@shin_ms/%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9CProtocol%EC%9D%98-%ED%99%95%EC%9E%A5Extension)[https://velog.io/@shin_ms/프로토콜Protocol의-확장Extension](https://velog.io/@shin_ms/%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9CProtocol%EC%9D%98-%ED%99%95%EC%9E%A5Extension)