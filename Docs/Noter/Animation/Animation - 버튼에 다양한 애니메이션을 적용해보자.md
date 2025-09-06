
> [!question]  
> GQ. 버튼에 다양한 애니메이션을 적용해볼까?

#### 1. 개요

버튼 하나에 여러 SwiftUI Animation을 적용해보며 어떤 애니메이션이 있고 어떻게 적용할 수 있는지 등을 학습한다. + 어떤 애니메이션이 "더 자연스럽다" 또는 "불필요하게 튄다"는 인상을 주는지 비교해본다!

#### 2. SwiftUI 버튼 탭 인터랙션에 쓸 애니메이션 종류

1. 크기(Scale)
		`.scaleEffect()`
		`.scaleEffect(x:y:)`
	버튼이 커졌다 작아지면서 눌리는 애니메이션

2. 회전(Rotation)
		`.rotationEffect()`
	버튼을 회전시켜 주목을 끄는 애니메이션

3. 불투명도(Opacity)
		`.opacity()`
	버튼이 사라지거나 강조되는 애니메이션

4. 위치 이동(Offset/Position)
		`.offset(x:y:)`
		`.postion(x:y:)`
	버튼이 살짝 이동하거나 튕기는 애니메이션

5. 모양 변화(CornerRadius, ClipShape 등)
		`.cornerRadius()`
		`.clipShape()`
	눌렀을 때 모양이 바뀌는 애니메이션

6. Matched Geometry Effect
		`matchedGeometryEffect(id:in:)`
	버튼 → 다른 뷰로 전환할 때 자연스럽게 연결되는 애니메이션

7. 3D 회전 (Rotation3DEffect)
		`.rotation3DEffect()`
	버튼이 입체적으로 회전하며 눌리는 애니메이션

8. 애니메이션 곡선 (Animation Curves)
		`.easeIn`, `.easeOut`, `.easeInOut`, `.spring()` 등
	모션의 리듬을 다르게 표현하기


#### 3. 버튼들을 만들어보자

1. 또잉또잉 버튼

• 탭하면 버튼이 또잉또잉 하면서 작아지고 또잉또잉하면서 커짐
• .spring()과 scaleEffect, rotationEffect 등을 섞음

```swift
Button(action: {
		withAnimation(.spring(response: 0.4, dampingFraction: 0.2, blendDuration: 0)) {
			isPressed.toggle()
		}
	}) {
		Text(isPressed  ? "🥚" : "🐔")
			.font(.system(size: 50))
			.frame(width: 100, height: 100)
			.background(Color.gray)
			.clipShape(Circle())
			.scaleEffect(isPressed ? 0.7 : 1.0)
			.rotationEffect(.degrees(isPressed ? Double.random(in: -50...50) : Double.random(in: -50...50)))
	}
```


2. 도망가는 버튼

• 누르면 랜덤 위치로 움직임

```swift
Button(action: {
		offsetX = CGFloat.random(in: -150...150)
		offsetY = CGFloat.random(in: -350...350)
	}) {
		Text("PUSH")
			.padding()
			.background(Color.green)
			.foregroundColor(.white)
			.clipShape(Capsule())
	}
	.offset(x: offsetX, y: offsetY)
	.animation(.spring(response: 0.4, dampingFraction: 0.5), value: offsetX)
```


3. 물결 생성 버튼

• 누르면 물에 돌을 떨어뜨린 듯 버튼을 중심으로 원형이 퍼짐
• 시간차 애니메이션

```swift
ZStack {
	ForEach(ripples) { ripple in
		Circle()
			.fill(Color.blue.opacity(0.3))
			.frame(width: 100, height: 100)
			.scaleEffect(ripple.scale)
			.opacity(ripple.opacity)
			.onAppear {
				withAnimation(.easeIn(duration: 1.6)) {
					if let index = ripples.firstIndex(where: { $0.id == ripple.id }) {
						ripples[index].scale = 5.0
						ripples[index].opacity = 0.0
					}
				}
				DispatchQueue.main.asyncAfter(deadline: .now() + 1.6) {
					ripples.removeAll { $0.id == ripple.id }
				}
			}
		}

	Button(action: {
		for i in 0..<4 {
			DispatchQueue.main.asyncAfter(deadline: .now() + Double(i) * 0.2) {
				ripples.append(Ripple())
			}
		}
	}) {
		Text("Ripple")
			.font(.title)
			.foregroundColor(.white)
			.padding(40)
			.background(.blue)
			.clipShape(Circle())
	}
}
```


4. 풍선 버튼

- 누르면 빠르게 부풀었다가 터짐
- `.timingCurve(c0x, c0y, c1x, c1y, duration: TimeInterval)`사용

```swift
if isVisible {
	Text("푸웅선")
		.font(.system(size: 40))
		.padding(40)
		.foregroundColor(.white)
		.background(.pink)
		.clipShape(Circle())
		.scaleEffect(isPopping ? 4.0 : 1.0)
		.opacity(isPopping ? 0.3 : 1.0)
		.onTapGesture {
			withAnimation(.timingCurve(0.9, -0.1, 0.3, 1.4, duration: 1.7)) {
				isPopping = true
			}
			DispatchQueue.main.asyncAfter(deadline: .now() + 1.7) {
				isVisible = false
			}
		}
}
```


#### 4. 정리

SwiftUI의 애니메이션은 **상태 변화에 따른 UI 속성의 변화**를 자연스럽게 연결해주는 선언형 방식이다. .scaleEffect, .opacity, .rotationEffect 등 다양한 속성을 withAnimation이나 .animation()과 함께 조합해 다양한 사용자 인터랙션을 표현할 수 있다.

- 안 써본 애니메이션 곡선(Timing Functions)들을 모두 써보았다.
	SwiftUI가 기본으로 제공하는 애니메이션 커브가 한정적이다. 키프레임 애니메이션이나 곡선 변화를 프레임 단위로 세밀하게 조작하는 것이 사실상 불가능하다. timingCurve를 쓰면 조금 더 유연하긴 하지만 CSS keyframe이나 플러터의 CurvedAnimation처럼 복잡한 모션 시나리오 구현에는 한계가 있다!

- `DispatchQueue.main.asyncAfter`와 반복문을 활용해 시간차를 두고 애니메이션을 발생시키는 것을 해보았다. 번거롭다. 리액트의 setTimeout처럼 자연스럽게 연쇄되는 구조가 아니라서 직접적인 시간 제어 로직이 필요하다!


[[Animation]]

#Noter 