>[!question]
>GQ1. MainActor 가 뭘까?
>GQ2. RunLoop 가 뭘까?
>GQ3. MainActor 와 RunLoop 는 왜 상호작용을 해야하는가?
>GQ4. MainActor는 RunLoop와 어떤 방식으로 상호작용하는가

## Description

- GQ1. MainActor 가 뭘까?
	GA1. MainActor 는 Swift-Concurrency 에서 메인 스레드와 연관된 Actor 다. 다시 말해, MainActor 로 격리된 코드는 메인 스레드(UI 스레드)에서만 실행되도록 한다.
	GA1-2. Actor는 여러 작업(Task)을 동시에 실행할 수 있는 능력을 안전하게 다루기 위한 구조체다. MainActor 를 사용하면 UI와 관련된 데이터 흐름을 메인스레드로 일원화하여 데이터 레이스나 스레드 충돌 없이 안전하게 업데이트할 수 있다.
	만약, 다른 스레드(예: 백그라운드 스레드)에서 UI를 변경하면, 앱이 오작동하거나 충돌할 수 있다.

```swift
@MainActor
class Photos: ObservableObject {
    @Published private(set) var items: [SpacePhoto] = []
    func updateItems() async {
        let fetched = await fetchPhotos()
        items = fetched
    }
}
```

메인 스레드는 UI 작업을 전담하는 앱의 중심 실행 흐름이다. UI는 반드시 메인 스레드에서 조작해야 안정적인데, MainActor는 이런 규칙을 자동으로 지켜주게 만들어주는 도구다.

- GQ2. RunLoop 가 뭘까?
	GA2. RunLoop 는 이벤트가 발생할 때까지 기다렸다가, 이벤트가 발생하면 그걸 처리하는 반복 구조를 가진 객체다.
	RunLoop의 핵심 목적은 해당 스레드에 처리할 작업이 있을 때 스레드를 계속 바쁘게 동작시키고, 처리할 작업이 없을 때에는 스레드를 휴면(대기) 상태로 전환함으로써 불필요한 리소스 소모를 막는 것에 있다.

* GQ3. MainActor 와 RunLoop 는 왜 상호작용을 해야하는가?
	GA3. 만약, 상호작용하지 않는다고 해보면 다음과 같다. 메인 스레드에서 RunLoop가 돌지 않으면, 터치 이벤트도 받지 않고, MainActor 가 예약한 UI 업데이트도 실행되지 않는다. 즉, 앱이 멈춘 것처럼 보이게 된다.
	이 둘의 관계는 다시말해, MainActor 가 보낸 UI 작업을, RunLoop 가 실행하는 관계다.

* GQ4. MainActor는 RunLoop와 어떤 방식으로 상호작용하는가?

```swift
@MainActor
func updateLabel() {
    label.text = "Hello"
}
```

이 코드는,
1. @MainActor 으로 인해 메인 스레드에 작업이 예약됨.
2.  메인 스레드의 RunLoop가 돌아가면서 그 예약된 작업을 실행함.
3. 그러면, label.text = "Hello"는 UI가 안전하게 업데이트됨.

## Keywords
+ [[MainActor]]

## 작성자
- #Hama 