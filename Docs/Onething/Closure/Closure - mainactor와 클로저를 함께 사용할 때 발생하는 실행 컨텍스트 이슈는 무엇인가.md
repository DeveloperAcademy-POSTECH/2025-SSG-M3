
> [!question]
> **@mainactor와 클로저를 함께 사용할 때 발생하는 실행 컨텍스트 이슈는 무엇인가**

## **개요**
Swift의 @MainActor는 UI 업데이트처럼 **메인 스레드에서 실행되어야 할 작업**을 안전하게 처리하기 위해 사용한다.
하지만, 클로저(특히 escaping 클로저)와 함께 사용할 때는 **실행 컨텍스트(Actor context)** 가 암묵적으로 바뀌거나 유지되지 않아서, **예상치 못한 버그나 UI 업데이트 실패**가 발생할 수 있다.

---

## **1. 기본 개념 정리**
### **@MainActor**
- 메인 스레드에서 실행되어야 함을 나타내는 속성
- UI 관련 작업을 안전하게 처리할 수 있도록 격리를 제공
### **클로저 (Closure)**
- 비동기 작업의 콜백으로 자주 사용
- escaping 클로저는 호출 시점이 현재 스코프를 벗어나므로 실행 컨텍스트가 보장되지 않음

## **2. 이슈 상황**
자주 발생하는 실수 케이스이다.
```swift
@MainActor
func updateUI() {
    performTask { result in
        self.updateLabel(result) // ❌ 메인에서 실행된다고 보장할 수 없다
    }
}
```
여기서 updateUI()는 @MainActor로 선언되어 있지만, performTask의 클로저는 메인 스레드에서 실행된다는 보장이 없다.

즉, **클로저 내부에서의 실행 컨텍스트는 암묵적으로 MainActor를 유지하지 않는다.**

---
## **3. 해결 방법**

**명시적으로  MainActor 로 hop 해주기**
```swift
@MainActor
func updateUI() {
    performTask { result in
        Task { @MainActor in
            self.updateLabel(result) // 이제는 메인에서 안전하게 실행
        }
    }
}
```
또는, performTask의 내부 구현이 메인 큐에서 실행되는지 확실히 알고 있다면 @MainActor가 없어도 되지만,
**대부분의 경우 안전을 위해 명시적으로 hop을 유도하는 게 좋다.**

---
## **4. 왜 발생하는가**
클로저는 **실행 컨텍스트를 캡처하지 않는다.**
즉, @MainActor에서 정의된 함수라고 해도, 그 내부의 클로저는 별개의 Actor에서 실행될 수 있다.
이걸 이해하려면 Swift의 Actor 모델과 **Actor hop** 개념을 같이 보면 좋다

### **Actor hop 개념 이해하기**
Swift의 **Actor 모델**은 **데이터 경쟁(race condition)** 없이 안전하게 병렬 처리를 할 수 있도록 설계된 구조다.
Actor는 내부 상태를 보호하고, **자기 자신을 통해서만 접근하도록 제한**한다.

그런데 문제가 하나 생긴다.
**어떤 작업이 특정 Actor에서 실행되던 중, 다른 Actor의 컨텍스트에서 실행되어야 할 코드가 필요할 때가 있다.**
이때 발생하는 것이 바로 **Actor hop**이다.

> Actor hop이란?
> 현재 실행 중인 코드 컨텍스트(예: background thread)에서 → 다른 Actor(예: MainActor)로 제어를 넘기는 과정이다.

```swift
Task {
    await someMainActorFunction() // 여기서 hop 발생
}
```
이 hop은 비용이 존재한다.
따라서 **의도하지 않은 hop이 반복적으로 일어나면 성능 저하**로 이어질 수 있다.

---
## **5. 실무에서 주의할 점**

- 클로저 내부에서 UI 업데이트가 있을 경우, 반드시 @MainActor hop을 명시해야 한다.
- ViewModel이 @MainActor로 선언되어 있어도, 내부 클로저가 그 보장을 자동으로 물려받지 않는다.
- Combine이나 async/await을 사용할 때는 .receive(on:), MainActor.run 등으로 컨텍스트 전환을 명확히 해야 한다.
---

### **Combine이나 async/await에서의 컨텍스트 전환**

Combine에서는 .receive(on:)을 통해 명확히 어떤 스레드(또는 큐)에서 동작할지 지정해야 한다.
```swift
publisher
    .receive(on: DispatchQueue.main)
    .sink { value in
        self.label.text = value // ✅ 메인에서 실행됨
    }
```
- .subscribe(on:): 데이터 생성이 실행될 컨텍스트
- .receive(on:): 이후 연산자가 실행될 컨텍스트

만약 .receive(on:)을 쓰지 않고 백그라운드에서 sink를 처리하면, UI는 백그라운드에서 접근되어 크래시가 발생할 수 있다

----

### **2. async/await**
async/await에서는 다음과 같은 방법으로 컨텍스트 전환을 명시할 수 있다
```swift
Task {
    let result = await fetchData()
    
    await MainActor.run {
        self.label.text = result // 안전한 UI 업데이트
    }
}
```

여기서 MainActor.run은 내부 클로저를 MainActor의 컨텍스트로 hop시켜준다.
사실상 Task { @MainActor in ... }와 같은 의미지만, **더 명시적**이고 **중간에 작업 분기**를 넣기에 적합하다.

---
#### ViewModel이 @MainActor로 선언되어 있어도, 내부 클로저가 그 보장을 자동으로 물려받지 않는다

```swift
@MainActor
class ViewModel {
    func fetch() {
        service.load { result in
            self.data = result  // ❌ MainActor에서 실행된다고 보장할 수 없다
        }
    }
}
```
```swift
Task { @MainActor in
    self.data = result  // ✅ 안전한 전환
}
```

## **Finding & Synthesis**
- **Actor 간 컨텍스트 전환은 자동이 아니다.**
    클로저 내부에서는 @MainActor의 실행 보장이 유지되지 않으므로, 명시적인 hop이 필요하다.
- **클로저는 컨텍스트를 캡처하지 않는다.**
    Swift의 클로저는 생성 당시의 Actor context를 함께 저장하지 않기 때문에, 후속 작업에서 안전하게 실행된다는 보장이 없다.
- **Combine과 async/await 모두 명시적인 컨텍스트 전환을 요구한다.**
    .receive(on:), MainActor.run, Task { @MainActor in ... } 등을 통해 실행 환경을 명확히 지정해야 UI 크래시를 피할 수 있다.
- **Actor hop은 비용이 있다.**
    반복되는 불필요한 hop은 성능에 악영향을 줄 수 있으므로, 컨텍스트 설계를 명확히 하고 최소화하는 방향으로 접근해야 한다.

## GQ
- 클로저 내부에서 @MainActor로 선언된 객체의 프로퍼티를 직접 수정하면 어떤 문제가 발생할 수 있으며, 이를 방지하는 안전한 방법은 무엇일까

## Keywords
- [[클로저 (Closure)]]

#Onething 