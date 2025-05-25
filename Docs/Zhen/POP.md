>[!question]
>GQ6. **POP을 사용할 때 오히려 구조가 복잡해지는 경우**는 언제일까?

## Description
- 프로토콜 지향 프로그램은, 기본적으로 상속의 한계를 넘어서 **코드의 재사용성과 모듈화를 높이는 것**을 목표로 한다. 
- 프로토콜은 기본적으로 규약이다. 설계 혹은 추상화를 통해 타입이 가져야 할 속성과 행동을 명세하기 위한 것이고, 따라서 전역에서만 사용이 가능하다. (함수와 다른 점)
- 비효율적인 경우를 알려면 효율적인 경우를 먼저 생각해야 함. 

- 그렇다며


## 코드 예시
- 1. 공통 인터페이스로 다형성을 제공하는 경우
 ```
 let vehicles: [Drivable] = [Car(), Bus()]
for v in vehicles {
    v.drive()
}
```
- 2. extension 을 사용하면 기본 구현을 둘 수도 있음 (+override)
```
protocol Drivable {
    func drive()
}

extension Drivable {
    func drive() {
        print("기본 운전 방식입니다")
    }
}

struct Car: Drivable {}       // 구현 안 해도 기본 제공
struct Bus: Drivable {
    func drive() { print("버스 운전은 좀 달라요") }
}

Car().drive() // 기본 운전 방식
Bus().drive() // 버스 운전은 좀 달라요

```


## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)