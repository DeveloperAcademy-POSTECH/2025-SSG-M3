>[!question]
>GQ6. **POP을 사용할 때 오히려 구조가 복잡해지는 경우**는 언제일까?

## Description
- 프로토콜 지향 프로그램은, 기본적으로 상속의 한계를 넘어서 **코드의 재사용성과 모듈화를 높이는 것**을 목표로 한다. 
- 프로토콜은 기본적으로 **규약**이다. 설계 혹은 추상화를 통해 타입이 가져야 할 속성과 행동을 명세하기 위한 것이고, 따라서 전역에서만 사용이 가능하다. (함수와 다른 점)

- 그렇다면 언제 비효율적일까? 
- 


## 코드 예시
### Protocol 을 효율적으로 사용하는 사례 
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
### 비효율적으로 사용하는 사례
1. 동일한 로직이 여러 타입에서 중복 사용될 때 
~~~
protocol Drivable {
    func drive()
}

struct Car: Drivable {
    func drive() { print("Drive car") }
}

struct Bus: Drivable {
    func drive() { print("Drive bus") } // 거의 비슷한 코드인데도 반복
}

~~~
1-1. 해결방안
~~~
protocol Drivable {
    func drive()
}

extension Drivable {
    func drive() {
        print("Driving in a default way")
    }
}

//이렇게 하면 디폴트 값을 넣어서 쓸 수도 있고 override 해서 쓰기도 가능. 
~~~
2. 저장 속성(stored property)을 사용하고 싶을 때. 
		cf. stored property 와 computed property 의 차이는 ? 
```
//저장 속성
struct Dog {
    var name: String
}
//연산 속성
var greeting: String {
    return "Hello!"
}
```
> 프로토콜에는 구체적인 값이 들어가지 않고, 실제 저장되는 것도 아니기 때문에 저장 속성을 사용할 수 없다. 

만약 var id 를 계속 넣어야 할 때, 아래처럼 비효율적이 될 수 있음 
~~~
struct User: Identifiable {
    var id: String // 여기서 직접 선언
}

struct Book: Identifiable {
    var id: String // 또 다시 선언
}

~~~
## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)