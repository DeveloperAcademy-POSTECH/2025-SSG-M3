>[!question]
>GQ. rawValue와 associated value는 각각 어떤 상황에서 쓰는 게 적절할까?

## Description
- Swift의 Enum은 단순한 값 열거 이상의 기능을 제공한다. 특히 raw value와 associated value는 각각 다른 용도와 상황에 따라 사용된다.
- Raw Value
	- 각 케이스에 고정된 기본 값을 할당하는 방식이다. 각 케이스 별 타입은 동일해야 한다. 주로 외부 값과의 매핑이 필요하거나, 식별용 상수 값이 필요한 경우에 사용된다.
- Associated Value
	- 각 케이스가 다양한 타입과 개별적인 값을 가질 수 있다. 케이스 별로 필요한 데이터를 런타임에 저장할 수 있어서 유연성이 높다. 주로 복합적인 데이터를 표현하거나, 상태/이벤트 표현 등에 적합하다.

## 주요 기능
| 항목        | Raw Value            | Associated Value |
| --------- | -------------------- | ---------------- |
| 값 타입      | 고정                   | 케이스마다 다르게 지정 가능  |
| 값 사용 시기   | 컴파일 타임               | 런타임              |
| 주 용도      | 외부 데이터 매핑, 설정 값      | 복합 데이터 표현, 상태 관리 |
| 중복 가능 여부  | rawValue는 유일해야 함     | 중복 허용            |
| 케이스 접근 방식 | .init(rawValue:) 초기화 | 패턴 매칭으로 값 추출     |
#### Raw Value 사용 예
- 서버에서 받은 코드값 (status = "OK") 매핑
- 메뉴, 언어 코드, 방향 값 등 고정된 상수 표현
#### Associated Value 사용 예
- 네트워크 상태 표현 (loading, success(data), failure(error))
- 폼 입력 값, 사용자 액션에 따라 다른 데이터 전달

## 코드 예시
#### Raw Value
```Swift
enum HttpStatus: Int {
	case ok = 200
	case notFound = 404
	case internalServerError = 500
}

// 초기화 예시
let status = HttpStatus(rawValue: 404)
```
#### Associated Value
```Swift
enum NetworkResult {
	case success(data: String)
	case failure(error: Error)
	case loading(progress: Float)
}

func handle(result: NetworkResult) {
	switch result {
	case .success(let data):
		print("Data received: \(data)")
	case .failure(let error):
		print("Error: \(error.localizedDescription)")
	case .loading(let progress):
		print("Loading: \(progress * 100)%")
	}
}
```

```Swift
enum Barcode {
	case upc(Int, Int, Int, Int)
	case qrCode(String)
}

var productBarcode = Barcode.upc(8, 85909, 51226, 3)
productBarcode = .qrCode("ABCDEFGHIJKLMNOP")

```

```Swift
enum Planet: Int {
	case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}

let earthsOrder = Planet.earth.rawValue // earthsOrder is 3

let possiblePlanet = Planet(rawValue: 7)
```

## Keywords
+ [[열거형 (Enumeration)]]

## References
- [Apple Documentation: Enumerations](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations/)
## 작성자
- #Air 