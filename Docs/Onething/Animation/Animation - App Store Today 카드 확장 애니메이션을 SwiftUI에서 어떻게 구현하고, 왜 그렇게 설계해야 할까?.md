
> [!question]
> **GQ**: App Store Today 카드 확장 애니메이션을 SwiftUI에서 어떻게 구현하고, 왜 그렇게 설계해야 할까?

## 0) 개요

목표는 **작은 카드 → 전체 화면**으로 부드럽게 확장하고, 닫으면 **원래 자리로 자연 복귀**하는 전환을 만드는 것.  

핵심은 두 가지다:  
1) **상태 머신**(collapsed ↔ expanded)로 전환을 모델링한다.  
2) 두 뷰를 **같은 실체**처럼 보이게 하는 `matchedGeometryEffect`를 사용한다.

![[ios_app_store_expanded_animation.mp4]]
![[ios_app_store_expanded_drag_animation.mp4]]

---

## 1) 문제 정의 — Today 카드 확장 전환에서 무엇을 풀어야 하나

- **상태 전이**: `collapsed ↔ expanded` 두 상태만 명확히 모델링한다.  
  - 탭하면 카드가 **전체 화면**으로 커지고, 닫으면 **원 위치**로 자연 복귀.
- **연속성**: 카드의 **위치, 크기, 코너 반경, 불투명도**가 “점프” 없이 **연속적으로** 변해야 한다.
- **레이어링**: 확장된 카드는 **전면(zIndex↑)** 으로, 배경은 **딤/블러**로 시선 집중.
- **입력 정책**: 전환 중엔 **스크롤·탭 충돌**이 나지 않도록 잠금; 드래그로 **진행률 기반 닫기**도 지원.
- **접근성/성능**: Reduce Motion 대응(애니메이션 강도 완화), 불필요한 리렌더 최소화.

> 요약: 이 전환은 좌표 수동 계산 문제가 아니라 **상태 전이 + 연속성 보장** 문제다.  
> 그래서 우리는 “어떻게 이동시킬까”를 직접 계산하지 않고, **보간에 맡긴다**는 전략을 쓴다.

---

## 2) 보간이란? — “중간값을 시스템이 매끄럽게 채우는 것”

### 한 줄 정의
**보간(Interpolation)**은 **시작값**과 **끝값** 사이의 **중간값들을 연속적으로 생성**하는 과정이다.  
UI 애니메이션에선 시간 `t`(0…1)에 따라 **위치·크기·투명도** 등의 **중간 상태**를 매 프레임 계산한다.

### 가장 기본 공식(선형 보간, Lerp)
`t=0`이면 시작값, `t=1`이면 끝값, 그 사이면 비율만큼 섞인다.
```swift
func lerp(_ a: Double, _ b: Double, _ t: Double) -> Double {
  (1 - t) * a + t * b
}

let x = lerp(0, 100, 0.3) // 30: 0→100 사이의 30%
```

### **이징(Easing)과의 관계**

- **보간**은 t에 따라 값을 채우는 행위
- **이징**은 그 t의 **속도 곡선**(선형, easeIn, spring 등)을 정의

```swift
withAnimation(.spring(response: 0.45, dampingFraction: 0.82)) {
  expanded = true // t의 흐름을 스프링 곡선으로 매핑
}
```

### **“보간에 맡긴다”의 실제 의미**

개발자가 좌표/트랜스폼을 프레임마다 **직접 계산하지 않고**,
**시작 상태**와 **끝 상태**만 선언하면 **시스템이 중간값을 자동 계산**한다는 뜻.

| **바꾸는 것** | **시작(collapsed)** | **끝(expanded)** | **중간값 계산 주체**                       |
| --------- | ----------------- | --------------- | ----------------------------------- |
| 프레임/위치    | 작은 카드 프레임         | 전체화면 프레임        | matchedGeometryEffect               |
| 코너 반경     | 16                | 24              | matchedGeometryEffect 혹은 Animatable |
| 딤 불투명도    | 0.0               | 0.25            | SwiftUI 애니메이션 엔진                    |
| 드래그 오프셋   | 0                 | 진행률×거리          | 제스처 → 값 보간                          |

