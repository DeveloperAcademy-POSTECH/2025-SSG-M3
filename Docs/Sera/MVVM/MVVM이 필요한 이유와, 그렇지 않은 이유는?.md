https://gist.github.com/unnnyong/439555659aa04bbbf78b2fcae9de7661

# mvvm (model–view–viewmodel)

- 개념: 
	- model: 데이터 구조(dto, 엔티티, 저장/네트워크 레이어).
    - view: ui 그리는 쪽(바인딩만 담당).
    - viewmodel: 뷰와 모델 사이의 변환기/로직 담당(데이터 포맷팅, 입력 검증, 비동기 호출 등).


UIKit 환경에서 개발을 할때 ViewModel을 두고 데이터 바인딩을 해주는 역할

SwiftUI.View라는 것은 이미 "View + ViewModel"의 기능을 갖고 있어서, 직접 Model의 값을 Reactive하게 View에 반영하는 것이 가능함, SwiftUI에서 View는 자체적으로 Data Binding이 가능한 PropertyWrapper를 지원해줌

ViewModel의 역할중 가장 중요한건 데이터 바인딩임, 즉 모델과 뷰 사이에 양방향 소통을 해줌으로 바인딩을 시켜주는것. 그런데 SwiftUI에서는 View에서 다 해줄수 있기에 이럴 필요가 없다는 소리임


UI와 로직의 분리- MVVM을 사용하지 않았을 때, "ViewModel이 없어지면 UI와 로직의 분리는 어떻게하지?" 답은 Model 또는 Flux적인 Store로 분리. 중소규모의 어플이라면 MV(Model과 View)라면 충분하다. 단순히 View와 로직을 분리하기 위해서 ViewModel을 만드는 것은, 이름과 해야하는 일이 맞지 않기 때문에 더 이상 필요하지 않다고 생각함

글쓴이는 ObservableObject의 이름을 @@ViewModel로 하는 것보다, @@Model, @@Store 등과 같이 하는 역할에 따라 명명하는 것을 추천함 → **ViewModel이라고 표현하는것이 어색하다는 주장** (비지니스 로직을 분리 시키는게 필요하다면 사실 viewmodel이 아니더라도 다른걸로 분리시키면 되지않나.)

![[Pasted image 20250816215058.png]]애플이 권장하는 단방향 데이터 플로우

### 그럼 MVVM 대신 무엇을?
- **SwiftUI 네이티브 방식 활용:** 간단한 뷰는 `@State`를 사용하고, 여러 뷰에서 상태를 공유해야 할 때는 `@Binding`, `@EnvironmentObject` 등 SwiftUI가 제공하는 도구를 최대한 활용하는 것. 모델 객체 자체를 `ObservableObject`로 만들어 뷰가 직접 구독하는 **MV(Model-View)** 패턴
- **단방향 아키텍처(Unidirectional Architecture) 채택:** 상태 관리가 정말로 복잡해진다면, 여러 ViewModel로 상태를 분산시키는 대신 **TCA**와 같은 아키텍처를 도입해 **하나의 거대한 상태(Single Source of Truth)**와 **예측 가능한 데이터 흐름(Action -> Reducer -> State)**을 갖추는 것이 더 낫다는 주장


### 그럼에도 MVVM이 여전히 1등 아키텍처로 쓰이는 이유
 
 1. **유지보수와 확장성**
- 프로젝트가 커지면 View와 비즈니스 로직을 분리하지 않으면 **코드가 꼬이기 쉬움.**
- MVVM은 **ViewModel을 통해 UI와 로직 분리** → 테스트 용이, 재사용성 증가.
- 예를 들어, **네트워크 요청, 데이터 변환, 상태 관리**를 ViewModel에서 처리하고 View는 단순 렌더링만 하도록 유지 가능.

2. **테스트 용이**
- SwiftUI에서 View는 UI 테스트가 어려움.
- ViewModel에 로직을 몰아주면 **단위 테스트(Unit Test)와 로직 검증**이 쉬워짐.

3. **복잡한 상태 관리에 강점**
- 앱이 커지고 여러 화면이 서로 상태를 공유할 때, **ViewModel로 상태를 중앙 집중화**하면 관리가 편리함.
- Combine이나 SwiftData와 함께 쓰면 reactive + 상태 관리가 더 명확해짐.

4. **아키텍처 관습**
- MVVM은 iOS 커뮤니티에서 가장 많이 사용되는 패턴이라 **문서, 예제, 라이브러리, 경험치 공유**가 풍부함.
- 팀 프로젝트에서 합의된 패턴으로 사용하기 용이.