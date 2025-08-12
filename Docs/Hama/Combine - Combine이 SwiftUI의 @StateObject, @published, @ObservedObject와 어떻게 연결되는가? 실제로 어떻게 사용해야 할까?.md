[!question]
>[!question]
>GQ1. Combine 이 뭘까?
>GQ2. @StateObject, @published, @ObservedObject 은 뭘까?
>GQ3. Combine이 SwiftUI의 @StateObject, @published, @ObservedObject 와 어떻게 연결되는가?
>GQ4. 실제로 어떻게 사용해야 할까?

→ Combine 기반 ViewModel과 SwiftUI 간의 데이터 흐름을 설계하고 연동하는 실무적인 질문.
## Description

- GQ1. Combine 이 뭘까?
데이터가 바뀌거나 이벤트가 발생했을 때, 시간축에 따라 'Publisher → Operator → Subscriber' 할 수 있게 해주는 반응형 프레임워크다.

```swift
import Combine

class ViewModel {
    @Published var name: String = "" // Publisher
    private var cancellables = Set<AnyCancellable>()

    init() {
        $name
            .map { $0.uppercased() } // Operator
            .sink { print("변경된 이름:", $0) } // Subscriber
            .store(in: &cancellables)
    }
}

let vm = ViewModel() // ViewModel 인스턴스를 vm 으로 생성
vm.name = "Hama"
// 출력: 변경된 이름: HAMA
```

* GQ2. @StateObject, @published, @ObservedObject 은 뭘까?
'데이터가 변하면 UI를 갱신하는' 흐름을 만드는 핵심 속성 래퍼들이다.

@StateObject → View가 ViewModel을 소유하고 있을 때.
@Published → ViewModel 안에서 변경 알림을 보내는 도구.
@ObservedObject → View가 다른 View의 ViewModel을 전달받아 관찰만 할 때.

* GQ3. Combine이 SwiftUI의 @StateObject, @published, @ObservedObject 와 어떻게 연결되는가?

| 개념                   | Combine와의 관계       | 구체적인 동작                                                                           |
| -------------------- | ------------------ | --------------------------------------------------------------------------------- |
| **@Published**       | Publisher 생성       | ObservableObject 안에서 프로퍼티에 붙이면, 값 변경 시 Combine Publisher 이벤트를 자동 발행               |
| **@StateObject**     | Publisher 구독 + 소유권 | View가 ObservableObject를 직접 생성하고, Combine의 objectWillChange를 구독해 UI를 자동 갱신         |
| **@ObservedObject**  | Publisher 구독       | View가 외부에서 전달받은 ObservableObject의 Combine Publisher를 구독해 UI를 갱신, 하지만 생성·유지는 하지 않음 |

* GQ4. 실제로 어떻게 사용해야 할까?
```swift
import SwiftUI
import Combine

class MyViewModel: ObservableObject {
    @Published var name: String = ""
}

struct ContentView: View {
    @StateObject var vm = MyViewModel() // View가 직접 ViewModel 소유
    
    var body: some View {
        VStack {
            Text("이름: \(vm.name)")
            Button("이름 변경") {
                vm.name = "Hama"
            }
        }
    }
}

struct ChildView: View {
    @ObservedObject var vm: MyViewModel // 다른 View에서 전달받아 관찰만 함
    
    var body: some View {
        Text("자식 뷰 이름: \(vm.name)")
    }
}
```

## Keywords


## 작성자
- #Hama 