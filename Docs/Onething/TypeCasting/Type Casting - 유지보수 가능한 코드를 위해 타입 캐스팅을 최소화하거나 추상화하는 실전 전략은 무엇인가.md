
> [!question]
> **유지보수 가능한 코드를 위해 타입 캐스팅을 최소화하거나 추상화하는 실전 전략은 무엇인가**

## 개요
Swift는 타입 세이프티(type safety)를 강하게 지향하는 언어다.  
하지만 실무에선 `as`, `as?`, `as!` 같은 **타입 캐스팅 연산자**를 쓰게 되는 순간들이 있다.  
이 자체가 나쁜 건 아니지만, **잘못된 설계 구조나 책임의 흐트러짐**을 드러내는 신호일 수 있다.  
결국 중요한 건 **캐스팅을 줄이고, 타입 추상화로 의도를 드러내는 것**이다.

---
## 1. 왜 타입 캐스팅이 위험할까?
- **런타임 오류의 원인**  
  `as!` 연산자는 다운캐스팅이 실패할 경우 앱이 크래시 난다.  
  옵셔널 캐스팅(`as?`)도 잦은 nil 체크나 조건문 남발로 코드 가독성을 해친다.

- **응집도와 책임 흐림**  
  객체를 특정 타입으로 강제 캐스팅하는 건,  
  해당 타입의 내부 구현을 외부에서 알거나 기대하고 있다는 의미다.  
  이는 OOP의 기본 원칙인 **은닉(encapsulation)**을 위반한다.

---

## 2. 실전에서 흔히 만나는 캐스팅 예

```swift
func handle(_ item: Any) {
    if let user = item as? User {
        print(user.name)
    } else if let group = item as? Group {
        print(group.title)
    }
}
```
이 코드는 스위치 문처럼 보이지만,
사실 타입 기반 분기를 한다는 건 **설계가 객체지향적으로 풀리지 않았다는 신호**다.

---
## **3. 해결 전략 1 – Protocol 추상화**

**공통된 인터페이스**를 명확히 정의해서,
타입을 몰라도 행동할 수 있게 만드는 게 핵심이다.

```swift
protocol Displayable {
    var displayName: String { get }
}

struct User: Displayable {
    var name: String
    var displayName: String { name }
}

struct Group: Displayable {
    var title: String
    var displayName: String { title }
}

func handle(_ item: Displayable) {
    print(item.displayName)
}
```

이렇게 하면 handle()은 User와 Group의 구체 타입을 몰라도 동작할 수 있다.
**타입이 아니라 역할에 집중하는 사고 방식**이 중요하다.

---
## **4. 해결 전략 2 – 열거형(Enums)으로 표현할 수 있을까?**

의미 있는 경우에는 enum으로 **의도를 드러내는 표현**이 낫다.
```swift
enum Item {
    case user(User)
    case group(Group)
}

func handle(_ item: Item) {
    switch item {
    case .user(let user):
        print(user.name)
    case .group(let group):
        print(group.title)
    }
}
```

이 방식은 타입이 명시적으로 정의되기 때문에 **컨트롤 흐름이 명확하고 예측 가능**하다.
단, enum이 과도하게 확장될 경우엔 오히려 **유연성이 떨어질 수 있다.**

---
## **5. 해결 전략 3 – 제네릭 활용**

제네릭을 활용하면 코드의 **재사용성과 타입 안정성**을 모두 잡을 수 있다.
```swift
func render<T: Displayable>(_ item: T) {
    print(item.displayName)
}
```
제네릭은 **구체 타입에 구애받지 않고, 동작을 일반화**할 수 있는 강력한 도구다.

---

## **6. 해결 전략 4 – 의존성 주입과 추상화 조합**

객체가 내부에서 구체 타입을 직접 생성하거나 캐스팅하지 않도록,
**역할 중심의 프로토콜을 주입받아 동작**하게 한다.
```swift
protocol Service {
    func fetch() -> String
}

class ViewModel {
    let service: Service
    init(service: Service) {
        self.service = service
    }

    func show() {
        print(service.fetch())
    }
}
```
여기서 ViewModel은 실제 구현체의 타입을 몰라도 된다.
결국 **구조의 유연성과 테스트 편의성** 모두 챙길 수 있다.

## Keywords
- [[Type Casting]]

## GQ
- **제네릭과 프로토콜 지향 설계는 어떻게 조합될 수 있으며, 이 조합이 타입 캐스팅 최소화에 어떤 기여를 할 수 있을까**

#Onething 