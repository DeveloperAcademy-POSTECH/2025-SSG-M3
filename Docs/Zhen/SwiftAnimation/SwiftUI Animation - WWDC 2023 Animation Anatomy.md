>[!question]
>GQ1. 애플에서는 앱에 애니메이션을 추가하는 것을 아주 간단하게 만드는것이 SwiftUI 개발을 시작할 때 핵심 동기중 하나였다고 한다... WWDC 2023 에서 무엇이 업데이트 되었는지 전반적으로 이해해 보자. 
>GQ2. 애니메이션 작동 원리를 해부해보자... 

## Description
- Animation 이란: 상태변화가 시간에 따라 보이게 만드는 메커니즘
- SwiftUI 는 `@State` 같은 값이 바뀌면 뷰를 다시 그리게 되는데, 이때 
	- 무엇을 애니메이션 할지 (ex. opacity, scale, offset 등... 이런걸 "Animatable 속성"이라고 함)
	- 시간에 따라 어떻게 바뀔지 (ex. easeInOut, spring, bouncy 같은 Animation)
	- 이번 업데이트에 적용할 공통 규칙 (Transaction: 한 번의 업데이트 맥락 )
	이 세 축으로 애니메이션이 동작하게 됨. 

## 어떤 관점으로 이해하면 될까? 
+ Update Anatomy (업데이트.. 해부학.. )
	+ 상태값 변경 → 트랜젝션 열림 → body 재계산 → 렌더링 
	- 애니메이션은 이 이번 트랜잭션에 붙여서 같이 보내는 개념. 
	- ![[Pasted image 20250903205637.png]]
	- ![[Pasted image 20250903205714.png]]
	- ![[Pasted image 20250903205813.png]]
	- ![[Pasted image 20250903205856.png]]
	- ![[Pasted image 20250903205923.png]]
		scaleEffect 는 애니메이션이 가능한 속성이라, 발동? 하기전에 트랜젝션에 애니메이션이 설정되어 있는지 확인함. 설정되어있으면 암튼엄청복잡하게 로컬 사본을 만든담에 그걸 뭐 비교하고 어쩌고 해서 다음 프레임을 만듦.

- Animatable (무엇을 바꾸는가?)
	- 크기(CGFloat/CGSize), 위치(CGPoint), 사각형(CGRect) 처럼 벡터처럼 빼고 스케일링할 수 잇는 값이 애니메이션 대상이 됨
	- 그래서 scaleEffect, opacity, offset 같은 모디파이어가 부드럽게 변함
- Animation (시간을 어떻게 흐르게 할 것인가! )
	- Timing: `linear`, `easeInOut` 등 속도 곡선으로 제어함
	- Spring: `smooth`, `snappy`, `bouncy` 같은 자연스러운 스프링 계열을 제공 
	- Higher-order: `delay`, `speed`, `repeat` 으로 애니메이션 조합하고 수정
	- CustomAnimation(ㅋㅋ): 직접 시간→값 함수를 정의 가능함... 실무에서는 내장형으로 대부분 해결하고 진짜진짜 필요할때 하는거 권장... (어려움! ^^)
### Transaction (이번(2023) 업데이트의 context)
- withAnimation 이 트랜잭션 딕셔너리에 애니메이션을 설정함 
	→그 프레임의 하위 트리에 전파됨
- `transaction {...}/withTransaction` 으로 범위를 좁혀 예기치 않은 하위 애니메이션 전파를 막을 수 있음 

- 실질적인 팁
	- 단일 컴포넌트에는 `.animation(_, value:)`가 편함
	- 임의의 자식 컨텐츠가 있는 상위 뷰에서는 전역 `.animation` 남발하면 곤란 (당연함)
	- 자연스러운 UI엔 스프링 계열이 기본적으로 잘 맞음 
	- 스유에서는 내장된 애니메이션을 쓰는게 되~게 효율적이다 ~ 왜냐면 되~게 잘 설계해서 내부적으로 만들어놧기 때문이다 ~ 라는 말을 하는 듯. 
