>[!question]
>GQ. ObservableObject를 MainActor와 함께 사용할 때 스레드 안전성은 어떻게 확보할 수 있을까

## Description
- SwiftUI의 뷰는 상태 변화에 따라 자동으로 업데이트된다.
- 이런 반응형 동작을 가능하게 하는 핵심요소는 `ObservableObject` (및 `@Observable`)이다.
- SwiftUI에서 모든 UI 관련 작업은 메인 스레드에서만 수행되어야 한다.
- 백그라운드 스레드에서 UI 요소나 이를 구동하는 `ObservableObject`의 `@Published` 속성을 직접 업데이트하려고 시도하면 "보라색 런타임 에러"가 발생한다.
- `MainActor`는 특정 코드가 메인 스레드에서만 실행되도록 컴파일 시점에 보장하는 전역 액터이다.

## 주요 기능
+ SwiftUI 및 UIKit는 Thread Safety 하지 않으므로, 모든 UI 업데이트는 반드시 메인 스레드에서 이루어져야 한다.
+ 따라서 `ObservableObject`에 `MainActor`를 사용하여 메인 스레드에서만 실행되도록 보장한다.
+ 클래스, 함수, 속성에 적용할 수 있고, 전체 클래스에 적용하면 모든 멤버가 메인 액터에 격리된다.

## 코드 예시
#### 잘못된 예 (보라색 에러 발생)
```Swift
class MyViewModel: ObservableObject {
    @Published var message: String = "초기 메시지"

    func fetchMessageFromBackground() {
        DispatchQueue.global().async {
            let fetchedMessage = "백그라운드에서 가져온 메시지"
            self.message = fetchedMessage // "Publishing changes from background thread" 보라색 런타임 문제 발생
        }
    }
}

struct MyView: View {
    @StateObject var viewModel = MyViewModel()

    var body: some View {
        VStack {
            Text(viewModel.message)
            Button("메시지 가져오기 (잘못된 방식)") {
                viewModel.fetchMessageFromBackground()
            }
        }
    }
}
```
#### MainActor 사용한 올바른 예
```Swift
@MainActor // ViewModel 전체를 MainActor에 격리
class MyViewModel: ObservableObject {
    @Published var message: String = "초기 메시지"
    @Published var isLoading: Bool = false

    func fetchMessageFromBackgroundSafe() async {
        isLoading = true
        let fetchedMessage = await Task.detached {
            try? await Task.sleep(nanoseconds: 2_000_000_000)
            return "백그라운드에서 안전하게 가져온 메시지"
        }.value

        // ViewModel이 @MainActor에 격리되어 있으므로,
        // 이 속성 업데이트는 자동으로 메인 스레드에서 이루어짐
        self.message = fetchedMessage?? "데이터 로드 실패"
        isLoading = false
    }
}

struct MyView: View {
    @StateObject var viewModel = MyViewModel()

    var body: some View {
        VStack {
            if viewModel.isLoading {
                ProgressView("메시지 로드 중...")
            } else {
                Text(viewModel.message)
            }
            Button("메시지 가져오기 (안전한 방식)") {
                Task {
                    await viewModel.fetchMessageFromBackgroundSafe()
                }
            }
        }
    }
}
```

## Keywords
+ [[MainActor]]
+ [[ObservableObject]]
+ [[병렬 처리(Swift-Concurrency)]]

## 작성자
- #Air 