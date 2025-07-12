>[!question]
>GQ. 프로토콜 타입과의 캐스팅은 어떻게 이뤄지는가? as?, as!, as의 사용 예시는

## Description
- 프로토콜과 함께 사용될 때, 타입 캐스팅은 Swift의 다형성을 극대화하고 유연한 설계를 가능하게 하는 필수적인 도구가 된다.

## 주요 기능
#### 연산자별 주요 기능 및 활용
| 연산자   | 주요 목적              | 실행 시점  | 반환 타입            | 실패 시 동작    |
| ----- | ------------------ | ------ | ---------------- | ---------- |
| `is`  | 런타입 타입 확인          | 런타임    | `Bool`           | `false` 반환 |
| `as`  | 보장된 변환 (업캐스팅, 브리징) | 컴파일 타임 | `T` (대상 타입)      | 컴파일 에러     |
| `as?` | 안전한 다운캐스팅 시도       | 런타임    | `T?` (옵셔널 대상 타입) | `nil` 반환   |
| `as!` | 강제 다운캐스팅           | 런타임    | `T` (대상 타입)      | 런타임 에러     |

## 코드 예시
+ 프로토콜을 사용하여 클래스와 구조체를 아우르는 다형적 코드, 타입 캐스팅 활용

##### `as`
```Swift
protocol Vehicle {
    func drive()
}

class Car: Vehicle {
    func drive() {
        print("자동차가 달립니다.")
    }
}

let myCar: Car = Car()
let myVehicle: Vehicle = myCar as Vehicle // 업캐스팅, 항상 성공하므로 as 사용 가능

myVehicle.drive() // 출력: 자동차가 달립니다.
```
##### `as?`
```Swift
protocol Flyable {
    func fly()
}

class Bird: Flyable {
    func fly() {
        print("새가 하늘을 납니다.")
    }
}

class Fish {
    func swim() {
        print("물고기가 헤엄칩니다.")
    }
}

let creatures: [Any] = [Bird(), Fish(), Bird()]

for creature in creatures {
    // creature가 Flyable 프로토콜을 준수하는지 확인하고 캐스팅
    if let flyingThing = creature as? Flyable {
        flyingThing.fly()
    } else {
        print("이 생물은 날 수 없습니다.")
    }
}

// 새가 하늘을 납니다.
// 이 생물은 날 수 없습니다.
// 새가 하늘을 납니다.
```
##### `as!`
```Swift
protocol Runnable {
    func run()
}

class Person: Runnable {
    func run() {
        print("사람이 뜁니다.")
    }
}

// 이 배열에는 Runnable을 준수하는 인스턴스만 들어있다고 확신하는 상황
let runners: [Any] = [Person(), Person()]

for runner in runners {
    // runner는 항상 Runnable 프로토콜을 준수한다고 확신하므로 강제 캐스팅 사용
    let runnableRunner = runner as! Runnable
    runnableRunner.run()
}

// 만약 배열에 다른 타입이 포함되면 런타임 에러 발생
// let mixedRunners: [Any] = [Person(), "Not a runner"]
// for runner in mixedRunners {
//     let runnableRunner = runner as! Runnable // "Not a runner"에서 크래시 발생
//     runnableRunner.run()
// }

// 사람이 뜁니다.
// 사람이 뜁니다.
```

## Keywords
+ [[Type Casting]]
+ [[Protocol]]

## References
- [The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/typecasting)

## 작성자
- #Air 