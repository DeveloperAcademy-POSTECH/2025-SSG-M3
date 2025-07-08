>[!question]
>[!question]
>GQ1. 타이머(Timer), 알림센터(NotificationCenter), delegate 등에서 발생할 수 있는 메모리 누수 상황은 어떤 것이 있으며, 실무에서 어떻게 방지하는가?

## Description
- 1️⃣ 메모리 누수는 왜 생길까? 
		- 강한 순환 참조 (strong reference cycle) 때문. 
		- 예를 들어, Timer.scheduledTimer 는 타겟을 강하게 잡음
		- NotificationCenter는 등록한 Observer 를 강하게 참조
		- delegate 프로퍼티를 strong 으로 선언하면, 위임한 객체와 서로 강한 참조를 가지면서 순환이 생김 
		- > 결국 이 참조 고리가 끊기지 않으면 객체가 해제되지 않아 메모리가 점점 쌓임. 
		
- 2️⃣ 실무에서 어떻게 방지? 
	- Timer
		- `Timer` 는 `scheduledTimer`로 만들면 기본적을 타겟을 Strong 으로 유지
		- `weak self` 캡처를 사용하거나, `invalidate()` 로 적절한 시점에 해제해야 한다. 

	- NotificationCenter
		- 등록한 옵저버는 해제하기 전에 removeObserver를 호출하거나 IOS 9 이후 지원되는 `NotificationCenter.addObserver(forName:object:queue:using:)` 를 사용하고, 클로저 안에서 `weak self` 로 캡처

	- Delegate 
		-  델리게이트 프로퍼티는 항상 `weak` 이나 `unowned` 로 선언
```
weak var delegate: SomeDelegate?
```

 
## 코드 예시
+ Timer에서 메모리 누수가 되는 상황
```swift 
import SwiftUI

struct LeakingTimerView: View {
	@StateObject private var timerHolder = TimerHolder()
	var body: some View {
		VStack{
			Text("Timer count: \(timerHolder.counter)")
		}
	}
}

final class TimerHolder: ObservableObject {
	@Published var counter = 0
	var timer: Timer?

	init(){
		//여기서 누수 발생
		timer = Timer.scheduledTimer(withTimeInterval: 1.0,repeats: true){
		 _ in self.counter += 1
		 }
	}
	deinit {
		print("timeHolder deinit")
		timer?.invalidate()
	}
}
```

- 여기서 `Timer` 가 self를 강하게 참조, `TimerHolder`는 `Timer`를 프로퍼티로 강하게 소유 = 강한 순환 
- `deinit` 이 호출되지 않음. -> 메모리 누수

- 해결방법
```swift
timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true){
[weak self] _ in self?.counter += 1
}
//필요없는 시점에 timer?.invalidate()를 반드시 호출해 끊어 줘야 함. 
```

- 3️⃣ XCode 에서 메모리 누수 확인하기 
- 실행 후 Product > Profile > Leaks or Allocations 
- View 에서 LeakingTimerView를 여러번 띄웠다 닫으면 deinit 로그가 출력되지 않거나 instruments 에서 객체가 해체되지 않고, 계속 쌓이면 메모리 누수가 의심됨. 
## Keywords
- [[ARC]]

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- #Zhen 