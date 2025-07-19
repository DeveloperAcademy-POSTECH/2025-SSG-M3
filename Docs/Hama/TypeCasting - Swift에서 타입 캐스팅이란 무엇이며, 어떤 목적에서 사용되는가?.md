[!question]
>[!question]
>GQ1. Swift에서 타입 캐스팅이란 무엇인가?
>GQ2. 어떤 목적에서 사용되는가?

## Description

타입 캐스팅이란, 어떤 값이 실제로 어떤 타입인지 확인하고, 그 타입처럼 사용하는 것을 말한다.
타입 캐스팅을 사용하는 이유는 처음엔 공통 타입으로 묶어놓고 다루다가, 나중에 실제 타입의 기능이 필요할 때 꺼내서 쓰기 위함이다.

Swift의 타입 캐스팅은 is 연산자와 as 연산자를 사용한다.

is 연산자는 객체가 특정 클래스 타입이나, 그 서브클래스인지 검사할 때 사용된다.
as 연산자는 타입을 변환할 때 사용된다.

특히, 상속 관계에서의 타입 변환에는 두 개념이 있다.

1. 업캐스팅(Upcasting): 예를 들어 apple 이 fruit 을 상속한다면,
```swift
let fruits: [Fruit] = [Apple(), Banana(), Orange()]  // 모두 업캐스팅
```
   서로 다른 하위 타입 객체들을 공통 부모인 fruit 타입의 배열에 넣을 수 있다.
   업캐스팅 후에는 부모 클래스에 정의된 멤버만 접근 가능하고, 자식 클래스에만 있는 프로퍼티나 메서드는 보이지 않는다.
   ```swift
class Fruit {
    func wash() {
        print("과일을 씻어요")
    }
}

class Apple: Fruit {
    func peel() {
        print("사과 껍질을 벗겨요")
    }
}

let apple = Apple()
apple.peel()  // Apple 타입이니까 peel 사용 가능

let fruit: Fruit = apple  // 업캐스팅

fruit.wash()   // Fruit에 있는 기능 사용 가능
// fruit.peel() ❌ 오류! Fruit 타입은 peel을 모름
   ```
   
   
2. 다운캐스팅(Downcasting): 다운캐스팅은 실패할 가능성이 있으므로, Swift에서는 두 가지 다운캐스팅 연산자를 제공한다. as? (조건부 캐스팅)와 as! (강제 캐스팅).
```swift
if let realApple = fruit as? Apple {
    realApple.peel()  // 다운캐스팅 성공 → 사과 기능 사용 가능
}
```
1. as? 는 캐스팅에 실패하면 nil을 반환하므로, 주로 if let 또는 guard 구문과 함께 사용한다. 안전한 캐스팅이다.
```swift
let fruit: Fruit = Banana()

if let realApple = fruit as? Apple {
    // 이건 실패! Banana는 Apple이 아니니까
}
```
2. as! 는 캐스팅이 실패하면 런타임 오류가 발생하므로, 캐스팅이 반드시 성공한다고 확신할 때만 사용한다. 일반적으로 앱이 크래시 나는 것을 피하기 위해 as! 보다는 as? 로 안전하게 처리하는 것을 권장한다.
```swift
class Fruit {}
class Apple: Fruit {
    func peel() {
        print("사과 껍질 벗기기")
    }
}

let fruit: Fruit = Apple()   // 업캐스팅

// 다운캐스팅 시도
if let realApple = fruit as? Apple {
    realApple.peel()   // 사과 껍질 벗기기
}
```

## Keywords
+ [[타입 캐스팅 (Type Casting)]]

## 작성자
- #Hama 