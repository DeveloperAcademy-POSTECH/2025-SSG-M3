>[!question]
>GQ. AVFoundation의 활용 사례를 알아보자

## Description
- AVFoundation은 Apple의 운영체계에서 시간 기반 시청각 미디어를 처리하기 위한 강력한 프레임워크이다. 이를 통해 손쉽게 오디오 및 비디오를 재생, 녹화, 편집하고 분석할 수 있다.

## 주요 기능
+ 미디어 재생 (Playback): `AVPlayer`와 `AVPlayerViewController`를 사용하여 비디오와 오디오 파일을 간편하게 재생할 수 있다.
+ 미디어 캡쳐 (Capture): `AVCaptureSession`을 사용하여 카메라와 마이크로부터 실시간 오디오 및 비디오 데이터를 캡쳐(녹화)할 수 있다.
+ 미디어 편집 (Editing): `AVComposition`을 사용하여 기존 미디어 파일들을 자르거나 합치는 등 비선형 편집 작업을 수행할 수 있다.
+ 미디어 포맷 처리: 다양한 포맷의 미디어 파일을 읽고 쓸 수 있으며, 다른 포맷으로 변환하는 기능을 제공한다.

## 코드 예시
#### 비디오 재생하기
```Swift
import UIKit
import AVFoundation

class ViewController: UIViewController {

    var player: AVPlayer!
    var playerLayer: AVPlayerLayer!

    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 1. 비디오 파일 URL 준비
        guard let url = URL(string: "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4") else { return }

        // 2. AVPlayer를 만들고 비디오 URL을 전달
        player = AVPlayer(url: url)

        // 3. AVPlayerLayer (영상이 보일 스크린)를 만들고 player를 연결
        playerLayer = AVPlayerLayer(player: player)
        playerLayer.videoGravity = .resizeAspectFill // 화면 채우기 옵션
        
        // 4. 뷰의 레이어에 비디오 스크린(playerLayer)을 추가
        view.layer.addSublayer(playerLayer)

        // 5. 비디오 재생
        player.play()
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        // 뷰의 크기가 정해지면, 비디오 스크린의 크기도 맞춤
        playerLayer.frame = view.bounds
    }
}
```

## Keywords
+ [[AVFoundation]]

## References
- [Apple Developer Documentation: AVFoundation](https://developer.apple.com/documentation/avfoundation)
- [Apple Developer, AVFoundation Overview](https://developer.apple.com/av-foundation/)
## 작성자
- #Air 