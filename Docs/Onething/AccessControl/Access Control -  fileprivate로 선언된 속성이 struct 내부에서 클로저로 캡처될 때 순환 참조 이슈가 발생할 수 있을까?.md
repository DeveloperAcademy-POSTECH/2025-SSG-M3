
Swift에서 접근 제어는 단순히 외부 접근을 제한하는 수단이 아니다. 구조와 책임을 명확히 하고, 예상치 못한 참조를 줄여 **메모리 안정성**까지도 챙길 수 있는 기초 도구다.

fileprivate 접근 제어자가 **struct 내 클로저 캡처**와 관련하여 어떤 영향을 주는지, 실제 코드 스니펫과 함께 탐구해본다.

### **1. Struct와 순환 참조?**

보통 ARC 기반의 순환 참조 문제는 class에서 발생한다. class는 참조 타입이기 때문에 인스턴스가 서로를 강하게 참조하면 메모리 해제가 되지 않는다.

하지만 struct는 **값 타입**이다. 따라서 내부 클로저가 자신을 캡처한다고 해도 **복사**가 이루어질 뿐, 기본적으로 순환 참조는 발생하지 않는다.

그럼에도 struct에서도 클로저 캡처가 문제가 될 수 있는 케이스가 있다. 특히 클로저가 escape 하거나, 내부에 참조 타입을 담고 있는 경우다.

### 2. 실험:  fileprivate 접근제어자와 캡처

```swift
fileprivate struct Counter {
    fileprivate var count = 0
    fileprivate lazy var increment: () -> Void = {
        self.count += 1
    }
}
```
이 코드 자체는 컴파일 오류 없이 잘 작동한다. fileprivate 키워드는 **같은 파일 내**에서는 접근 가능하기 때문에, self.count 접근에 문제가 없다.
또한 lazy 속성이기 때문에 클로저는 self를 캡처하게 된다.

그렇다면 순환 참조가 발생할까?

### **3. 순환 참조는 발생하지 않는다**

```swift
var counter = Counter()
counter.increment() // count = 1
```
문제없이 작동한다. 이유는 간단하다:
- struct는 값 타입이라 클로저가 self를 강하게 참조해도 메모리 누수는 없다.
- 단, struct 내부에 **class 인스턴스 프로퍼티**가 있고 그걸 캡처하면 이야기가 달라진다.

### **4. struct 내 참조 타입 캡처 시 예외**

```swift
class Logger {
    func log() { print("logging...") }
}

struct Container {
    fileprivate var logger = Logger()
    
    lazy var closure: () -> Void = {
        self.logger.log()
    }
}
```
이 경우 Container는 Logger를 참조하고 있고, closure가 그걸 캡처하게 된다.
만약 이 Container 자체가 escape 되거나 retain cycle 구조 속에 들어가면, Logger가 해제되지 않는 문제가 생길 수 있다.

즉, struct 자체는 안전하지만 **struct 안에 참조 타입이 들어가면, 캡처 시 순환 참조의 위험이 생긴다.**

### **5. 외부 접근 제한 실험**

```swift
let myCounter = Counter()
// myCounter.increment() // ❌ 'increment' is inaccessible due to 'fileprivate' protection level
```

같은 파일 내에서는 접근 가능하지만, 다른 파일에서는 fileprivate 때문에 접근 불가능하다.
이를 통해 내부 구현을 외부에서 숨기고 싶을 때 효과적이다.


---
## **Finding & Synthesis**

- struct는 기본적으로 값 타입이라 순환 참조와는 거리가 있다.
- 그러나 내부에 참조 타입(class)이 있을 경우 클로저 캡처로 인해 retain cycle이 생길 수 있다.
- fileprivate은 클로저 캡처와 직접 관련은 없지만, 구현을 외부로부터 은닉하는 데 유용하다.
- lazy var로 선언된 클로저는 self를 캡처하며, 참조 타입이 들어가 있는 경우엔 주의가 필요하다.
- 실전에서는 unowned self, weak self는 struct보다는 class에서 고려할 문제이며, struct에서는 내부 참조 타입만 신경 쓰면 된다.


## GQ
- **값 타입인 struct가 참조 타입처럼 메모리 이슈를 유발할 수 있는 상황은 언제일까?**

## Keyword
- [[접근 제어 (Access Control)]]]


#Onething 