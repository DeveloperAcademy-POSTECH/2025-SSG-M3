>[!question]
>GQ1. 데이터 바인딩이란 뭘까?
>GQ2. 클로저 기반 바인딩이란 뭘까?
>GQ3. 클로저 기반 바인딩을 사용한 예시는?
>GQ4. 클로저 기반 바인딩에서 왜 메모리 누수가 발생할까?
>GQ5. SwiftUI에서 클로저 기반 바인딩을 메모리 누수 없이 다루려면 어떤 구조가 필요할까?

## Description

* GQ1. 데이터 바인딩이란 뭘까?
	GA1-1. SwiftUI에서 데이터 바인딩(data binding)은 뷰(UI)와 모델(데이터) 간의 값을 연결하여 한쪽의 변경이 자동으로 다른 쪽에 반영되도록 하는 기술을 말한다. SwiftUI는 이러한 양방향 바인딩을 편리하게 처리하기 위해 여러 프로퍼티 래퍼(property wrapper)를 제공한다.
	GA1-2. 일반적으로 사용되는 바인딩 방식들은,
	(@State, @Binding, @ObservedObject, @EnvironmentObject) 가 있다.

- GQ2. 클로저 기반 바인딩이란 뭘까?
	GA2. 개발자는 때론, 일반적 바인딩 방식이 아닌 직접 바인딩 객체를 만들어야 하는 상황이 있다. 클로저 기반 바인딩은 Binding(init get:set:) 생성자를 사용하여 getter와 setter 클로저를 수동으로 넘겨줌으로써 바인딩을 생성하는 방식이다. 쉽게 말해, 프로퍼티 래퍼 없이 직접 Binding을 만들고 관리하는 방법을 말한다.

* GQ3. 클로저 기반 바인딩을 사용한 예시는?
	GA3. 클로저 기반 바인딩은 커스텀 동작이나 변환이 필요할 때 유용하다. 예를 들어, 두 개의 토글(Toggle) 스위치 중 오직 하나만 켤 수 있는 UI를 구현한다고 가정해보자.

```swift
struct ExclusiveTogglesView: View {
    @State private var firstToggle: Bool = false
    @State private var secondToggle: Bool = false

    var body: some View {
        // 각 토글에 대해 사용자 정의 Binding 생성
        let firstBinding = Binding<Bool>(
            get: { firstToggle },
            set: { newValue in
                firstToggle = newValue
                if newValue {
                    secondToggle = false // 첫번째 토글이 켜지면 두번째 토글을 끔
                }
            }
        )
        let secondBinding = Binding<Bool>(
            get: { secondToggle },
            set: { newValue in
                secondToggle = newValue
                if newValue {
                    firstToggle = false  // 두번째 토글이 켜지면 첫번째 토글을 끔
                }
            }
        )
        VStack {
            Toggle("첫번째 옵션", isOn: firstBinding)
            Toggle("두번째 옵션", isOn: secondBinding)
        }
    }
}
```

하나의 토글을 켜면 자동으로 다른 토글은 꺼지도록 하는 로직이 필요한데, SwiftUI에서는 이처럼 두 상태가 밀접하게 연관되어 있을 경우 직접 바인딩에 로직을 추가하면 편리하다.

* GQ4. 클로저 기반 바인딩에서 왜 메모리 누수가 발생할까?
	GA4. 구조를 잘못 설계하면, 메모리 누수가 발생한다.
```swift
struct MyView: View {
    @StateObject private var viewModel = MyViewModel()

    var body: some View {
        let customBinding = Binding<String>(
            get: {
                viewModel.text
            },
            set: { newValue in
                viewModel.text = newValue
            }
        )

        return TextField("입력", text: customBinding)
    }
}
```

잘못된 구조의 예시
1. viewModel은 @StateObject로 선언되어 SwiftUI가 소유하고 관리함.
2. 그런데 Binding(get:set:) 안의 클로저가 viewModel을 직접 참조하고 있음.
3. 이 Binding은 SwiftUI 뷰 구조 안에 보관됨. (TextField가 소유하게 됨)
4. 그런데 viewModel 은 MyView의 일부임.
5. MyView 는 또 그 Binding 을 만들게 됨.

결국, viewModel → Binding → 클로저 → viewModel
이라는 순환 고리(Retain Cycle)가 만들어진다.

* GQ4-2. 왜 문제일까?
	GA4-2.
	1. 참조 카운트가 0이 되지 않음.
	2. 객체가 메모리에서 해제되지 않음.
	3. 메모리 누수가 발생함.

* GQ5. SwiftUI에서 클로저 기반 바인딩을 메모리 누수 없이 다루려면 어떤 구조가 필요할까?
	GA5-1. weak viewModel 을 사용한다.
```swift
let customBinding = Binding<String>(
    get: { viewModel.text },
    set: { [weak viewModel] newValue in
        viewModel?.text = newValue
    }
)
```

GA5-2. 굳이 클로저 안 써도 될 때는 $viewModel.text 을 사용한다.
```swift
TextField("입력", text: $viewModel.text)
```
SwiftUI 는 $viewModel.text 처럼 자동 생성된 Binding 은 메모리 누수 없이 잘 관리해준다.
직접 Binding(get:set:) 를 쓰는 건 꼭 필요한 경우에만 사용하자.

## Keywords
+ [[클로저 (Closure)]]

## 작성자
- #Hama 