## **3) 최소 구현 ** 

### 3.1 왜 `matchedGeometryEffect`인가

- **역할**: 같은 `id`를 공유하는 뷰를 **한 실체**로 간주해 **프레임/코너/클리핑**을 자동 보간한다.
- **효과**: 카드 셀 ↔ 디테일 뷰가 **같은 기하학적 공간**을 공유하듯 자연스럽게 확장/축소된다.

```swift

CardCell(card: card)
  .matchedGeometryEffect(id: card.id, in: cardNamespace)
  
CardDetail(card: selectedCard, onClose: closeDetail)
  .matchedGeometryEffect(id: selectedCard.id, in: cardNamespace)
```

> 팁: 매칭 대상은 **핵심 요소(배경 카드, 대표 텍스트/썸네일)** 로 최소화하면 깜빡임/보간 충돌을 줄일 수 있다.

### **3.2 상태 관리와 애니메이션 트리거**

- selectedCard: 어떤 카드가 선택됐는지 추적
- showDetail: 디테일 표시 여부(= 전환 트리거)

```swift
// ContentView.swift:18-24
@Namespace private var cardNamespace
@State private var selectedCard: TodayCard? = nil
@State private var showDetail = false
@Environment(\.accessibilityReduceMotion) private var reduceMotion

// ContentView.swift:54-65
private func selectCard(_ card: TodayCard) {
  selectedCard = card
  withAnimation(
    AnimationHelpers.adaptiveAnimation(
      for: AnimationHelpers.cardExpansion,
      reduceMotion: reduceMotion
    )
  ) {
    showDetail = true
  }
}
```


### **3.3 Spring 애니메이션 세밀 제어**
- **왜 스프링?** iOS 네이티브 감성(가속/감속)과 **물리적 자연스러움**을 그대로 얻는다.
```swift
// AnimationHelpers.swift:7-15
enum AnimationHelpers {
  static let cardExpansion: Animation = .spring(
    response: 0.6,        // 총 반응 시간(지속 길이 체감)
    dampingFraction: 0.8, // 탄성(낮을수록 통통 튐)
    blendDuration: 0.1    // 전환 블렌딩
  )

  static let dragResponse: Animation = .spring(response: 0.35, dampingFraction: 0.8)
}
```


### **3.4 ZIndex와 레이어 관리**

- 디테일을 **전면**으로 끌어올리고, 배경은 **딤**으로 시선 집중.
- .transition(.identity)로 **기본 전환을 제거**하고 **매칭 보간만** 사용.

```swift
// ContentView.swift:38-50
if let selected = selectedCard, showDetail {
  Color.black.opacity(AnimationConstants.backgroundDimOpacity)
    .ignoresSafeArea()
    .onTapGesture { closeDetail() }

  CardDetail(card: selected, onClose: closeDetail)
    .matchedGeometryEffect(id: selected.id, in: cardNamespace)
    .zIndex(10)               // 최상단
    .transition(.identity)    // 추가 전환 제거(매칭만)
}
```


### **3.5 드래그 제스처와 인터랙티브 디스미스**

- **거리 기준**: 일정 거리 이상 끌면 닫기
- **속도 기준**: 빠르게 스와이프하면 더 짧은 거리로도 닫기
- **피드백**: 드래그 중 스케일/투명도/코너 반경을 **진행률**로 보간

