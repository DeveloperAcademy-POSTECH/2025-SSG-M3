
>[!question]
>GQ1. Combine의 sink와 SwiftUI의 onReceive는 어떤 차이가 있는가?

## Description
<u> Combine의 sink 란?</u>
- Combine 의 프레임워크에서 Publisher 의 값을 직접 구독하고 처리하는 메서드. 
- 클로저를 통해 받은 값을 처리하거나 완료 이벤트를 핸들링, `AnyCancellable` 반환
- 주로 ViewModel 이나 Service 로직에서 사용됨. 
```Swift
$text
  .sink { value in
      print("값 수신: \(value)")
  }
  .store(in: &cancellables) //이 객체 보관하지 않으면 구독이 즉시 사라지고, viewMdoel 클래스가 해제되지 않는 한 ㄱ
```

<u> SwiftUI의 onReceive 란?</u>
- SwiftUI View에서 사용되는 **Modifier 형식의 구독 메서드**로,View가 특정 Publisher의 값을 수신했을 때 **UI와 연동된 처리**를 실행함.
- SwiftUI 뷰의 생명주기와 함께 **자동**으로 구독이 관리됨. 
```Swift
Text("안녕하세요")
    .onReceive(timerPublisher) { time in
        print("현재 시간: \(time)")
    } // 이 뷰가 사라지면 구독도 자동으로 취소되고 해제할 필요도 없음.
```

<u> 둘의 공통점은? </u>
- 둘 다 Combine의 Publisher를 구독하고 값을 수신할 수 있음
- 둘 다 비동기적으로 이벤트 스트림을 처리함
- `@Published`,`NotificationCenter`, `Timer.publish`
 등 다양한 Publisher에 사용 가능함
<u>차이점?</u>

| **항목**       | sink                                         | onReceive                                   |
|----------------|----------------------------------------------|----------------------------------------------|
| **정의 위치**   | 일반 코드 또는 ViewModel 등 어디서나 사용 가능     | SwiftUI View 내부에서만 사용 가능             |
| **리턴 값**     | AnyCancellable 반환 → 메모리 관리 필요             | 반환값 없음 → View 생명주기에 따라 자동 해제   |
| **주 용도**     | 로직 처리, 데이터 저장, side effect 등            | UI 업데이트와 연동                            |
| **메모리 관리** | store(in:)을 통해 명시적으로 관리                | SwiftUI가 자동 관리 (View와 함께 사라짐)       |
| **Test 용도**   | 유닛 테스트에서 유용                            | UI 테스트나 View 내 처리에 적합                |
## 결론 ! 
- `sink`는 Combine의 범용 구독 방식이고, `onReceive`는 SwiftUI View에서 값 수신 후 **UI 반영에 특화된** Modifier임.



## Keywords
+ [[Combine]]

## References
-  [Apple Developer Documentation – sink](https://developer.apple.com/documentation/combine/publisher/sink\(receivevalue:\))
-  [Apple Developer Documentation – onReceive](https://developer.apple.com/documentation/swiftui/view/onreceive\(_:perform:\))

## 작성자
- #Zhen 