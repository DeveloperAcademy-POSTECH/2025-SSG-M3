### 1번

```swift
Button {
	withAnimation(.interpolatingSpring(stiffness: 300, damping: 18)) {
		isOn.toggle()
	}
} label: {
	**if** isOn {
		ZStack {
			Circle()
				.fill(Color.purple)
			Image(systemName: "camera")
				.font(.system(size: 28, weight: .semibold))
				.foregroundColor(.white)
		}
		.frame(width: 72, height: 72)
		.transition(AnyTransition.asymmetric(
			insertion: .scale(scale: 0.01, anchor: .center).combined(with: .opacity),
			removal: .opacity
		)
		)
	} **else** {
		Image(systemName: "magnifyingglass")
			.resizable()
			.scaledToFit()
			.frame(width: 72, height: 72)
			.foregroundColor(.purple)
			.transition(AnyTransition.asymmetric(
				insertion: .scale(scale: 0.01, anchor: .center).combined(with: .opacity),
				removal: .opacity
			)
			)
	}
}
```
- `withAnimation(...) { isOn.toggle() }`  
    → **상태 변경을 애니메이션 트랜잭션 안**에서 실행 (그래야 `transition`이 실제로 “등장/퇴장”을 애니메이션함)
- `withAnimation(_:_:)`
	- 클로저 안의 **상태 변화**(여기선 `isOn.toggle()`)를 지정한 애니메이션으로 실행.
- `.interpolatingSpring(stiffness:damping:)`
    - **물리 기반 스프링**(질량-스프링-댐퍼)으로, 자연스러운 “튕김/오버슛”을 만듦.
    - 파라미터 의미:
        - **stiffness**(스프링 강성, 큼→빠르고 탱탱/오버슛↑)
        - **damping**(감쇠, 큼→덜 튐·빨리 멈춤 / 작음→더 튀고 오래 흔들)
    
- 각 분기 끝의 `.transition(...)`  
    → **어떻게** 등장/퇴장할지 정의 (예: 작게 → 크게, 투명 → 불투명)


### 2번
```swift
Button {
	withAnimation(.spring(response: 0.2, dampingFraction: 0.48)) {
		isOn.toggle()
	}

} label: {
	Group {
		if isOn {
			ZStack {
				Circle()
					.fill(Color.purple)
				Image(systemName: "camera")
					.font(.system(size: 28, weight: .semibold))
					.foregroundColor(.white)
			}
			.frame(width: 72, height: 72)
			.transition(
				.scale(scale: 0.01, anchor: .center)
				.combined(with: .opacity)
			)
		}
		else {
			Image(systemName: "magnifyingglass")
				.resizable()
				.scaledToFit()
				.frame(width: 72, height: 72)
				.foregroundColor(.purple)
				.transition(
					.scale(scale: 0.01, anchor: .center)
					.combined(with: .opacity)
				)
		}
		}
}
```
1번에서 withAnimation만 조금 수정한 것
- **무슨 스프링?**  
    heuristic 기반 스프링(간편형). 물리 파라미터(`stiffness/damping`) 대신 **반응 속도**와 **되튐 정도**로 조절해요.
- **파라미터 의미**
    - `response: 0.2` → 반응 속도. 작을수록 더 빠르고 날쌤(팝 느낌↑).
    - `dampingFraction: 0.48` → 감쇠 비율. 작을수록 더 튐/오버슛↑, 클수록 차분.
    - `blendDuration`(생략 시 0) → 다른 애니메이션과 섞이는 시간. 보통 0이면 가장 즉각적.
- **튜닝 팁**    
    - 더 팝: `response` ↓(예: 0.18), `dampingFraction` ↓(예: 0.42)
    - 더 차분: `response` ↑, `dampingFraction` ↑(0.6~0.7)

이전에 쓰던 `.interpolatingSpring(stiffness:damping:)`는 **물리 모델**을 직접 만지는 타입,  
`.spring(response:dampingFraction:)`는 **느낌 위주로 빠르게 튜닝**하는 타입이라고 보면 됨.

### 3번

```swift
@State var isOn: Bool = false
    @State private var play = 0
    
    struct Values {
        var scale: CGFloat = 1.0
        var y: CGFloat = 0
        var hueDeg: Double = 0
    }
    
    var body: some View {
        Button {
            play += 1
            isOn.toggle()
        } label: {
            ZStack {
                Image(systemName: "heart")
                    .resizable().scaledToFit()
                    .foregroundStyle(.black)
                    .frame(width: 72, height: 72)
                Image(systemName: "heart.fill")
                    .resizable().scaledToFit()
                    .frame(width: 72, height: 72)
                    .foregroundStyle(.pink)
                    .keyframeAnimator(initialValue: Values(), trigger: play) { content, v in
                        content
                            .scaleEffect(v.scale)
                            .offset(y: v.y)
                            .hueRotation(.degrees(v.hueDeg))
                    } keyframes: { _ in
                        // 스케일: 1.0 → 1.5 → 1.0
                        KeyframeTrack(\\.scale) {
                            CubicKeyframe(1.5, duration: 0.16)
                            CubicKeyframe(1.0, duration: 0.20)
                        }
                        // 위치(Y): 0 → -72 → 0  (위로 72pt 갔다가 제자리)
                        KeyframeTrack(\\.y) {
                            CubicKeyframe(-72, duration: 0.16)
                            CubicKeyframe(0, duration: 0.20)
                        }
                        // 색상: 0° → +40° → 0°
                        KeyframeTrack(\\.hueDeg) {
                            CubicKeyframe(40, duration: 0.16)
                            CubicKeyframe(0, duration: 0.20)
                        }
                    }
            }
        }
    }
```
- struct Values { scale, y, hueDeg }
	- 키프레임에서 다룰 **애니메이션 값 묶음**(모델).
	    - `scale`: 하트 크기(스케일)
	    - `y`: 세로 이동(오프셋). **음수면 위로 이동**
	    - `hueDeg`: 색상 회전 각도(도 단위). **0° 기준 → +각도면 색상이 주황 방향으로 이동**
	- `initialValue: Values()`로 **시작 상태**를 명시(스케일 1, 위치 0, 색상 변화 0°).