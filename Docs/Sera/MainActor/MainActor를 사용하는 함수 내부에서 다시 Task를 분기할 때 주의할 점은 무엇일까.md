>[!question]
>GQ1.  MainActor를 사용하는 함수 내부에서 다시 Task를 분기할 때 주의할 점은 무엇일까

## Description
- ### **Actor** 
- `actor`는 Swift의 새로운 타입 중 하나로, 기본적으로 `class`와 유사한 구조를 갖는다. 가장 중요한 특징은 데이터 경합(race condition)을 방지해준다는 것이다. 
	- race condition: 두 개 이상의 쓰레드가 동시에 같은 데이터를 접근하고, 그 중 하나 이상이 **쓰기(write)** 작업일 때 발생.
- `actor`는 내부 상태에 접근할 때 자동으로 동기화 메커니즘을 적용하여, 동시에 여러 작업이 접근하지 못하게 막아준다.  비동기 컨텍스트에서만 접근 가능하기 때문에, 외부에서 `actor`의 메서드나 속성에 접근할 땐 보통 `async/await`와 함께 사용한다. 

``` swift
actor Counter {
    var value = 0
    
    func increment() {
        value += 1
    }
}

let counter = Counter()
await counter.increment()  // 외부에서 접근할 땐 await 필요

```

- ### **MainActor**
- iOS 앱에서 UI 업데이트는 반드시 메인 스레드에서만 실행되어야 한다. 하지만 Swift의 `async/await` 비동기 코드 흐름 안에서는 어떤 코드가 어느 스레드에서 실행될지 예측하기 어렵다.  그래서 Swift는 "이 코드는 무조건 메인 스레드에서 실행하라"는 의미로 `@MainActor`를 제공한다.
- `@MainActor`를 함수, 속성, 타입에 선언하면, 컴파일러가 해당 코드가 반드시 메인 스레드에서 실행되도록 보장해준다.
``` swift
@MainActor
class ViewModel: ObservableObject {
    @Published var name: String = ""

    func updateName() {
        name = "Juno"  //  UI와 연결된 데이터라서 메인에서 안전하게 실행됨
    }
}
```

 - `MainActor`도 일종의 actor이기 때문에, 비메인 스레드에서 접근하려면 `await`가 필요하다.
 ``` swift 
Task {
    await viewModel.updateName()  // 메인 스레드에서 안전하게 실행됨
}

 ```
 

## 주요 기능
+ ### Task 
+ `Task { ... }`는 새로운 비동기 작업을 시작하는 Swift의 기본 구조. 내부적으로 새로운 컨커런시 task를 생성해서 실행 큐에 등록해. 기본적으로 `Task`는 백그라운드 우선순위에서 실행됨. 하지만 context에 따라 `Task`가 메인에서 실행될 수도, 백그라운드에서 실행될 수도 있음.

MainActor 에서 Task를 생성하면 기본적으로 해당 Task는 MainActor 컨텍스트를 상속받지 않음.  즉, 내부 코드가 백그라운드에서 실행될 수도 있기 때문에 ui업데이트를 하면 위험할 수 있음. 
``
```swift
@MainActor
func problematicFunction() {
    Task {
        // 이 코드는 MainActor에서 실행되지 않음
        // UI 업데이트 시 런타임 에러 발생 가능
        someUILabel.text = "Updated"
    }
}
```

해결방법:
1. Task { @MainActor in ... } 로 선언하기 ``
```swift
@MainActor
func problematicFunction() {
    Task { @MainActor in
        someUILabel.text = "Updated"
    }
}
```
이렇게 선언하면 Task가 MainActor에서 실행될 것을 보장해줌. 
2. Task(priority:) { await MainActor.run { ... } }
```swift
@MainActor
func problematicFunction() {
    Task {
         await MainActor.run {
        someUILabel.text = "Updated"
        }
    }
}
```
1번보다 좀 더 유연하게 사용가능하다. 필요할 때만 명시적으로 MainActor로 전환함.


## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
#Sera 