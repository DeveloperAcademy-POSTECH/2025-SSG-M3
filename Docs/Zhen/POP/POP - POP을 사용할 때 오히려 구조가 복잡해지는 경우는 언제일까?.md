>[!question]
>GQ6. **POP을 사용할 때 오히려 구조가 복잡해지는 경우**는 언제일까?

## Description
- 프로토콜 지향 프로그램은, 기본적으로 상속의 한계를 넘어서 **코드의 재사용성과 모듈화를 높이는 것**을 목표로 한다. 
- 프로토콜은 기본적으로 **규약**이다. 설계 혹은 추상화를 통해 타입이 가져야 할 속성과 행동을 명세하기 위한 것이고, 따라서 전역에서만 사용이 가능하다. (함수와 다른 점)

- 그렇다면 언제 비효율적일까? 


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
***
### 비효율적으로 사용하는 사례
#### 1. 동일한 로직이 여러 타입에서 중복 사용될 때 
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
* 1-2. 해결방안 : extension 과 함께 사용하자 ~ 
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
***
#### 2. 저장 속성(stored property)을 사용하고 싶을 때. 
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
- 프로토콜에는 구체적인 값이 들어가지 않고, 실제 저장되는 것도 아니기 때문에 저장 속성을 사용할 수 없다. 

만약 var id 를 계속 넣어야 할 때, 아래처럼 비효율적이 될 수 있음 
~~~
struct User: Identifiable {
    var id: String // 여기서 직접 선언
}

struct Book: Identifiable {
    var id: String // 또 다시 선언
}

~~~
* 2-2. 해결방안 : 이럴 때는 class 와 상속을 사용하는 게 편하다~ 
***
#### 3. Generic 관련 문제 (⭐️⭐️⭐️⭐️) - *associatedtype*
- cf. Generic 이란? 
~~~
func swapInts(_ a: inout Int, _ b: inout Int) {
    let temp = a
    a = b
    b = temp
}

//이 함수를 제네릭으로 바꾸면 

func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

// cf. inout은 값 자체를 넘기는게 아니라 참조로 전달해줌
~~~

즉, 제네릭은 추상화(유연성)를 위해 필요한 것인데, protocol 에서 Generic을 사용하려면 'associatedtype'을 사용해야 함. 

~~~
protocol Stack {
	associatedtype value    //타입 제한 두고 싶으면 Equatable 도 ok
    func push(value: value)
    func pop() -> value

}

//이렇게 정의하면 채택하는 곳에서는 다음과 같이 사용

struct VStack: Stack {
	typealias value = Int  // 생략하고 바로 value 에 Int 넣어도 됨
    func push(value: value) { }
    func pop() -> value { ... }

}

~~~

근데? 이런식으로 associatedtype 을 사용하면 꺼내 쓸 때 주의해야 함. 
~~~
protocol DataSource {
    associatedtype Item
    func item(at index: Int) -> Item
}

//여기서 에러 발생
let source: DataSource
~~~
컴파일러가 DataSource 는 아는데 그 안에 Item의 타입을 정확하게 정의하지 않았기 때문에 에러가 나는 것. 

해결방법
- 3-1. 제네릭으로 감싸줌
~~~
> func printItems<Source: DataSource>(source: Source) {
    let firstItem = source.item(at: 0)
    print(firstItem)
}
~~~

- 3-2. type erasure (타입 지우기 ?)
~~~
struct AnyDataSource<T>: DataSource {
    typealias Item = T
    private let _item: (Int) -> T

    init<DS: DataSource>(_ dataSource: DS) where DS.Item == T {
        _item = dataSource.item
    }

    func item(at index: Int) -> T {
        return _item(index)
    }
}

//이렇게 하면 let source: DataSource 해서 쓸 수 있다고 함..
~~~

#### 3-1. associatedtype + Generic 일 때는 기본값 가질 수 x 
위에서 배웠던 것 처럼 extension 을 통해 디폴트 값을 만들고 싶을 때 주의할 점임. 
~~~
protocol DataSource {
    associatedtype Item
    func add(_ item: Item)
}

extension DataSource {
    func add(_ item: Item = ???) {
    }
}

//이러면 Generic parameter 'Item' could not be inferred 에러 등장
~~~

에러가 나는 이유 자체는 똑같음 ! 
컴파일러가 타입 추론을 못해서... -> 결국 타입을 확정하거나 다른 방식으로 작성하면 됨

방법 1 
~~~
struct IntSource: DataSource {
    func add(_ item: Int = 0) {
        print("Added \(item)")
    }
}
~~~
방법 2
~~~
func add<T>(_ item: T = 0) {
    print(item)
}
~~~

## Keywords
+[[Docs/Common/Keyword/프로토콜 지향 프로그래밍 (POP)]]
+[[제네릭 (Generic)]]
 
## References
- 블로그 
	- https://babbab2.tistory.com/136
	- https://babbab2.tistory.com/181
	- https://mojitobar.tistory.com/9
- Apple 의 공식 문서 
	- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/

#Zhen