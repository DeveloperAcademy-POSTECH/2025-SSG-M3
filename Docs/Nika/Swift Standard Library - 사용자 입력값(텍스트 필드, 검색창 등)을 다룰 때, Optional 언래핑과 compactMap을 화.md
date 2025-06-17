**GQ. 사용자 입력값(텍스트 필드, 검색창 등)을 다룰 때, Optional 언래핑과 compactMap을 활용하여 널값으로 인한 크래시를 방지하는 실전 패턴은 무엇인가?**


잘못된 방식(강제 언래핑은 크래시 유발 가능)

```swift
let text = textField.text!
```

만약 `textField.text`가 적절하지 않은 값(nil)인 경우:

Fatal error: Unexpectedly found nil while unwrapping an Optional value

안전한 언래핑

```swift
if let text = textField.text, !text.isEmpty {
    // text 사용 가능
}
```

```swift
guard let text = textField.text, !text.isEmpty else {
    // 에러 메시지 표시
    return
}
// text 사용 가능
```

여러 입력값을 한꺼번에 다룰 때 compactMap 사용(map과 달리 nil 자동으로 처리함)

```swift
let textFields: [UITextField] = [nameField, emailField, ageField]

let nonEmptyTexts = textFields
    .compactMap { $0.text?.trimmingCharacters(in: .whitespacesAndNewlines) }
    .filter { !$0.isEmpty }
```

- compactMap으로 nil 제거
- filter로 공백 문자열 제거

SwiftUI에서의 텍스트 필드 예시

```swift
@State private var input: String = ""

var body: some View {
    VStack {
        TextField("나이 입력", text: $input)
            .keyboardType(.numberPad)

        Button("확인") {
            if let age = Int(input.trimmingCharacters(in: .whitespaces)), age > 0 {
                print("유효한 나이: \\(age)")
            } else {
                print("입력 오류")
            }
        }
    }
}
```

#Nika 