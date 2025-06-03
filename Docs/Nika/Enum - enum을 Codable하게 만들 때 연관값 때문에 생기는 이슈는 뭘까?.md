enum의 연관값은 내부적으로 어떤 케이스인지와 해당 값들을 함께 다뤄야 하는 복잡한 구조이다.

[[Codable]]이 자동으로 작동하려면

1. 클래스 혹은 구조체의 모든 속성이 Codable일 때
2. Swift가 `encode(to:)`와 `init(from:)`를 자동 생성 가능할 때 위의 조건을 만족해야 하는데, 연관값을 가지고 있는 enum처럼 복잡한 구조는 자동 합성이 불가능하다.

즉,

```
enum Result: Codable {
	case success(data: String)
	case failure(error: String)
}
```

와 같은 연관값이 있는 enum에 Codable을 그냥 붙이면 컴파일 에러가 난다. error message: `Enum with associated values cannot automatically synthesize Codable conformance`

### 해결 방법

1. init(from:)과 encode(to:)를 직접 구현
    
    ```
    enum Result: Codable {
        case success(String)
        case failure(Int)
    
        enum CodingKeys: String, CodingKey {
            case type
            case value
        }
    
        enum CaseType: String, Codable {
            case success, failure
        }
    
        init(from decoder: Decoder) throws {
            let container = try decoder.container(keyedBy: CodingKeys.self)
            let type = try container.decode(CaseType.self, forKey: .type)
    
            switch type {
            case .success:
                let value = try container.decode(String.self, forKey: .value)
                self = .success(value)
            case .failure:
                let value = try container.decode(Int.self, forKey: .value)
                self = .failure(value)
            }
        }
    
        func encode(to encoder: Encoder) throws {
            var container = encoder.container(keyedBy: CodingKeys.self)
    
            switch self {
            case .success(let str):
                try container.encode(CaseType.success, forKey: .type)
                try container.encode(str, forKey: .value)
            case .failure(let code):
                try container.encode(CaseType.failure, forKey: .type)
                try container.encode(code, forKey: .value)
            }
        }
    }
    ```
    
2. 중간 래퍼 타입을 도입
    
    - enum 자체는 Codable이 아니고, 래퍼 구조체만 Codable하게 설계
    
    ```
    enum Result {
        case success(String)
        case failure(Int)
    }
    
    struct CodableResult: Codable {
        let type: String
        let stringValue: String?
        let intValue: Int?
    
        var toResult: Result {
            switch type {
            case "success":
                return .success(stringValue ?? "")
            case "failure":
                return .failure(intValue ?? -1)
            default:
                fatalError("Invalid type")
            }
        }
    
        init(from result: Result) {
            switch result {
            case .success(let str):
                self.type = "success"
                self.stringValue = str
                self.intValue = nil
            case .failure(let code):
                self.type = "failure"
                self.stringValue = nil
                self.intValue = code
            }
        }
    }
    ```
    
    사용 예시
    
    ```
    let result: Result = .success("Done")
    let codable = CodableResult(from: result)
    let data = try JSONEncoder().encode(codable)
    ```
    
3. 외부 라이브러리 활용(ex. `SwiftEnumCodable`
    
    ```
    import SwiftEnumCodable
    
    @CodableEnum
    enum Result {
        case success(String)
        case failure(Int)
    }
    ```

#Nika 