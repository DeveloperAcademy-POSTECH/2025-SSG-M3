
| 구성 요소                          | 설명                           |
| ------------------------------ | ---------------------------- |
| `ARView`                       | AR 콘텐츠를 보여주는 뷰 (RealityKit)  |
| `ARSession`                    | 카메라 및 센서 추적 정보 관리            |
| `ARWorldTrackingConfiguration` | 세션 설정 (지면 인식, 텍스처링 등)        |
| `Entity`                       | 3D 모델, 라이트, 앵커 등 AR 씬의 구성 요소 |
| `AnchorEntity`                 | 3D 객체를 공간에 고정시키는 위치 기준점      |

목표
- 후면 카메라로 현실 보기
- 탭한 지점에 3D 모델(.usd) 놓기

```swift
import SwiftUI
import RealityKit
import ARKit

struct ARViewContainer: UIViewRepresentable {
    func makeUIView(context: Context) -> ARView {
        let arView = ARView(frame: .zero)

        // 1. AR 세션 구성
        let config = ARWorldTrackingConfiguration()
        config.planeDetection = [.horizontal]
        config.environmentTexturing = .automatic
        arView.session.run(config)

        // 2. 탭 제스처 추가
        let tapGesture = UITapGestureRecognizer(target: context.coordinator, action: #selector(context.coordinator.handleTap(_:)))
        arView.addGestureRecognizer(tapGesture)

        context.coordinator.arView = arView
        return arView
    }

    func updateUIView(_ uiView: ARView, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator()
    }

    class Coordinator: NSObject {
        weak var arView: ARView?

        @objc func handleTap(_ sender: UITapGestureRecognizer) {
            guard let arView = arView else { return }
            let location = sender.location(in: arView)

            // 3. Raycast (지면 찾기)
            guard let result = arView.raycast(from: location, allowing: .estimatedPlane, alignment: .horizontal).first else {
                print("Raycast 실패")
                return
            }

            // 4. 앵커에 모델 배치
            let anchor = AnchorEntity(world: result.worldTransform)

            do {
                let model = try Entity.loadModel(named: "toy_robot.usdz")
                if let modelEntity = model as? ModelEntity {
                    modelEntity.generateCollisionShapes(recursive: true)
                    anchor.addChild(modelEntity)
                }
                arView.scene.anchors.append(anchor)
            } catch {
                print("모델 로딩 실패")
            }
        }
    }
}

```

SwiftUI에 적용
```swift
struct ContentView: View {
    var body: some View {
        ARViewContainer()
            .edgesIgnoringSafeArea(.all)
    }
}
```


출처
- [Apple ReailityKit 공식문서](https://developer.apple.com/documentation/realitykit)

#Nika