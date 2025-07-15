>[!question]
>[!question]
>GQ1. Room이 무엇인가?
>GQ2. 무엇을 할 수 있는지?

## Description

Swift에서 RoomPlan, iPhone이나 iPad의 LiDAR 센서를 이용해 실내 공간을 스캔하고 3D 공간 모델을 생성할 수 있게 해주는 기술입니다. Swift 개발자가 증강현실(AR) 기술을 기반으로 실내 구조를 자동으로 인식하고, 방의 구조를 실시간으로 시각화하거나 저장하는 앱을 만들 수 있도록 지원합니다.

## 주요 기능

실내 구조 인식: 벽, 문, 창문, 가구 등의 위치와 크기를 자동 인식
3D Room 모델 생성: .usdz 파일 등으로 방의 3D 모델 생성 가능
Swift로 통합: Swift 기반 앱에서 쉽게 통합 가능
ARKit 기반: ARKit + RealityKit와 함께 작동하며 AR 장면에서 실시간 피드백 제공

## 코드 예시

```swift
import RoomPlan

let scanViewController = RoomCaptureViewController()
scanViewController.delegate = self
present(scanViewController, animated: true)
```
- RoomCaptureViewController를 통해 스캔 UI 제공
- RoomCaptureSessionDelegate를 통해 스캔 결과 처리 가능

스캔이 끝나면 다음과 같은 정보들을 받을 수 있습니다
- 방의 경계, 벽의 위치
- 창문, 문 위치
- 방에 있는 가구들의 위치 및 종류
- 3D 모델 (RoomCaptureResults) → .usdz 파일로 export 가능

## 스캔 프로젝트 작동 흐름

1. **스캔시작
```swift
import RoomPlan
// ...
let sessionConfig = RoomCaptureSession.Configuration()
roomCaptureView.captureSession.run(configuration: sessionConfig)
```
2. **Delegate로 스캔 중 실시간 데이터 수신
```swift
func captureSession(_ session: RoomCaptureSession, didUpdate room: CapturedRoom) {
    // 갱신된 CapturedRoom 객체
}
```
3. **스캔 종료 후 모델 export
```swift
func captureView(_ view: RoomCaptureView, didPresent processedResult: CapturedRoom, error: Error?) {
    try processedResult.export(to: url)
}
```

## 작성자
- #Hama 