
>[!question]
>GQ1. Swift가 Protocol-Oriented를 강조한 이유는 무엇일까?

### 1. 개요
Swift는 참조 타입인 클래스보다 값 타입인 구조체를 더 자주 쓰는 언어다.
그런데 구조체는 상속이 안 되므로 "공통 기능을 어떻게 나눌 수 있을까?"하는 문제가 생긴다.

그래서 Swift는 클래스를 상속해서 기능을 재사용하는 방식(OOP) 대신,
프로토콜을 기준으로 기능을 나누고 조합하는 방식(POP)을 더 강조한다.
프로토콜 지향 프로그래밍을 통해 **구조적 설계, 코드 재사용, 유연한 추상화**를 동시에 실현하고자 한다.

### 2. POP을 강조하게 된 배경
**2.1 구조체 중심 설계 지향**
Swift는 값 타입(value type)인 구조체(struct), 열거형(enum)을 우선적으로 활용하는 언어다.
하지만 구조체는 클래스처럼 상속이 불가능하므로, 기능 공유를 위해 상속 대신 프로토콜을 활용하는 설계 방식이 필요하다.

**2.2 단일 상속의 한계 극복**
객체지향 언어 클래스는 단일 상속만 지원하므로, 기능을 계층 구조로 관리하면 구조가 복잡해지고 유연성이 떨어진다. 반면 프로토콜은 **다중 채택(multiple conformance)** 이 가능해 기능을 조합하는 방식으로 더 유연한 설계를 할 수 있다.

**2.3 타입 안정성과 추상화의 균형**
Swift는 타입 안정성이 강한 언어다.
프로토콜은 명확한 인터페이스(약속)을 정의함으로써, 다양한 타입을 추상화하면서도 **타입 안정성(type safety)** 을 유지할 수 있게 한다.

### 3. 주요 특징 비교: OOP vs. POP
| **항목** | **객체지향 프로그래밍 (OOP)** | **프로토콜 지향 프로그래밍 (POP)**      |
| ------ | -------------------- | ---------------------------- |
| 중심 개념  | 클래스(class)           | 프로토콜(protocol)               |
| 코드 재사용 | 상속(Inheritance)      | 조합(Composition)              |
| 타입 중심  | 참조 타입(class)         | 값 타입(struct, enum), class 가능 |
| 다형성 구현 | 클래스 상속 기반            | 프로토콜 채택 기반                   |
| 기본 구현  | 부모 클래스에서 정의          | 프로토콜 + Extension             |

### 4. 예제 코드
**4.1 OOP(클래스)**
Dog은 Animal 클래스를 상속해서 sound 기능을 바꾼다.

```swift
class Animal {
    func sound() -> String {
        return "Some sound"
    }
}

class Dog: Animal {
    override func sound() -> String {
        return "Woof"
    }
}

func makeSound(of animal: Animal) {
    print(animal.sound())
}

makeSound(of: Dog()) // 출력: Woof
```

여기서 클래스 대신 구조체를 쓰려면?

프로토콜 + extension 조합을 쓴다!

**4.2 POP(구조체)
```swift
protocol Animal {
    func sound() -> String // sound()라는 함수가 필수라고 선언. 약속
}
  
extension Animal {
// 기본 구현을 제공. 타입이 직접 sound()를 구현하지 않았을 때 extension에 정의한 기본 구현이 쓰인다.
    func sound() -> String {
        return "Some sound"
    }
}

struct Dog: Animal { // sound()를 직접 구현 -> "Woof"
    func sound() -> String {
        return "Woof"
    }
}

struct Sheep: Animal {} // sound() 구현 없음

func makeSound(of animal: Animal) {
    print(animal.sound())
}

makeSound(of: Dog())
// Swift가 내부적으로 animal이 어떤 타입인지를 보고 만약 Dog가 sound()를 구현하고 있으면 그 구현히 호출됨 -> Woof 출력
makeSound(of: Sheep()) // 출력: Some sound
```

extension*
- 프로토콜을 더 간단하게 채택할 수 있도록 기본값이 제공해주는 수단
- 반드시 커스텀 구현이 필요하지 않은 함수들에 대해 기본 동작 제공

**정리**
> protocol은 인터페이스(약속. 이 타입은 이 기능을 반드시 갖고 있어야 한다는.),
extension은 약속을 안 지킬 경우의 기본 행동,
struct가 직접 약속을 지키면 그게 우선 !

왜 좋은가?
1. 상속 없이도 공통 기능을 재사용할 수 있다.
2. 구조체나 enum도 프로토콜을 채택할 수 있으니 훨씬 유연하다.
3. 기능을 조합하듯이 만들 수 있어서 코드가 덜 복잡해진다. 필요한 기능만 골라 쓰기 간편.

보다 현실적인 코드 예시
**4.3 POP(구조체로 정의한 다양한 알림 수단)**
```swift
struct PushNotifier: Notifier {
    func notify(message: String) {
        print("📲 푸시 알림: \(message)")
    }
}

struct EmailNotifier: Notifier {
    func notify(message: String) {
        print("📧 이메일 발송: \(message)")
    }
}

struct InAppNotifier: Notifier {
    func notify(message: String) {
        print("🔔 앱 내부 알림: \(message)")
    }
}

struct SilentNotifier: Notifier {
    func notify(message: String) {
        // 아무것도 안 함
    }
}

// 사용
func sendWelcome(notifier: Notifier) {
    notifier.notify(message: "환영합니다! DeepPearl에 오신 걸 환영해요.")
}

sendWelcome(notifier: PushNotifier())
sendWelcome(notifier: EmailNotifier())
sendWelcome(notifier: InAppNotifier())
sendWelcome(notifier: SilentNotifier())

```

**4.4 POP(공통 네트워크 요청 기능 추상화)**
```swift
protocol APIRequestable {
    func fetchData(from url: URL) async throws -> Data
}

extension APIRequestable {
    func fetchData(from url: URL) async throws -> Data {
        let (data, _) = try await URLSession.shared.data(from: url)
        return data
    }
}

struct UserAPI: APIRequestable {}
struct ProductAPI: APIRequestable {}

// 사용
Task {
    let url = URL(string: "https://example.com/user")!
    let data = try await UserAPI().fetchData(from: url)
}
```
네트워크 기능을 class로 상속하지 않고
여러 API 타입에서 공통으로 사용할 수 있는 **유연한 추상화 구조**


번외)
값 타입은 왜 상속이 안 될까?
값 타입은 '필요하면 새로 복사해서 쓰는' 게 철학이다.
상속은 '부모 자식이 연결되어 있고, 관계가 유지되는' 구조.

이 둘은 방향성이 다르다.

상속은 참조 기반 관계 유지,
값 타입은 복사해서 독립 사용.

값 타입에 상속이 들어오면
- 상위 타입에서 변경한 것이 하위에 영향을 주고
- override, super 호출, init 복잡성 등 복잡도와 버그 가능성이 올라간다.
- Swift는 "값 타입은 가볍고 안전하게 쓰자"라는 철학이라 애초에 상속 자체를 금지한다.


### 5. 정리
 Swift가 POP을 강조하는 이유는 **값 타입 중심의 설계를 가능하게 하면서도** **기능 재사용과 다형성을 더 유연하게 구현**할 수 있기 때문이다.
  
> POP은 단지 상속을 안 쓰자는 게 아니라
**기능을 조립하듯 설계하고, 더 유연하고 안전한 코드를 만들기 위한 철학!**



## Keyword
- [[프로토콜 지향 프로그래밍 (POP)]]