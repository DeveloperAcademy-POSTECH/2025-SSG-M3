
>[!question]
>GQ1. Combine에서의 메모리 관리는 어떻게 이루어지며, AnyCancellable은 왜 필요한가?

## Combine
**Combine**은 Apple이 2019년에 발표한 프레임워크로, **시간의 흐름에 따라 발생하는 비동기적인 이벤트들을 처리하고 조합하기 위한 선언형 Swift API**임. 쉽게 말해, 비동기 코드를 더 깔끔하고 읽기 쉽게 만들어주는 도구
Combine에서의 메모리 관리는 **구독(Subscription)의 생명 주기를 명시적으로 관리**


## 메모리 관리
Combine에서 Publisher는 구독자가 나타나기 전까지 아무 일도 하지 않음. `sink`나 `assign` 같은 구독자를 연결하는 순간, Publisher는 값을 생성하고 보내기 시작함. 이 연결, 즉 **구독**은 하나의 객체이며, 이 객체가 메모리에 살아있는 동안에만 Publisher와 Subscriber 사이의 데이터 흐름이 유지됩니다.

만약 이 구독 객체가 더 이상 필요 없어지기 전에 메모리에서 해제된다면, 데이터 스트림은 **즉시 종료**. 반대로, 구독 객체가 필요 이상으로 계속 살아있으면 **메모리 누수(Memory Leak)** 가 발생.

예를 들어, 어떤 뷰(View)가 자신의 데이터를 업데이트하기 위해 뷰모델(ViewModel)의 Publisher를 구독했다. 사용자가 다른 화면으로 이동하여 해당 뷰가 화면에서 사라졌다면, 이 구독은 더 이상 필요가 없습니다. 하지만 구독이 취소되지 않고 계속 메모리에 남아있다면, 보이지 않는 뷰를 업데이트하기 위해 불필요한 작업이 계속 수행되고 메모리를 낭비하게 되는 것



## AnyCancellable
위의 메모리 문제를 해결하기 위해 AnyCancellable가 필요함
`AnyCancellable`은 **구독을 취소할 수 있는 기능을 제공하는 타입 소거(Type-Erased) 클래스**`.store(in:)` 메서드를 통해 구독을 `AnyCancellable` 인스턴스들의 `Set`에 저장하면, 구독의 생명주기를 특정 객체의 생명주기와 일치시킬 수 있음

**핵심 원리:** `AnyCancellable`은 **`deinit` (소멸자)이 호출될 때 자동으로 `cancel()` 메서드를 실행**

1. **구독과 객체의 생명주기 연결**: 뷰컨트롤러나 뷰모델 같은 클래스 내부에 `private var cancellables = Set<AnyCancellable>()` 프로퍼티를 선언
2. **구독 저장**: 생성된 구독의 마지막에 `.store(in: &cancellables)`를 호출하여 해당 `Set`에 저장
3. **자동 취소**: 이제 구독의 생명은 `cancellables` Set을 소유한 객체(뷰컨트롤러 등)에 묶이게 됩니다. 해당 **객체가 메모리에서 해제될 때(`deinit` 호출)**, 프로퍼티인 `cancellables` Set도 함께 해제. 이때 Set 안에 저장된 모든 `AnyCancellable` 인스턴스들의 `deinit`이 호출되면서, 각각의 구독이 **자동으로 취소(cancel)


따라서 `AnyCancellable`과 `.store(in:)`을 사용하는 것은 "이 구독은 `cancellables` Set을 가진 객체가 살아있는 동안에만 유효하게 해줘. 그리고 그 객체가 사라지면 이 구독도 깨끗하게 정리해줘." 라고 Combine 프레임워크에 명시적으로 알려주는 행위이다. 

정리하자면 , `AnyCancellable`은 **구독의 종료 시점을 명확하게 정의**하여 원치 않는 메모리 사용과 불필요한 작업을 막고, **자동으로 리소스를 정리**해주는 매우 중요한 안전장치임

```swift
import Foundation
import Combine

// (NewsFeedProvider는 위와 동일)

// 2. 메모리 문제를 해결한 뉴스 화면
class ProperNewsViewController {
    let provider = NewsFeedProvider()
    // 구독을 담을 '가방' 준비. 이 가방이 사라지면 내용물(구독)도 함께 사라짐.
    private var cancellables = Set<AnyCancellable>()

    init() {
        print("😊 ProperNewsViewController 화면이 나타남")
        subscribeToNews()
    }

    func subscribeToNews() {
        provider.newsPublisher
            .sink { _ in
                print("😊 새로운 뉴스를 받았습니다.")
            }
            // 구독을 '가방'에 저장!
            .store(in: &cancellables)
    }

    deinit {
        // 이 화면이 메모리에서 해제될 때, 'cancellables' 가방도 함께 버려집니다.
        // 그러면서 가방 안의 모든 구독이 자동으로 .cancel() 됩니다.
        print("✅ ProperNewsViewController 화면이 메모리에서 해제됨")
    }
}

```
## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- #Sera 