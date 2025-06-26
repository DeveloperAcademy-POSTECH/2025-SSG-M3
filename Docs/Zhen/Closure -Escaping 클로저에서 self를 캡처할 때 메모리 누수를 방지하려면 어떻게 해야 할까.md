
>[!question]
>GQ1.  Escaping 클로저에서 self를 캡처할 때 메모리 누수를 방지하려면 어떻게 해야 할까

# Description

## 1. 클로저(Closure)란?
- 클로저는 “코드 블록” 자체를 값처럼 다룰 수 있게 해 주는 Swift의 문법
- **함수**처럼 매개변수와 반환값을 가질 수 있고,**람다(lambda)** 또는 **익명 함수(anonymous function)** 와 유사한 개념
- 주변 스코프(scope)에 있는 변수·상수를 “캡처(capture)”해서 내부에서 사용할 수 있다.

```swift
let factor = 10
let multiply = { (x: Int) in
    return x * factor   // factor를 캡처
}
print(multiply(3))      // 30

```

## 2. Escaping 클로저란?
- 함수 호출이 끝난 이후에도 실행될 가능성이 있는(탈출하는) 클로저
- Swift는 이런 파라미터에 `@escaping` 키워드를 **반드시** 붙여야 컴파일이 통과하게 만듦. 
- 주로 **비동기 콜백**, **타이머**, **DispatchQueue**, **프로퍼티 저장** 등에 사용

```swift
var handlers: [() -> Void] = []

func registerHandler(_ handler: @escaping () -> Void) {
    handlers.append(handler)  // 함수가 리턴된 뒤에 호출될 수 있으니 @escaping
}

func doAsyncWork(completion: @escaping (Data?) -> Void) {
    DispatchQueue.global().async {
        let data = fetchData()
        completion(data)        // 나중에 실행
    }
}

```

## 3. “self를 캡처한다”는 의미
- **캡처**: 클로저가 자신이 정의된 주변 환경 내의 변수나 객체(`self` 포함)에 대한 참조(reference)나 복사본을 내부에 저장하는 것
- 값 타입(Struct, Enum)은 복사(copy)
- 참조 타입(Class)은 기본적으로 **강한 참조(strong reference)** 로 캡처

```swift
class A {
  var value = 0
  func doSomething() {
    DispatchQueue.global().async {
      self.value += 1     // 여기서 self(A 인스턴스)를 강하게 캡처
    }
  }
}
```

## 4. 메모리 누수(Memory Leak)와 그 원인
- **메모리 누수**: 더 이상 사용되지 않는 객체가 해제되지 않고 메모리에 남아 있는 상태
- **원인(순환 참조 Retain Cycle)**:
	1. **객체(self)**가 클로저를 **강하게 참조(strong)**
	2. 클로저가 그 **self**를 강하게 참조  → 서로를 잡고 놓지 못해 둘 다 해제되지 않음

```swift
class MyViewController {
  var completion: (() -> Void)?

  func start() {
    completion = {
      self.view.backgroundColor = .red
    }
  }
  // MyViewController 인스턴스는 completion에 의해 해제되지 않음
}

```


## 5. 메모리 누수 방지: 캡처 리스트(Capture List) ⭐️
### 1. `[weak self]` 사용
- 약한 참조(weak reference) 로 캡처 -> 클로저 실행 시점에 `self`가 이미 해제되면 `nil`이 되어 안전하게 빠져나갈 수 있음

```swift
class MyViewController {
  var completion: (() -> Void)?

  func start() {
    completion = { [weak self] in
      guard let self = self else { return }    // self가 살아 있을 때만 실행
      self.view.backgroundColor = .red
    }
  }
}
```

### 2. `[unowned self]` 사용
- 비소유 참조(unowned reference)로 캡처
- 주의 : `self`가 절대 해제되지 않을 것이 “확실할 때”만 사용
    - 만약 `self`가 해제된 뒤 클로저가 호출되면 **런타임 크래시** 발생

```swift
class MyViewController {
  var completion: (() -> Void)?

  func start() {
    completion = { [unowned self] in
      // self는 언제나 살아 있다고 가정
      self.view.backgroundColor = .red
    }
  }
}
```

## 결론 ! : 권장패턴 


```swift
network.fetchData { [weak self] data in //기본 선택지
  guard let self = self else { return } //클로저 본문 중간에 self 유효성 한 번만 체크
  self.updateUI(with: data) //같은 메서드 내부에서 UI/상태 변경을 집중 처리
}

```


## 코드 예시
+ 실제 코드 예시를 작성

## Keywords
+ [[클로저 (Closure)]]

## References
- https://babbab2.tistory.com/164
- https://velog.io/@parkgyurim/Swift-escaping-closure

## 작성자
- #Zhen 