
> [!question]  
> GQ: enum에서 메서드와 연산 프로퍼티를 정의하는 건 어떤 장점이 있을까?

### 1. 개요

Swift에서 `enum`은 단순한 열거형(enumeration)을 넘어서, **메서드나 연산 프로퍼티까지 가질 수 있는 강력한 타입**이다. 이를 통해 더 깔끔하고 타입 중심적인 설계를 할 수 있다.

기존 열거형은 단지 상수를 나열하는 용도였다면, Swift의 열거형은 **로직과 데이터를 함께 다룰 수 있어** 더 유연하고 강력한 도구로 활용된다.

### 2. 메서드와 연산 프로퍼티를 enum에 정의하는 이유

**2.1 연관된 행위를 각 case에 맞게 묶기**

메서드나 연산 프로퍼티를 enum 내부에 두면
**각 case에 맞는 로직을 해당 타입 내부에서 직접 정의**할 수 있어 유지보수와 가독성이 높아진다.

```swift
enum Weather {
    case sunny, rainy, cloudy

    var description: String {
        switch self {
        case .sunny: return "맑음"
        case .rainy: return "비"
        case .cloudy: return "흐림"
        }
    }

    func emoji() -> String {
        switch self {
        case .sunny: return "☀️"
        case .rainy: return "☔️"
        case .cloudy: return "☁️"
        }
    }
}

let today = Weather.sunny
print(today.description) // "맑음"
print(today.emoji())     // "☀️"
```

**2.2 타입 안전성 향상**

외부에서 switch-case로 로직을 분기하는 대신, 내부에서 메서드나 프로퍼티로 처리하면 **타입 외부에 분기 로직이 흩어지지 않는다**\* → 코드 구조가 단단해지고, 에러 가능성도 줄어든다.

타입 외부에 분기 로직이 흩어지지 않는다*
: 어떤 타입을 만들었는데, 이 타입에 따라 동작을 다르게 해야 할 때 그 로직을 타입 바깥에서 여기저기 switch문으로 처리할 필요 없이 타입 안에 묶어둘 수 있다는 것.

**2.3 rawValue나 연관값을 활용한 표현력 강화**

타입이 가진 **기본 정보(rawValue, 연관값)** 만으로는 부족한 경우 rawValue를 바탕으로 **보여줄 정보, 판단 로직, 표현 방식**을 **연산 프로퍼티나 메서드에 담아** 타입에 녹여낼 수 있다. (더 풍부한 정보를 제공할 수 있음)

```swift
enum Beverage: String {
    case coffee = "Coffee"
    case tea = "Tea"

    var caffeineContent: Int {
        switch self {
        case .coffee: return 95
        case .tea: return 40
        }
    }
}

let drink = Beverage.tea
print("Caffeine: \(drink.caffeineContent)mg") // Caffeine: 40mg
```

Beverage는 원래 'Coffee' 와 'Tea'라는 값만 가지고 있었지만 caffeineContent이라는 연산 프로퍼티(저장되어있는 게 아니라 동적으로 계산해서 반환하는 프로퍼티)를 붙여 더 풍부한 정보를 가질 수 있게 되었다!

### 3. 사용 사례

**3.1 상태(state) 표현**

```swift
enum AppState {
    case loading
    case success(data: String)
    case failure(error: Error)

    var isLoading: Bool {
        if case .loading = self { return true }
        return false
    }
}
```

- 앱 상태 로직을 enum 내부에서 정의함으로써 깔끔한 표현 가능

**3.2 라우팅 / 화면 전환**

```swift
enum AppRoute {
    case home
    case detail(id: Int)
    case settings

    var title: String {
        switch self {
        case .home: return "홈"
        case .detail: return "상세 보기"
        case .settings: return "설정"
        }
    }
}
```

### 4. 정리

Swift의 enum은 단순한 값 나열을 넘어서, **자기 자신이 갖는 의미와 기능을 하나의 타입 안에서 표현할 수 있는 고수준 추상화 도구**다.

- 메서드와 연산 프로퍼티를 정의하면:
    1. **관련 로직을 내재화**할 수 있어 응집도가 높아진다.
    2. **타입 안전성**이 증가한다.
    3. **코드 재사용과 유지보수**에 유리하다.

> 결과적으로, enum 자체를 하나의 상태머신이나 기능 단위로 사용할 수 있게 해주는 디자인이 가능해진다.

---
## Keyword

- [[열거형 (Enumeration)]]

#Noter 