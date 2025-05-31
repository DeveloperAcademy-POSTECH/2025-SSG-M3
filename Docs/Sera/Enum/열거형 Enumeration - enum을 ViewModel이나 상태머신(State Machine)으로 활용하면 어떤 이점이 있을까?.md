
> [!question] GQ1. enum을 ViewModel이나 상태머신(State Machine)으로 활용하면 어떤 이점이 있을까?

## Description

- 열거형을 사용하면 좋은 경우 : 제한된 선택지를 주고 싶을 때, 정해진 값 이외에는 입력값을 받지 않을 때, 예상된 입력값이 한정되어 있을 때
- 열거형이란 종류가 비슷한 값들을 그룹 개념으로 모아둔 것

## 주요 기능

- 열거형은 타입 자체가 제한된 값만 사용함 -> 변수나 함수는 유효성 검사를 해줘야함 , 열거형은 컴파일 할 때 미리 타입 검사함
- enum을 상태 관리에 사용하면 좋은 점 -  상태가 열거형에 정의된 것들만 존재하게 되어, 무분별한 상태 남용을 방지, 예상치 못한 상태가 발생할 가능성이 낮아짐. 새 상태 추가 시 컴파일러가 switch 문 등에서 에러로 알려주게 됨(실수 예방) 또한 `switch` 문을 활용해서 명확하게 상태가 어떤지 분리할 수 있다. 
- 또한 enum을 사용했을 때, viewmodel과 자연스럽게 연결 가능하다.
- boolean 값 사용하지 않고 그냥 상태로 표시가능해서 불필요한 boolean 제거
## 코드 예시

- 기본적인 Enum 형식 - case로 정의

```swift
enum Weather{
  case sunny
  case rainy
}
```
- ViewModel 사용 예시
```swift
enum LoginState {
    case idle
    case loading
    case success(User)
    case failed(String)
}

class LoginViewModel: ObservableObject {
    @Published var state: LoginState = .idle
    
    func login(username: String, password: String) {
        state = .loading
        
        
    }
}

```

원래라면 ViewModel에서 다양한 상태를 관리할 때, 여러 개의 프로퍼티를 따로따로 만들어야 함. 근데 열거형을 사용함으로  상태를 하나의 `state`로 명확하게 표현하여 SwiftUI는 @Published`된 state만 보면 되서 구조가 깔끔해진다. 


#Sera 