>[!question]
>GQ1. 클로저 내부에서 self를 명시적으로 사용하는 이유는 무엇인가

## Description
- ## 클로저(Closure)
   클로저는 Named Closure가 있고, Unnamed Closure가 있는데, Unnamed Closure가 우리가 흔히 알고 있는 클로저이고, Named Closure는 이름이 있는 함수인데 우리가 클로저라고 부르지 않고 그냥 함수라고 불렀을 뿐임. 
   정리: 클로저는 Named Clousre & Unnamed Closure 둘다 포함하지만, 보통 Unnamed Closure를 말한다!
- 클로저는 함수처럼 입력 매개변수와 리턴값을 가질 수 있고, 변수에 저장하거나, 함수에 인자로 넘기거나, 리턴값으로 반환할 수도 있다. 다른 함수 안에서 정의되거나, 비동기 작업에서 자주 사용됨. 클로저도 익명이긴 하지만 함수이기에, 함수의 특징들을 다 가지고 있음

## 주요 기능

### 클로저 내부에서 self를 명시적으로 사용하는 이유 
## 1. 첫 번 째 - 캡처 의도를 명시하기 위해
+ 클로저는 자신이 선언된 외부 환경의 변수와 객체를 캡처(capture)해서 내부에서 사용할 수 있음 
+ 캡처한다 = 클로저가 외부 변수나 인스턴스를 붙잡고, 내부에서 계속 사용할 수 있게 저장해둔다

클로저는 기본적으로 자신이 사용한 외부 객체를 "강하게(capture by strong reference)" 참조
- 만약 클로저가 `self`를 강하게 캡처하고, `self`가 그 클로저를 프로퍼티로 갖고 있다면→ 서로가 서로를 참조하게 됨 → 메모리에서 안 사라짐 → retain cycle(서로가 서로를 붙잡고 있어서 메모리에서 해제되지 않는 문제)발생 → 따라서 self를 명시적으로 사용해야함.

Swift는 클로저 안에서 `self`를 참조할 때 컴파일러가 "너 진짜 의도한 거 맞아?" 하고 확인하려고 한다. 
그래서 보통 클로저 내부에서 `self.`를 명시적으로 쓰게 함. → 실수로 인한 retain cycle을 줄임 

```swift
class ViewModel {
    var callback: (() -> Void)?

    func setup() {
        callback = {
            self.doSomething()  //  self를 클로저가 강하게 잡음
        }
    }

    func doSomething() {}
}
```

- 만약 `self`가 `callback`을 프로퍼티로 갖고 있다면,  클로저도 `self`를 강하게 붙잡고 → 서로 강한 참조 → 메모리에서 안 사라짐. 그래서 컴파일러가  “진짜 self를 여기서 붙잡을 의도야? 그냥 실수로 넣은 거 아냐?” 라고 묻는거임.. self 빼면 오류나용 

## 2.  두 번 째 - 값에 접근하기 위해

 클로저 내부에서 외부 객체의 프로퍼티를 사용하려면, 그 값을 어디서 가져올지 클로저가 알아야 함. 
근데 클로저 자체 안에는 외부 객체의 이름을 가진 객체가 없으니까,  "그 값은 내가 캡처한 self 안에 있어"* → 그래서 `self.userName`이라고 명시해야 함. 

```swift
class TestViewController {
    var userName = "이준"

    func setup() {
        let closure = {
            print(self.userName) // ✅ 가능! self를 캡처해서 userName 접근
        }
    }
}
```

-  만약 `self.`를 안 쓰면,   → Swift는 “이거 지역 변수니? 외부 거니?” 헷갈릴 수 있고,  → 메모리 참조 문제가 생겨도 개발자가 의도한 건지 실수한 건지 알 수가 없다. 


## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- #Sera 