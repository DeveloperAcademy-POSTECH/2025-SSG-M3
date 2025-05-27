>[!question]
>GQ1. 함수나 그냥 변수 선언 대신 열거형을 쓰는 이유가 뭘까?
>GQ2. 열거형을 switch 문과 사용했을 때 왜 좋다고 하는 걸까? 

## Description
- 열거형이란 종류가 비슷한 값들을 그룹 개념으로 모아둔 것
- 열거형을 사용하면 좋은 경우 : 제한된 선택지를 주고 싶을 때, 정해진 값 이외에는 입력값을 받지 않을 때, 예상된 입력값이 한정되어 있을 때


## 주요 기능
- 열거형은 타입 자체가 제한된 값만 사용함 -> 변수나 함수는 유효성 검사를 해줘야함 , 열거형은 컴파일 할 때 미리 타입 검사함
- Switch문을 사용하면 enum의 다양한 기능 사용 가능, Enum에서 모든 값 출력하거나, 특정 값 출력가능
## 코드 예시
 +  Enum 형식 - case로 정의

```swift
enum Weather{
  case sunny
  case rainy
}
```

- Switch 문과 함께 사용하는 예시

```swift
let stateA = "State A"
let stateB = "State B"
let currentState = stateA
if currentState == stateA {
print("현재 상태는 A입니다. ")
} else if currentState == stateB {
print("현재 상태는 B입니다. ")
} else {
print ("상태를 알 수 없습니다.")
}
```

```swift
enum State{
 case A
 case B
}
let currentState: State = . A
switch currentState {
case .A:
print ("현재 상태는 A입니다. ")
case .B:
print("현재 상태는 B입니다. ")
```

누가봐도 밑이 깔끔하긴함..
## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)