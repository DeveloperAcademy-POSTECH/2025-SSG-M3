
>[!question]
>GQ1. Swift에서 바인딩은 어떤 개념일까?

바인딩에 대해 진지하게 알아보는 시간을 갖자. 컴바인을 공부하기 전에 바인딩을 먼저 공부했어야 했던 것 같다.

### 1. 개요

바인딩(Binding): 데이터와 UI를 실시간+양방향으로 연결하는 것.
간단히 말해 데이터가 바뀌면 UI가 바뀌고 UI가 바뀌면 데이터가 바뀌도록 하는 연결 고리이다.

### 2. 왜 바인딩이 필요할까?

SwiftUI는 선언형 UI 프레임워크다.
즉 UI는 상태(State)를 기준으로 그려진다.

```swift
	@State private var isOn = false
```
	
		여기서 `isOn`이라는 불리언 값이 true면 스위치가 켜지고 false면 꺼지는 UI처럼
		SwiftUI에서는 데이터 값이 바뀌면 그에 맞춰 화면이 자동으로 다시 그려지도록 설계되어 있다.
		이걸 "상태 주도형 UI"라고 한다.

하지만 자식 뷰에서 부모 뷰의 상태를 직접 수정할 수는 없다.
SwiftUI에서는 데이터의 흐름을 **부모 → 자식** 방향으로만 흐르도록 명확히 설계했기 때문이다.

```swift
	struct ParentView: View {
		@State private var isOn = false
	
		var body: some View {
			ChildView(isOn: isOn) // 값 넘김
		}
	}
```
		여기서 ChildView는 isOn 값을 읽기만 할 수 있고 직접 수정할 수는 없다.
		State는 소유자만 수정 가능하기 때문!

		"부모의 데이터는 부모만 바꿀 수 있다"
		이건 SwiftUI가 데이터 일관성을 지키기 위해 정한 엄격한 규칙이라고 한다.

SwiftUI는 이런 상황에서 `@Binding` 이라는 해결책을 준다.
**바인딩을 통해 부모 상태의 "참조"를 전달하면 자식 뷰도 해당 값을 읽고 쓸 수 있도록** 한 거다.

```swift
	struct ParentView: View {
		@State private var isOn = false
	
		var body: some View {
			ChildView(isOn: $isOn) // 바인딩 넘김
		}
	}
```

		`$isOn`은 바인딩 타입의 참조값이다.
		즉 isOn이라는 값 자체가 아니라 그 값을 읽고 쓸 수 있는 창구를 전달하는 것

		그리고 자식 뷰에서는 이렇게 받는다.

```swift
	struct ChildView: View {
		@Binding var isOn: Bool
	
		var body: some View {
			Toggle("스위치", isOn: $isOn)
		}
	}
```

		이제 자식 뷰에서 스위치를 토글하면 그 변화가 직접 부모의 상태 값에 반영된다.
		바인딩을 통해 부모의 값을 읽고 쓰는 권한을 자식에게 위임한 셈.

### 3. 바인딩이 없었다면?

앞서 본 것처럼 부모 뷰의 `@State` 값을 자식 뷰에 단순히 전달하면 자식은 그 값을 읽기만 할 수 있고 쓸 수는 없다. 자식 뷰 안의 UI가 사용자의 입력에 따라 상태를 바꾸고 싶어도 그 값을 직접 수정할 방법이 없는 것.

그럼 상태 수정을 어떻게 할 수 있을까?


1. 콜백 클로저로 간접 수정하기

```swift
struct ParentView: View {
	@State private var isOn = false

	var body: some View {
		ChildView(
			isOn: isOn, // 현재 상태를 값으로 전달(읽기 전용)
			toggleAction: {
				isOn.toggle() // 여기서는 isOn을 바꿀 수 있음
				// 값을 바꿔달라고 부모에게 요청할 수 있는 버튼을 넘겨주는 셈
			}
		)
	}
}

struct ChildView: View {
	var isOn: Bool
	var toggleAction: () -> Void

	var body: some View {
		Toggle("스위치", isOn: .constant(isOn))
			.onTapGesture {
				toggleAction()
			}
	}
}
```

가능하긴 하지만 많은 단점이 있는 구조다.
- isOn 값은 읽기 전용이니 .constant(isOn)처럼 강제 변환해야 한다.
- 사용자가 상태를 바꿔도 화면이 다시 그려지지 않거나 잘못된 값이 표시될 수 있다.
- UI 동작과 상태 관리가 명확히 분리되지 않는다.


