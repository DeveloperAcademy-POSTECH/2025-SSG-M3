

> [!question]
>연관값이 클로저, 뷰, 모델 타입일 때 주의해야 할 점은 뭘까?


# 연관값(enum associated value)에 클로저, 뷰, 모델을 넣으면 무슨 일이 일어날까?

Swift 의 열거형(enum)은 단순한 상태 표현을 넘어서, **값을 함께 저장할 수 있는 기능**을 가지고 있다.
여기에 종종 클로저, 뷰, 모델 객체 같은 복잡한 타입도 넣곤 한다.

하지만,
이런 타임을 연관값으로 사용할 떄는 몇가지 중요한 주의점이 있다.
무심코 넣었다가 **메모리 문제나 의도치 않은 사이드 이펙트**를 겪을 수도 있기 때문이다.

## 🔥 1. 클로저를 연관값으로 쓸 때 주의할 점
클로저는 **캡처(capture)**라는 특성이 있다.
즉, 주변의 값을 참조하고 메모리에 보관.

```swift
enum ButtonAction {
    case tapped(() -> Void)
}
```

이렇게 클로저를 enum에 넣고,
그 안에서 self를 캡처하고 있다면

```swift
case .tapped({ self.doSomething() }) // 캡처됨
```

이 클로저가 강하게 참조하는 self와
그 self 가 또 enum을 소유하고 있다면?
→  **강한 순환 참조(Strong Reference Cycle)** 가 생겨 메모리 누수로 이어질 수 있다.

---
#### 강한 순환 참조란?
Swift의 class나 closure는 참조 타입이다.
즉, 하나의 인스턴스를 여러 객체가 "참조"하고 있을 수도 있고,
참조가 계속 살아 있는 한, 메모리에서 해제되지 않는다.

> 만약 A가 B를 참조하고, B도 A를 참조하면?
> 서로를 붙잡고 있어서 **누군가 해제하지 않으면 메모리에서 사라지지 않음** → **Memory Leak**

---
####  **Enum과 클로저에서 왜 문제가 될까?**

```swift
class ViewModel {
    enum State {
        case loading
        case loaded(() -> Void)
    }

    var state: State?

    func load() {
        // self를 클로저에서 강하게 캡처하고 있어요
        state = .loaded({
            self.doSomething()
        })
    }

    func doSomething() {
        print("Doing something")
    }
}
```

1. ViewModel이 state라는 프로퍼티로 enum을 **소유**.
2. state의 연관값인 클로저는 self(ViewModel)을 캡처한다.

> 즉, ViewModel → State(.loaded) → 클로저 → ViewModel
> 이렇게 **참조 고리가 닫혀버림** = 강한 순환 참조 🔁

그래서 ViewModel을 더 이상 사용하지 않아도, 메모리에 남아 있게 돼요.
 → 메모리 누수


### **해결법 ①:** **[weak self] 캡처 리스트 사용**

```Swift
state = .loaded { [weak self] in
    self?.doSomething()
}
```

여기서 [weak self]는 self를 **약한 참조(weak reference)** 로 바꾼다.
이렇게 하면 self가 먼저 해제돼도 참조가 깨지면서 고리를 끊을 수 있다.

여기서 주의할 점
- self가 없어졌을 수도 있기 때문에 self?처럼 **옵셔널 체이닝**으로 접근해야함.
- 내부에서 self를 강하게 다시 잡지 않도록 조심.

---

### ** 해결법 ②: 클로저 외부에서 의존성 주입**

클로저 안에서 self의 함수를 직접 호출하지 말고,
**클로저가 외부에서 필요한 동작을 주입받도록** 만들 수 있다.

```swift
func load(onLoaded: @escaping () -> Void) {
    state = .loaded(onLoaded)
}
```

```swift
// View.swift

struct MyView: View {

    @StateObject var viewModel = MyViewModel()

  

    var body: some View {

        VStack {

            Text("Hello")

  

            Button("Load") {

                // ✅ 외부에서 클로저를 직접 정의하고 넘겨줌

                viewModel.load {

                    print("Loaded from View")

                }x

            }

        }

    }
```

