
>[!question]
>GQ1. 1. **Implicitly Unwrapped Optional은 왜 존재할까? 언제 쓰면 안 될까?**

## Description

- Optional은 nil을 사용할 수 있는 타입과 없는 타입을 구분하기 위함이며, nil을 사용할 수 있는 Type을 Optional Type이라 부른다 'nil' 이라는 값을 가질 수 있으면 Optional Type이고, 이 Optional Type을 선언할 땐 타입 옆에 ?(물음표)를 붙인다. 예시) let name : String? nil을 저장할 수 있는 것은 오직 Optional Type 이다.
- nill은 값이 없다는 것. 언제 사용되냐? → 너 방금 오류났어 근데  이 정도로 앱을 중단시키긴 좀 그러니까 프로그램의 안정성을 위해 오류는 안 낼게 대신 nil을 돌려줄테니 너 오류인 건 알고 있어라 이럴 때 사용한다. 때문에 Optional을 **Swift 언어의 안전성**에 해당한다고 함
- wrapping? **래핑**은 어떤 값을 **다른 타입으로 감싸는 것.** `Int` → `Optional<Int>`로 감싸는 걸 **래핑(wrapping),** nill을 안전하게 다루기 위해서. Optional은 무조건 래핑된거임
- **Implicitly Unwrapped Optional**은 '암묵적으로 언래핑한 옵셔널' 이라는 용어 그대로 옵셔널이지만 언래핑하지 않고 사용할 수 있는 옵셔널임
- Optional을 만들 때 question mark(물음표)를 사용했으니까 Implicitly Unwrapped Optional을 만들 때는 exclamation point(느낌표) 를 사용해주면 됨. 예시) String!

## 주요 기능

- 클래스나 구조체의 속성으로 사용될 때, 초기화 과정에서는 값이 설정되지 않지만, 초기화 이후에는 반드시 값이 존재하는 경우에 사용함.
- IUO는 어떤면에서는 그냥 optional이 아닌 변수와 비슷함. 왜냐면 값의 존재여부를 확인하지 않아도 되니까.. → 따라서 반드시 값을 가지고 있다고 판단될때만 사용해야함.

## 코드 예시

```swift
class SampleClass {
    var mustHaveValue: Int?  // 일반 옵셔널
}

let sample = SampleClass()
sample.mustHaveValue = 10
print(sample.mustHaveValue)  // 🔴 경고! 타입이 Int?라 그냥 출력하면 옵셔널로 출력됨. 
                             //Optional(10)

```

```swift
class SampleClass {
    var mustHaveValue: Int!
}

let sample = SampleClass()
sample.mustHaveValue = 10
print(sample.mustHaveValue)  // 10 출력, 별도의 unwrapping 없이 사용 가능
```

그냥 옵셔널과 IWO의 코드 차이

```swift
class MyViewController: UIViewController {
    @IBOutlet weak var label: UILabel!  // 아직 연결 안 됐지만, 나중엔 반드시 연결됨
}

```

- 이 때 label 은 초기화 직후엔 nill 이지만,뷰가 로드되면서 Interface Builder에서 **반드시 연결이 될 거라고 보장됨.** 그래서 UIlabel! 로 선언함 → **나중엔 그냥 label.text = … 식으로 바로 사용 가능**
- 만약 `UILabel?`로 선언하면, 매번 `if let`, `!`, `??` 등으로 값이 들어있는지 확인을 해가며 써야 함 → 코드가 복잡해짐

## Keywords

- 옵셔널
- 옵셔널 체이닝 → 후에 공부해볼 것

## References

- [https://zeromin-code.tistory.com/157](https://zeromin-code.tistory.com/157)
- [https://uv0lpurmoon.tistory.com/16](https://uv0lpurmoon.tistory.com/16)