## 코드 예시
+ 기본 
```Swift
import SwiftUI

struct BasicAnimationView: View {
    @State private var expanded = false

    var body: some View {
        VStack(spacing: 20) {
            RoundedRectangle(cornerRadius: 20)
                .fill(.blue)
                .frame(width: expanded ? 220 : 120, height: 120)
                .opacity(expanded ? 1.0 : 0.7)

            Button("Toggle") {
                // 이번 업데이트 트랜잭션에 애니메이션을 실어 보냄
                withAnimation(.easeInOut(duration: 0.4)) {
                    expanded.toggle()
                }
            }
        }
        .padding()
    }
}

```
- 요점: **상태 변경을 withAnimation으로 감쌈 → 그 변화가 시간에 따라 보간됨**.

- 값 트레킹 방식 `.animation(.bouncy, value: angle)`

```Swift
struct ValueAnimationView: View {
    @State private var angle = 0.0

    var body: some View {
        Image(systemName: "arrow.triangle.2.circlepath.circle.fill")
            .font(.system(size: 80))
            .rotationEffect(.degrees(angle))
            .animation(.bouncy, value: angle)   // angle이 변할 때만 bouncy 적용
            .onTapGesture { angle += 45 }
    }
}
```
- 요점: **특정 값이 변할 때만** 해당 뷰 계층에 애니메이션을 적용함. 불필요한 전파 줄어듦.

- Transition
```Swift
struct TransitionDemoView: View {
    @State private var show = false

    var body: some View {
        VStack(spacing: 24) {
            ZStack {
                if show {
                    RoundedRectangle(cornerRadius: 24)
                        .fill(.green)
                        .frame(width: 160, height: 160)
                        .transition(.scale.combined(with: .opacity))
                }
            }
            Button(show ? "Hide" : "Show") {
                withAnimation(.snappy(duration: 0.35)) {
                    show.toggle()
                }
            }
        }
        .padding()
    }
}
```
- 요점: **조건부 뷰 + .transition** 조합으로 등장/퇴장을 자연스럽게 함.

- 범위형 애니메이션 (하위 전파 최소화)
```Swift
struct ScopedAnimationView: View {
    @State private var on = false

    var body: some View {
        VStack(spacing: 24) {
            // 이 블록 내부에만 애니메이션 규칙을 적용함
            .animation(.smooth, value: on) // iOS 17+에서 범위형 API도 고려
            Group {
                Text("Shadow & Scale")
                    .font(.title3)
                    .shadow(radius: on ? 12 : 2)
                    .scaleEffect(on ? 1.2 : 1.0)
            }

            Button(on ? "Off" : "On") { on.toggle() }
        }
        .padding()
    }
}
```

- 트랜잭션 더 세밀한 제어 
```Swift
struct TransactionDemoView: View {
    @State private var active = false

    var body: some View {
        VStack(spacing: 20) {
            RoundedRectangle(cornerRadius: 16)
                .fill(.mint)
                .frame(width: active ? 220 : 120, height: 120)
                .transaction { txn in
                    // 이 업데이트에서만 애니메이션 바꾸기
                    // (전역 규칙과 별개로, 이번 변화에 국소 적용)
                    if active {
                        txn.animation = .bouncy
                    } else {
                        txn.animation = .easeInOut(duration: 0.25)
                    }
                }

            Button("Toggle") {
                withTransaction { txn in
                    // 필요하면 커스텀 키를 실어 보낼 수도 있음(고급 패턴)
                    active.toggle()
                }
            }
        }
        .padding()
    }
}
```


## Keywords
+ [[Animation]]

## References
- https://green1229.tistory.com/375
- [https://developer.apple.com/videos/play/wwdc2023/10156/](https://developer.apple.com/videos/play/wwdc2023/10156/)

## 작성자
- #Zhen 