→ 이렇게 하면 ViewModel 내부에 클로저가 들어가지 않기 때문에 순환 참조가 아예 생기지 않아,
**역할을 분리하는 좋은 방식**이다.

---

### **2. 뷰(View)를 연관값으로 넣을 때의 주의점**

  뷰도 enum에 연관값으로 넣었을 때를 살펴보자
```swift
enum Screen {
    case home(HomeView)
    case detail(DetailView)
}
```

하지만 뷰는 상태(state)를 많이 담고 있고, 내부에서 클로저도 캡처하기 때문에
**예상치 못한 참조 관계** 가 만들어질 수 있따.
또한, SwiftUI 뷰는 Struct지만 내뷰는 참조 타입에 의존하고 있어 무겁고 복잡하다.

> ❗️enum이 뷰를 보관하고 있으면 메모리 릭이 발생하거나,
> 뷰가 재사용되지 않아 불필요한 렌더링이 생길 수 있다.

### **해결법**
- 뷰는 식별자로 관리하고, 필요 시 외부에서 생성
- enum에는 뷰 자체가 아닌 **뷰 타입 or 식별값** 만 보관

---

### **3. 모델(Model) 객체를 넣을 때 주의할 점**

모델은 보통 class 타입이고, 데이터 외에 기능도 담고 있음
```swift
enum State {
    case idle
    case loaded(DataModel)
}
```

여기서 DataModel이 **클래스(참조 타입)** 라면,
- 이 열거형이 .loaded(DataModel) 상태가 되는 순간
- State는 DataModel 인스턴스를 **참조**하고 있게 된다.

> 즉, State가 바뀌더라도 **이전 연관값인 DataModel이 즉시 해제되지 않을 수 있다.**

 → **enum** **은 값 타입이지만, 연관값은 참조 타입일 수 있다.**

---
#### **이전 상태가 메모리에서 바로 지워지지 않는 이유**?

**스택처럼 덮어쓰는 게 아닌, 참조 counting 기반이기 때문**
Swift에서 메모리는 **ARC (Automatic Reference Counting)** 로 관리.

```swift
var state: State = .loaded(DataModel())  
state = .idle
```

- 이 코드에서 .loaded(...) 상태였던 연관값 DataModel()은 **참조 카운트가 1**일 경우 → 상태가 바뀌는 순간 ARC에 의해 자동 해제된다.
- - 하지만 **어딘가에 참조가 남아 있다면?** → 참조 카운트가 남아서 메모리에 그대로 남는다.
예시)
```swift
let model = DataModel()
state = .loaded(model)
// model을 다른 곳에서도 참조하고 있으면 -> 상태를 바꿔도 model은 안 사라짐
```

---

#### **위험한 상황: 모델이 “가벼운 객체”가 아닐 때** ?

DataModel이 다음과 같은 경우, 더 심각.
- 내부에 **큰 이미지, 파일 캐시, 대용량 리스트**가 있을 경우
- **NotificationCenter, Combine, KVO** 등에 등록되어 있으면 메모리에서 해제 안 됨
- Timer를 캡처하고 있다면 → 계속 작동

> 즉, 상태만 바꿨다고 안심하면 안 된다.
> 실제로 메모리에 남아 있는지? 부작용이 일어나는지? **직접 관리해줘야 안전**하다.

해결법
1. 상태 변경 전에 클린업 : 상태가 .loaded(model) → .idle 되기 전, model 내부에서 Notification 해제, Timer 정지 등 해줌 
	1. ex) model.cleanup()
2.  **외부에서 주입 & 관리** : DataModel을 ViewModel이나 상위 객체가 관리하고, 상태 전환에는 참조만 넘김
	1. ex) state = .loaded(viewModel.model)
---

## GQ
- **Swift enum의 연관값으로 클로저나 클래스가 들어갈 때, 메모리 관리에서 어떤 주의가 필요할까?**


## Keywords
- [[열거형 (Enumeration)]]


#Onething 