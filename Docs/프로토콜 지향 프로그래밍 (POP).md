>[!question]
>GQ1. POP가 뭘까?
>GQ2. POP와 OOP의 차이가 뭘까?
>GQ3. POP의 장점이 뭔가?
>GQ4. 어떻게 사용해야하는가, 어떨때 사용해야하는가?
>

## Description

POP(Protocol-Oriented Programming), 프로토콜을 중심으로 설계하고 구현하는 프로그래밍 방식이다. Apple은 Swift의 설계 철학으로 이 프로토콜 지향 프로그래밍을 강조하며, 이를 통해 코드의 재사용성, 유연성, 테스트 용이성을 향상시킬 수 있다고 설명한다. POP은 클래스 상속의 한계를 극복하고, 구조체와 프로토콜을 활용하여 더 모듈화된 코드를 작성하는 데 중점을 두고 있다.

클래스를 중심으로 상속 구조를 만드는 OOP와 달리,
**공통된 요구사항은 프로토콜로 정의하고,
코드 재사용은 '프로토콜 + extension'의 조합으로 해결하는 방식**이다.

이러한 특징들은 **테스트, 기능 확장, 유지보수, 재사용성을 높이는데 도움**이 된다.

반대로 클래스는 단일 상속만을 지원하며, 상속은 클래스끼리만 할 수 있다. 클래스는 주로 상속을 통한 코드 재사용을 강조하는 반면, 프로토콜은 특정 기능을 가져오는 방식을 강조한다. 이는 더 모듈화된 코드와 유연한 디자인을 가능하게 한다.

| **항목**     | **OOP (객체지향)**   | **POP (프로토콜지향)**       |
| ---------- | ---------------- | ---------------------- |
| 중심 철학      | 클래스, 상속          | 프로토콜, 합성               |
| 코드 재사용     | 상속 (Inheritance) | extension, composition |
| 다형성        | 상속 구조에서 구현       | 프로토콜을 따르는 다양한 타입       |
| 유연성        | 낮음 (단단한 계층)      | 높음 (유연한 조합)            |
| 타입         | 클래스 위주           | 구조체/열거형 중심             |
| Swift와의 궁합 | 낮음               | **매우 높음**              |

>✏️ **요약**
>
> 프로토콜 지향 프로그래밍(POP)은
> 기존 객체지향(OOP)의 단점들을 보완하기 위해 Swift에서 제안된 설계 철학.
   OOP는 클래스를 중심으로 **상속을 통해** 기능을 확장하지만,
   이 방식은 상속 관계가 복잡해지고, 유연성이 떨어지는 단점이 있음.
   반면 POP는 **프로토콜 + 익스텐션 + 합성(composition)** 을 통해
   필요한 기능만 골라 조합할 수 있음.
   특히 구조체, 클래스, 열거형 등 모든 타입에 적용할 수 있고,
   기능 단위로 역할을 분리하기 때문에 유지보수성과 테스트 용이성이 훨씬 좋아짐.


## 주요 기능
+ 역할(기능) 단위로 쪼개고 조합하여 확장성 있는 앱 구조를 만들 수 있다. (유연한 조합설계)
+ **조합적 설계**방식으로 필요한 기능만 가볍게 구성 가능하다. (작은 단위로 기능설계)
+ 클래스 상속은 딱 **하나만 가능**하지만, 프로토콜은 여러 개를 동시에 채택할 수 있다. (여러 프로토콜 채택)

## 코드 예시
```swift
protocol Drivable {
    func drive()
}

extension Drivable {
    func drive() {
        print("운전 중입니다.")
    }
}

struct Car: Drivable {}
struct Bike: Drivable {}

let myCar = Car()
myCar.drive() // "운전 중입니다."

let myBike = Bike()
myBike.drive() // "운전 중입니다."
```

여러 개를 동시에 채택할 수 있다는 예시
```swift
protocol Flyable { func fly() }
protocol Swimmable { func swim() }

struct Duck: Flyable, Swimmable {
    func fly() { print("날아요") }
    func swim() { print("헤엄쳐요") }
}
```



## Keywords
+ OOP
+ Protocol
+ extension

## References
- https://chatgpt.com/share/682ae31f-0174-8013-9e7c-597093378cbc - Chatgpt
- https://lrl.kr/chdft 
- https://ios-development.tistory.com/1377 - 이 사람이 들어준 예시를 이해하고 싶다. (원칙들: DIP?, ISP? SRP?)