Process accelerometer, gyroscope, pedometer, and environment-related events.
**iOS, watchOS, macOS, visionOS 기기에서 모션과 환경 센서 데이터를 수집할 수 있게 해주는 프레임워크**
하드웨어로부터 원시(raw) 또는 가공(processed)된 데이터를 앱으로 전달해준다. 예를 들어 게임에서 디바이스를 흔들어 조작하거나, 건강 앱에서 사용자의 걸음 수를 추적할 때 활용된다.

ex)
- 사용자가 기기를 어떻게 움직이고 있는지 → 가속도계, 자이로스코프
- 현재 고도나 방향 → 기압계, 자력계
- 건강 관련 활동 데이터 → 만보기, 활동 감지, 낙상 감지 등


#### 필수 주의 사항(권한)
---

기기 센서에 접근하려면 Info.plist에 설명 키를 반드시 넣어야 한다. 없으면 앱이 강제 종료.

|**기능**|**필요 키**|
|---|---|
|모션/피트니스 데이터|NSMotionUsageDescription|
|낙상 감지|NSFallDetectionUsageDescription|

#### 핵심 구성 요소
---

| **클래스**                  | **설명**                              |
| ------------------------ | ----------------------------------- |
| CMMotionManager          | 모션 데이터 수집을 시작/관리. 센서 데이터 접근의 핵심 매니저 |
| CMDeviceMotion           | **중력/편향 보정**된 모션 정보                 |
| CMAttitude               | 기기의 **자세 (orientation)**            |
| CMAttitudeReferenceFrame | 자세 기준 좌표계 정의                        |

```swift
import CoreMotion

let motionManager = CMMotionManager()

if motionManager.isAccelerometerAvailable {
    motionManager.accelerometerUpdateInterval = 0.1  // 0.1초마다 측정
    
    motionManager.startAccelerometerUpdates(to: .main) { (data, error) in
        if let acceleration = data?.acceleration {
            print("X: \(acceleration.x), Y: \(acceleration.y), Z: \(acceleration.z)")
        }
    }
}
```

- CMMotionManager가 진짜 데이터를 가져오는 객체
- CMAccelerometerData는 그 데이터의 타입
- data?.acceleration은 CMAcceleration 타입으로, x, y, z 축의 값 포함

#### 주요 센서별 데이터(클래스) 요약
---
모두 iPhone, iPad, Apple Watch 등에 직접 탑재된 하드웨어 센서!

가속도계(Accelerometer)
- CMAccelerometerData: x, y, z 축의 가속도 원시값
- CMSensorRecorder: 일정 시간 동안 가속도 데이터 기록
- CMRecordedAccelerometerData: 기록된 샘플

자이로스코프(Gyroscope)
- CMGyroData: 회전 속도(raw) 값

자력계(Magnetometer)
- CMMagnetometerData: 지구 자기장 방향

고도/기압(Altitude/Pressure)
- CMAltimeter: 고도 변화 감지
- CMAltitudeData, CMAbsoluteAltitudeData: 상대/절대 고도 데이터
- CMAmbientPressureData: 환경 기압


#### 사용 예시
---
윷을 던지듯 아이폰을 아래에서 위로 흔들면 가속도 센서와 자이로스코프 센서가 이를 감지하고 윷을 가상 공간에 던지는 로직

```swift
import CoreMotion

let motionManager = CMMotionManager() // 가속도, 자이로 등을 관리하는 센서 매니저

func startMotionDetection(){
	// 가속도계 센서로 흔드는 힘 받아오기
	if motionManager.isAccelerometerAvailable { // 가속도계 센서가 장치에 있는지 확인
		motionManager.accelerometerUpdateInterval = 0.1 // 0.1초마다 데이터 갱신 (10Hz)
		motionManager.startAccelerometerUpdates(to: OperationQueue.main) { data, error in
		// 주기적으로 센서 데이터를 받아옴
			guard let data = data else { return } // 센서 데이터 없으면 무시

			// 아래에서 위로 흔드는 제스처 감지 (y축 가속도 값 기준)
			if data.acceleration.y > 1.2 {
			// acceleration.y 값이 특정 임계치를 넘으면 윷 던지기로 간주
				print("윷 던지기 동작 감지!")
				throwYut() // 윷 던지기 로직 실행
			}
		}
	}
	
	// 자이로스코프로 회전 정도 받아오기
	if motionManager.isGyroAvailable {
    motionManager.gyroUpdateInterval = 0.1
    motionManager.startGyroUpdates(to: OperationQueue.main) { gyroData, error in
        guard let gyroData = gyroData else { return }

        let rotationRate = gyroData.rotationRate
        print("회전 속도 x: \(rotationRate.x), y: \(rotationRate.y), z: \(rotationRate.z)")
		// rotationRate.x, .y, .z는 각각 축을 기준으로 한 초당 회전량(rad/s)

        // 회전이 충분히 빠르면 윷을 빠르게 던진 걸로 판단 가능
        if abs(rotationRate.y) > 2.0 {
            print("강한 회전 감지!")
        }
    }
}
```

하나하나 뜯어보자!

**`motionManager.accelerometerUpdateInterval = 0.1`**  

→ 0.1초 간격(10Hz)으로 가속도 데이터를 갱신하도록 설정.
**값이 작을수록 더 자주 업데이트되고**, 더 많은 전력을 사용하게 된다.

**`motionManager.startAccelerometerUpdates(to: OperationQueue.main) { data, error in ... } `**

→ 센서 데이터를 수집하기 시작한다.
- to:는 데이터를 어느 쓰레드에서 받을지를 지정 (main이면 UI 관련 작업 가능)
- { data, error in ... }는 *콜백 클로저*\*로, 데이터가 들어올 때마다 호출됨

**`if data.acceleration.y > 1.2`**  

가속도 값은 x, y, z 축으로 나뉘는데,
- y는 **세로 방향(상하)**
- 1.2는 실험적으로 정한 **임계값**: 위로 흔들었을 때의 가속도가 이 값을 넘는 경우를 감지

</br>
\* 콜백 클로저가 멀까?

콜백: 어떤 일이 끝난 후 실행할 함수
클로저: 변수처럼 다룰 수 있는 익명 함수

둘을 합치면?
"작업이 끝나면 실행해줘! 라고 넘겨주는 함수"


#### Keyword
---
- [[Apple Frameworks]]

#### 작성자
#Noter 