>[!question]
>GQ. Voice Memo 앱의 녹음 버튼 애니메이션은 어떻게 구현할까?

## Description
- 애플의 기본 앱인 Voice Memo 앱의 녹음 버튼에는 탭했을 때 원에서 둥근 사각형으로 부드럽게 변하는 애니메이션이 적용되어 있다. 이를 구현해보면서 SwiftUI animation의 사용법과 기본 작동원리를 이해해보고자 한다.

## 코드 예시
```Swift
import SwiftUI

struct RecordButton: View {
    @Binding var isRecording: Bool
    private let buttonSize: CGFloat = 60
    var body: some View {
        Button(action: {
            withAnimation(.spring(duration: 0.3)) {
                isRecording.toggle()
            }
        }) {
            ZStack {
                Circle()
                   .strokeBorder(Color.gray.opacity(0.7), lineWidth: 3)
                   .frame(width: buttonSize + 10, height: buttonSize + 10)

                RoundedRectangle(cornerRadius: isRecording ? 10 : buttonSize / 2)
                   .fill(Color.red)
                   .frame(width: buttonSize, height: buttonSize)
                   .scaleEffect(isRecording ? 0.5 : 1.0)
            }
        }
    }
} 

struct ContentView: View {
    @State private var isRecording = false
    var body: some View {
        VStack {
            Spacer()
            RecordButton(isRecording: $isRecording)
            Spacer()
            Text(isRecording ? "녹음 중..." : "녹음 시작")
               .font(.title)
               .bold()
               .padding()
        }
       .padding()
    }
}
```

`withAnimation { ... }` 클로저를 사용하여 상태 변경 코드를 감쌌다. 명시적 애니메이션이라고 하며 이 클로저 내에서 변경한 상태가 이에 관련된 모든 뷰에 애니메이션이 적용된다.

## Keywords
+ [[SwiftUI]]
+ [[animation]]

## References
- [SwiftUI Animations (공식 문서)](https://developer.apple.com/documentation/swiftui/animations)

## 작성자
- #Air