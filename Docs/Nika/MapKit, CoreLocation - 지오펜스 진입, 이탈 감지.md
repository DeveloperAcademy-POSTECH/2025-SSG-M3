## 지오펜스(Geofence)란?

위치 기반의 가상 울타리. 특정 좌표를 중심으로 반경을 지정해, 사용자가 그 영역에 **들어오거나 나갈 때(진입/이탈)** 이벤트를 발생시키는 기능이다.

### 위치 권한 요청

info.plist에서 아래 키들을 입력

- `NSLocationWhenInUseUsageDescription` - 위치를 사용할 때만 권한 요청
- `NSLocationAlwaysAndWhenInUsageDescription` - 항상 위치 접근 허용 요청
- `UIBackgroundModes` → Item 0: `location` - 백그라운드에서도 위치 추적 가능

### 핵심 개념

**`CLCircularRegion`** 지오펜스를 구현하는 객체. 원형 영역(좌표 + 반경)으로 구성. `notifyOnEntry`, `notifyOnExit` 설정 가능.

```swift
let region = CLCircularRegion(center: coordinate, radius: 50, identifier: "지역이름")
region.notifyOnEntry = true
region.notifyOnExit = true
```

**`CLLocationManager`** 위치 권한 요청 및 지오펜스 감지 처리 담당. 반드시 `delegate` 설정 필요. 진입/이탈 시 아래 메서드 호출

```swift
func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion)
func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion)
```

### **기본 코드 예시**

```swift
import CoreLocation

class LocationManager: NSObject, CLLocationManagerDelegate {
	let locationManager = CLLocationManager()

	override init() {
		super.init()
		locationManager.delegate = self
		locationManager.requestAlwaysAuthorization()
		locationManager.startUpdatingLocation()

		let center = CLLocationCoordinate2D(latitude: 36.0175, longitude: 129.3222)
		let region = CLCircularRegion(center: center, radius: 50, identifier: "포항공대 생활관 18동")
		region.notifyOnEntry = true
		region.notifyOnExit = true

		locationManager.startMonitoring(for: region)
	}

	func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
		print("진입: \\\\\\\\(region.identifier)")
	}

	func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
		print("이탈: \\\\\\\\(region.identifier)")
	}
}
```

**지도로 시각화**

```swift
Map(coordinateRegion: $region,
	showsUserLocation: true,
	annotationItems: [GeoPin(location: centerCoordinate)]) { item in
	MapMarker(coordinate: item.location)
}
```