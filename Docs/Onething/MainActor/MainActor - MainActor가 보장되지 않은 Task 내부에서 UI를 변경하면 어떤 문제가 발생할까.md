
> [!question]
> **MainActor가 보장되지 않은 Task 내부에서 UI를 변경하면 어떤 문제가 발생할까**

## **개요**
Swift의 async/await와 Task {} 구문은 비동기 작업을 간결하게 표현할 수 있게 해준다.
하지만 UI 업데이트는 여전히 메인 스레드에서 안전하게 이루어져야 한다. Swift는 이를 보장하기 위해 @MainActor라는 개념을 도입했다. 
이 Actor는 UI와 관련된 작업이 메인 스레드에서 안전하게 실행되도록 보장해준다.

문제는 Task {}를 사용할 때, **메인 액터가 암묵적으로 보장되지 않는다는 사실**이다.

----
## **1. Task는 항상 MainActor에서 실행되지 않는다**

아래 코드처럼 단순한 Task는 Global Executor에서 실행될 수도 있다.
```swift
Task {
  label.text = "업데이트 완료" // ❗️여기서 문제가 발생할 수 있다
}
```
Swift는 내부적으로 Task를 실행할 때, **Actor 컨텍스트를 명시적으로 지정하지 않으면** 어떤 스레드에서 실행될지 보장하지 않는다.

즉, 위의 label 업데이트는 메인 스레드가 아닐 수 있고, 이로 인해 다음과 같은 문제가 생길 수 있다.

---

## **2. 발생 가능한 문제**
- **UI 크래시**
    - UIKit이나 SwiftUI는 메인 스레드에서 UI를 변경해야 한다. 다른 스레드에서 접근하면 런타임 에러 또는 앱이 뻗을 수 있다.
- **레이아웃 이상**
    - UI 업데이트가 예기치 않은 시점에 발생하면, 레이아웃이 깨지거나 스크롤 위치가 어긋날 수 있다.
- **디버깅 난이도 상승**
    - 재현이 어렵고 intermittent한 문제로 나타난다. Xcode에서 스레드 체크 경고가 나와도 무시되기 쉬움.

---

## **3. 안전한 해결 방법**
Task를 사용할 때는 **명시적으로 MainActor를 지정**해주는 것이 좋다.
```swift
@MainActor
func updateUI() {
  label.text = "완료"
}

Task { @MainActor in
  updateUI()
}
```
혹은 메서드 자체에 @MainActor를 붙여, 호출하는 쪽에서 Actor hop이 발생하도록 만드는 것도 좋은 방법이다.

#### **Actor Hop?**
**Actor hop**이란 **현재 실행 중인 코드의 컨텍스트(Actor)가 다른 Actor로 이동(hop)해야 할 때 발생하는 컨텍스트 전환**을 말한다.

즉, 어떤 작업이 특정 Actor의 격리(context) 안에서 실행되고 있는데, 다른 Actor에 속한 메서드나 프로퍼티를 호출해야 할 때, **Swift가 내부적으로 실행 컨텍스트를 전환하는 과정**을 뜻한다.

#### 왜 중요할까?
Actor는 **데이터 경쟁(race condition)을 막기 위해 자신만의 실행 격리 공간을 가진다.**
이런 격리를 유지하기 위해 Swift는 **다른 Actor의 메서드를 호출할 때 해당 Actor의 큐로 hop**한다.

```swift
actor DataStore {
    func fetch() async -> String {
        return "data"
    }
}

@MainActor
func updateUI(with data: String) {
    print("UI updated with \(data)")
}

func loadData(store: DataStore) async {
    let data = await store.fetch()     // 👉 여기서 DataStore Actor로 hop
    await updateUI(with: data)         // 👉 다시 MainActor로 hop
}
```
위 코드에서는 store.fetch()를 호출할 때, 현재 Task는 DataStore라는 Actor로 hop하고, 이후 updateUI 호출 시에는 다시 MainActor로 hop한다.

#### **Actor hop의 비용**
- Actor hop은 **비동기 컨텍스트 전환**이므로 **비용이 존재**한다.
- 너무 자주 hop하면 성능에 영향을 줄 수 있다.
- Swift는 가능한 한 **최소한의 hop을 유도하도록 컴파일 최적화**를 진행한다.

#### **실무에서 주의할 점**
- **불필요한 hop을 피하려면**, 같은 Actor 내부에서 처리 가능한 로직은 묶어서 처리하는 것이 좋다.
- 클로저 내에서 @MainActor를 명시하지 않고 UI 업데이트를 하면, **암묵적인 hop**이 일어나는데, 이로 인해 예상치 못한 지연이나 경합이 생길 수 있다.

---

## **4. 실무에서의 판단 기준**

| **상황**            | **권장 방식**                      |
| ----------------- | ------------------------------ |
| 네트워크 응답 후 UI 업데이트 | Task { @MainActor in ... } 사용  |
| 버튼 탭 등 UI 이벤트 처리  | ViewModel 메서드에 @MainActor 명시   |
| 내부 로직 중 일부만 UI 접근 | await MainActor.run { ... } 사용 |

---
## **5. Swift의 Actor 모델과 UI의 관계**

Swift Concurrency에서 Actor는 **동시성 격리**를 제공한다.
@MainActor는 메인 스레드라는 단일 실행 컨텍스트에 접근하는 UI Actor라고 볼 수 있다.

즉, UI와 관련된 모든 작업은 @MainActor의 보호 아래 수행되어야 한다는 암묵적 규칙이 있다.


## **Finding & Synthesis**
- Task는 기본적으로 어떤 Actor에도 속하지 않기 때문에 UI를 안전하게 다루려면 MainActor를 명시해야 한다.
- UIKit이나 SwiftUI는 thread-safe하지 않으며, 잘못된 쓰레드에서의 UI 접근은 크래시나 버그의 원인이 된다.
- @MainActor, MainActor.run {} 등의 문법을 통해 안전하게 UI를 제어할 수 있다.
- 실무에서는 Task 내 비동기 로직 이후 UI 변경이 있다면 항상 컨텍스트 전환을 고려해야 한다.

## **GQ**
- Swift의 MainActor.run은 어떤 방식으로 기존 실행 컨텍스트를 전환하고, 내부적으로 어떤 queue를 사용하는가?

## Keywords
- [[MainActor]]

#Onething 