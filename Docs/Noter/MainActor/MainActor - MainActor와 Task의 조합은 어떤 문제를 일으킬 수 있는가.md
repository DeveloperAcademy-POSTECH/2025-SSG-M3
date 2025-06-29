
> [!question]
> GQ. MainActor와 Task의 조합은 어떤 문제를 일으킬 수 있는가

---
### **0. 키워드 개념 정리**

**1) MainActor**
- 특정 코드가 메인 스레드(UI 스레드)에서 실행되도록 보장해주는 액터
- UI 업데이트처럼 스레드 안정성이 필요한 작업을 보호하기 위해 사용
- 메서드나 타입에 @MainActor를 붙이면 해당 코드는 항상 메인 스레드에서 실행됨

**2) Task**
- Swift의 **비동기 작업 실행 컨테이너**, 비동기 작업을 실행하는 기본 단위임
- Task { ... }로 비동기 코드를 새로 실행, 즉 메인 코드 흐름과 별도로 백그라운드에서 작업을 시작함

왜 `Task`가 필요할까?

예를 들어 UI 이벤트가 발생했을 때 무거운 작업(네트워크 작업, 디스크 읽기 등)을 하려면 메인 스레드를 차단하지 않아야 한다. 이럴 때 Task를 쓰면 메인 스레드는 계속 돌아가고, 무거운 작업은 비동기적으로 처리할 수 있다.
```swift
Button("불러오기") {
    Task {
        let data = await fetchData()
        print(data)
    }
}
```
여기서 `fetchData()`가 오래 걸리는 네트워크 요청이어도 UI는 멈추지 않고 응답한다.

`Task`는 어디에서 실행될까?
```swift
Task {
    print(Thread.isMainThread) // false인 경우가 많음
}
```
기본적으로 `Task`는 메인스레드가 아닌 다른 스레드에서 실행된다. 성능을 위해 의도된 기본 설정이다.
그래서 이 안에서 UI를 다룰 땐 반드시 MainActor를 명시해야 한다.

요렇게
```swift
Task { @MainActor in 
    // 이 블록은 메인 스레드에서 실행됨
    self.title = "업데이트 완료"
}
```

- @MainActor는 이 블록이 **UI를 안전하게 변경할 수 있는 영역**임을 나타낸다.
- SwiftUI나 UIKit에선 UI는 무조건 메인 스레드에서만 수정해야 하니까 필수.

→ **UI 상태 변경, 애니메이션, 바인딩 등은 꼭 MainActor 안에서 해야 안정적이다!**

**3) MainActor와 Task의 조합**
- @MainActor 함수 내에서 다시 Task {}를 만들거나
- Task { @MainActor in } 형태를 쓰면
    → **의도치 않게 스레드 hop**이 발생하거나 **순서 보장 문제** 발생 가능

### **1. 개요**

Swift의 @MainActor는 UI 코드가 항상 **메인 스레드에서 안전하게 실행되도록** 해준다.

그런데 이 MainActor와 Task를 조합할 때는 실행 순서, 위치, 컨텍스트 전환(hopping)에 주의하지 않으면 예기치 못한 동작이나 버그, 성능 저하, 데드락 등이 발생할 수 있다.

특히 문제는 다음과 같은 상황에서 잘 드러난다:

- @MainActor context 안에서 다시 Task {}를 호출할 때
- Task { @MainActor in ... } 안에서 MainActor를 중복해서 요청할 때
- Task를 썼지만 UI 작업이 **MainActor로 hop하지 않아** UI 업데이트가 무시될 때

### **2. MainActor와 Task의 조합은 어떤 문제를 일으킬까?**

| **문제**                     | **설명**                                                                         |
| -------------------------- | ------------------------------------------------------------------------------ |
| **의도치 않은<br>MainActor 탈출** | Task는 기본적으로 메인 스레드 밖에서 실행되므로, @MainActor context를 **벗어나는 hop**이 발생할 수 있음       |
| **Double hop(이중 컨텍스트 전환)** | @MainActor 안에서 Task { @MainActor in ... } → 메인 → 백그라운드 → 다시 메인으로 hop → 오버헤드 발생 |
| **순서 역전**                  | UI 업데이트 코드가 Task에 감싸지면서 **비동기 큐에 나중에 등록**되어 실행 순서가 꼬일 수 있음                     |
| **잠재적 데드락**                | MainActor가 락을 걸고 있는데, 내부에서 또다시 MainActor 작업을 요청 → **교착 상태 가능성**                |

### **3. 예시 코드로 이해하기**

```swift
@MainActor
func updateUI() {
    print("UI 업데이트 시작") // 메인 스레드

    Task {
        print("여긴 어디?") // 기본 Task는 메인 스레드가 아님!
        await updateModel()
    }

    print("UI 업데이트 끝")
}
```

이 경우 Task는 @MainActor 컨텍스트 안에서 실행되었지만, 내부는 **백그라운드에서 실행됨**
→ updateModel()가 MainActor를 필요로 하면 다시 hop 발생
→ 이중 컨텍스트 전환 + 순서 꼬임 가능

### **4. 안전하게 쓰려면?**

**선택지 1: Task를 아예 @MainActor context 밖에서 사용**

```swift
func fetchAndUpdate() {
    Task { @MainActor in
        await updateUI()
    }
}
```

→ UI 업데이트는 **명확히 메인 스레드에서 실행됨**

 **선택지 2: @MainActor 함수 안에서 Task 대신 await**
```swift
@MainActor
func safeUpdate() {
    print("Start")
    await updateModel() // 바로 호출, 같은 MainActor이므로 hop 안 일어남
    print("End")
}
```
→ Task로 감싸지 않고 MainActor 내에서 곧바로 비동기 호출

### **5. 실제 발생 가능한 시나리오 예시**

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var title = ""

    func loadTitle() {
        Task {
	        // 이 블록은 기본적으로 백그라운드에서 실행됨
            // 이 시점에서 self는 MainActor가 아님
            let result = await fetchTitle()
            title = result // UI 업데이트 → crash 가능성
        }
    }
}
```
→ 해결:
1) Task 자체에 `@MainActor` 붙이기
```swift
func loadTitle() {
    Task { @MainActor in
        let result = await fetchTitle()
        title = result
    }
}
```
or
2) loadTitle 함수에 `@MainActor` + `async` 명시 
```swift
@MainActor
func loadTitle() async {
    let result = await fetchTitle()
    title = result
}
```
or
`Task { await self.loadtitle() }` 로 바깥에서 호출

### **6. 정리**

- MainActor는 **UI 안전성**을 보장하지만, Task와 함께 사용할 땐 실행 컨텍스트 전환(hop)을 정확히 이해해야 한다.
- @MainActor context 안에서 Task {}를 만들면 오히려 **컨텍스트를 이탈**할 수 있다.
- 불필요한 hop을 줄이고, 필요한 곳에만 명시적으로 @MainActor를 선언하는 것이 안전하다.

---
### **Keyword**

- [[MainActor]]
- [[병렬 처리(Swift-Concurrency)]]

### **작성자**

#Noter