```swift
// CardDetail.swift:83-115
@State private var isDragging = false
@State private var dragOffset: CGSize = .zero

private var dragDistance: CGFloat { abs(dragOffset.height) }
private var dragVelocity: CGFloat = 0 // 제스처 업데이트 시 계산했다고 가정

private var dragGesture: some Gesture {
  DragGesture()
    .onChanged { value in
      isDragging = true
      dragOffset = value.translation
      // 필요 시 velocity 추정 로직 추가
    }
    .onEnded { value in
      let shouldDismiss =
        dragDistance > AnimationConstants.dragThreshold ||
        (dragDistance > AnimationConstants.minDragDistance &&
         dragVelocity > AnimationConstants.velocityThreshold)

      if shouldDismiss {
        onClose()
      } else {
        withAnimation(AnimationHelpers.dragResponse) {
          dragOffset = .zero
        }
      }
    }
}
```
> 진행률(p = dragOffset.height / 기준거리) 하나로 **offset/딤/cornerRadius** 를 동시에 보간하면 설득력 있는 인터랙션이 된다.


### **3.6 접근성(동작 줄이기) 대응**
- **Reduce Motion** 사용자는 **즉시 전환(near-zero duration)** 으로 부담을 낮춘다.
```swift
// AnimationHelpers.swift:26-33
extension AnimationHelpers {
  static func adaptiveAnimation(for animation: Animation, reduceMotion: Bool) -> Animation {
    reduceMotion ? .linear(duration: 0.01) : animation
  }
}
```


### **3.7 애니메이션 후 상태 정리(깜빡임 방지)**

- 매칭 보간이 끝나기 전에 selectedCard를 nil로 만들면 **점프/깜빡임**이 생긴다.
- **딜레이 후** 해제하여 **상태 일관성**과 **메모리**를 함께 챙긴다.

```swift
// ContentView.swift:72-78
private func closeDetail() {
  withAnimation(AnimationHelpers.cardExpansion) { showDetail = false }
  DispatchQueue.main.asyncAfter(deadline: .now() + AnimationConstants.selectionClearDelay) {
    selectedCard = nil
  }
}
```

```swift
// AnimationConstants.swift:5-16
enum AnimationConstants {
  static let backgroundDimOpacity: Double = 0.25
  static let selectionClearDelay: TimeInterval = 0.45 // cardExpansion과 맞춤

  // 인터랙티브 디스미스 기준값(앱에 맞게 조정)
  static let dragThreshold: CGFloat = 120
  static let minDragDistance: CGFloat = 40
  static let velocityThreshold: CGFloat = 1200 // pt/s
}
```

---

## **4) 인터랙티브 닫기 — 진행률 기반 보간**

드래그 제스처의 이동량을 **진행률(progress)** 로 환산해 **offset/딤/코너**를 함께 보간한다.
```swift
@State private var drag = CGSize.zero

func detail(_ c: Card) -> some View {
  RoundedRectangle(cornerRadius: cornerRadius)
    .fill(c.color.gradient)
    .offset(y: max(0, drag.height))
    .background(Color.black.opacity(dimOpacity))
    .gesture(
      DragGesture()
        .onChanged { v in drag = v.translation }
        .onEnded { v in
          let p = v.translation.height / 300
          if p > 0.35 { close() }
          else {
            withAnimation(.spring(response: 0.35, dampingFraction: 0.8)) {
              drag = .zero
            }
          }
        }
    )
    .matchedGeometryEffect(id: c.id, in: ns)
    .zIndex(10)
}

var dimOpacity: Double { max(0, 0.25 - drag.height/800) }
var cornerRadius: CGFloat { 24 - min(24, drag.height/20) } // 아래로 끌수록 각을 세움
```
하나의 진행률을 여러 속성에 **동일하게 적용**해야 설득력 있는 체감이 나온다.

## **5) 레이어링 전략 — 무엇을 매칭할까**

- **매칭 최소화**: 배경 카드 + 대표 텍스트/썸네일만 매칭
- 내부 스크롤 컨텐츠는 별도 레이어로 두어 **불필요 보간** 회피
- 배경 효과는 **딤(반투명)** 우선, 블러는 비용이 크니 선택적으로

## **6) 접근성/입력 정책**
- 전환 중엔 ScrollView.disabled(true)로 **입력 충돌** 제거
- 탭/드래그 외 **대체 경로**(스와이프/ESC) 제공


## Keyword
- Animation


#Onething 