2. 자식이 부모에게 상태 변화를 요청하기

즉 바인딩 없이 상태를 바꾸려면 자식이 매번 부모에게 "나 이 값 바꾸고 싶어!"라고 요청하고,
부모는 그 요청을 받고 스스로 값을 바꿔줘야 한다. (복잡한 상태 전달 로직)
반면 바인딩을 쓰면 자식이 직접 상태를 조작할 수 있는 권한과 수단을 갖게 된다.


이런 경우 여러 자식 뷰가 동일한 상태를 공유하게 되면 매우 복잡한 흐름이 되어버리는 것. 유지보수 지옥!!

### 4. 바인딩의 구조

|**역할**|**설명**|
|---|---|
|@State|상태를 정의하는 쪽 (보통 부모 뷰)|
|@Binding|상태를 참조하는 쪽 (보통 자식 뷰)|
|$|바인딩 값을 꺼낼 때 사용하는 접두어|

### 5. 바인딩 사용 흐름

`@State`는 상태의 소유자
`$(변수명)`은 해당 상태에 대한 바인딩(참조)를 만들고
자식 뷰는 `@Binding`을 통해 상태를 읽고 수정할 수 있다.

|**단계**|**설명**|
|---|---|
|상태 선언|부모가 @State로 상태 선언|
|바인딩 생성|$를 붙여 바인딩 값 생성|
|전달|바인딩 값을 자식 뷰에 전달|
|참조 선언|자식은 @Binding으로 값 받기|
|사용|자식 뷰에서 읽고 쓰기 가능 (UI 바인딩)|

이렇게 사용하면,

사용자가 자식 뷰에서 UI를 조작할 때
→ @Binding이 연결된 값을 바꾸고
→ 바뀐 값이 **실제로는 부모 뷰의 @State 값**이기 때문에
→ 부모의 상태가 바뀌고 뷰 전체가 업데이트된다!

### 6. 바인딩 활용 예시(실전에서 자주 쓰이는 상황)

```swift
struct ContentView: View {
    @State private var name = ""

    var body: some View {
        VStack {
            TextField("이름을 입력하세요", text: $name)
            GreetingView(name: $name)
        }
    }
}

struct GreetingView: View {
    @Binding var name: String

    var body: some View {
        Text("안녕하세요, \(name)님!")
    }
}
```

- TextField, Toggle, Slider 등 사용자 입력을 처리할 때
- Sheet, NavigationLink, Alert의 표시 여부
- 재사용 컴포넌트에서 부모의 상태를 제어해야 할 때 등등

### 8. `@State`?

일반적으로 바인딩은 `@State` 와 함께 쓰이지만 꼭 그렇지는 않다.
바인딩은 단지 "값을 읽고 쓰는 창구(참조)"이기 때문에 Binding 타입만 만족하면 `@State`가 아니어도 바인딩이 가능하다. `@Published, @FocusState, @SceneStorage` 등

대표적인 예로 ViewModel의 `@Published` 값과의 바인딩이 있다.
```swift
class MyViewModel: ObservableObject {
    @Published var text: String = ""
}
```

```swift
struct ContentView: View {
    @StateObject private var viewModel = MyViewModel()

    var body: some View {
        ChildView(text: $viewModel.text) // 바인딩 전달
    }
}
```

```swift
struct ChildView: View {
    @Binding var text: String

    var body: some View {
        TextField("입력하세요", text: $text)
    }
}
```

`viewModel.text`는 `@Published`로 선언되어 있지만 `$viewModel.text`라고 하면 SwiftUI가 자동으로 바인딩 객체로 변환해준다.
자식 뷰에서는 이 값을 읽고 쓸 수 있게 되고 이 변화는 다시 뷰모델에 반영된다.

### 9. 요약

SwiftUI에서의 바인딩은 UI와 상태 간의 **양방향 연결**을 통해 **데이터 중심의 UI 선언을 가능하게 한다**.

@State로 소유된 값을 $를 통해 @Binding으로 전달함으로써, 자식 뷰에서도 상태를 실시간으로 반영하고 조작할 수 있게 만든다.

#### Keyword

- [[바인딩 (Binding)]]


#Noter