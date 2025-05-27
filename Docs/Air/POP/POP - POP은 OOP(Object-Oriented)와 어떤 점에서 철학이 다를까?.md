>[!question]
>GQ. POP은 OOP(Object-Oriented)와 어떤 점에서 철학이 다를까?

| 특징     | OOP              | POP                                    |
| ------ | ---------------- | -------------------------------------- |
| 핵심 개념  | 클래스 (class)      | 프로토콜 (protocol)                        |
| 코드 재사용 | 상속 (Inheritance) | 조합 (Composition)                       |
| 타입     | 참조 타입 중심 (class) | 값 타입 중심 (struct, enum), class에서도 사용 가능 |
| 다형성    | 클래스 상속 기반        | 프로토콜 채택 기반                             |
| 기본 구현  | 부모 클래스           | 프로토콜 익스텐션                              |
## 객체 중심과 프로토콜 중심
POP와 OOP 모두 소프트웨어의 재사용성과 유지보수성을 높이기 위해 도입된 프로그래밍 패러다임이다. 그러나 그 구현 방식에 차이가 있다.
- OOP는 클래스와 상속을 중심으로 설계된다. 데이터와 행위를 하나의 단위(객체)로 묶고, 계층적인 상속 구조를 통해 구현한다.
- POP는 프로토콜이 설계의 핵심이다. 행위의 계약을 먼저 정의하고, 구조체, 열거형, 클래스가 이 계약을 채택함으로 기능을 구현한다.

## 상속과 조합
- OOP에서 서브클래스는 단 하나의 슈퍼클래스에서만 상속 받을 수 있다. 상속 계층 트리가 복잡해질 수 있다.
- POP에서는 여러 프로토콜을 조합할 수 있기 때문에 더 유연하게 기능을 결합할 수 있다.

## 참조 타입 중심과 값 타입 중심
- OOP는 참조 타입인 클래스를 사용한다.
- POP는 주로 값 타입을 사용한다.

## 예제 코드
### OOP
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

class Cat: Animal {
    override func sound() -> String {
        return "Meow"
    }
}

func makeSound(of animal: Animal) {
    print(animal.sound())
} 

makeSound(of: Dog()) // Woof
makeSound(of: Cat()) // Meow
```
### POP
```swift
protocol Animal {
    func sound() -> String
}

extension Animal {
    func sound() -> String {
        return "Some sound"
    }
}

struct Dog: Animal {
    func sound() -> String {
        return "Woof"
    }
}

struct Cat: Animal {
    func sound() -> String {
        return "Meow"
    }
}

func makeSound(of animal: Animal) {
    print(animal.sound())
}

makeSound(of: Dog()) // Woof
makeSound(of: Cat()) // Meow
```

## Keywords
- [[Docs/Common/Keyword/프로토콜 지향 프로그래밍 (POP)]]
- [[객체 지향 프로그래밍 (OOP)]]


#Air