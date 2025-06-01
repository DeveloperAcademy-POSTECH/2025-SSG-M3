>[!question]
>GQ1. enum은 어떤 상황에서 struct나 class보다 더 적절할까?

## Description
- enumeration - Model custom types that define a list of possible values.
- enum / class / struct 를 비교해보고, 어떤 때 enum을 사용하는게 효율적인지 생각해보자 

## 주요 기능
- 세 가지의 차이점은? 
	- 초기화함수의 유무(Initializer) : class o / struct o / enum x (설정이 안되어도 default 값을 가짐)
	- 상속(Inheritance), Type casting : class o / struct x / enum x
	- Type
		- Struct, Enum : 값 타입 (Value Type)
		- Class : 참조 타입 (Reference Type)


- 그럼 언제 Enum 이 효율적일까? 
	- 유한한 case 일 때 (월, 요일 등)
	- 패턴 매칭 (switch 와 함께 깔끔하게 사용 가능)
	- 연관값과 매칭값 (`case date(year: Int, month: Int, day: Int)`)
	- 의미상 상태의 표현 (Error log 남길 때)
	- 프로토콜 채택이 쉬움

- 추가) 그럼 언제 struct 나 class 를 사용해야 할까? 
- - **`struct`**
    - 비교적 단순한 데이터 덩어리(모델)를 표현.
    - **불변성(Immutable)** 을 지향하거나, 복사 시 복제되는 값 타입이 필요한 경우.
    - 예: API에서 받은 JSON을 파싱해서 저장할 모델, 좌표나 크기를 나타내는 `Point(x:y:)` 등.
        
- **`class`**
    - **상속(inheritance)** 이 필요하거나, **객체 식별성(identity)**(참조가 동일해야 하는 경우)이 필요할 때.
    - 예: 데이터베이스 연결 객체, 뷰 컨트롤러, 개별 인스턴스 간의 참조 관계를 유지해야 하는 코어 오브젝트.
        
- **`enum`**
    - 위에서 설명한 대로 **유한 개의 케이스**만 가질 때.
    - 예: 앱 내에서 특정 모드(편집모드 / 읽기모드), 서버 응답 상태, 에러 타입, 옵션 토글(on/off) 등.
## 코드 예시

Enum이 효율적인 실제 예시들 
+ 옵션이 한정적인 경우
```
enum Weekday {
    case monday, tuesday, wednesday, thursday, friday, saturday, sunday
}
```

- app 상태 관리
```
enum LoadingState {
    case idle
    case loading
    case success(data: [User])
    case failure(error: Error)
}
```

- Error 타입 정의
```
enum MyServiceError: Error {
    case invalidURL
    case networkFailed(code: Int)
    case parsingError(message: String)
}
```

- 뷰 혹은 컴포넌트 전환 케이스 
```
enum AppTab {
    case home
    case search
    case profile
    case settings
}
```

- 상호 배타적인 옵션 (오직 한개만 선택해야 할 때)
```
enum AppTab {
    case home
    case search
    case profile
    case settings
}
```

이 외에도 api 응답 상태 등을 표현할 때 enum이 효율적이다. 

## Keywords
+ [[열거형 (Enumeration)]]

## References
- Apple 공식 문서
	- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures/ class - struct 비교
	- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations/ enum
- blog
	- https://jayb-log.tistory.com/216

## 작성자
- #Zhen 