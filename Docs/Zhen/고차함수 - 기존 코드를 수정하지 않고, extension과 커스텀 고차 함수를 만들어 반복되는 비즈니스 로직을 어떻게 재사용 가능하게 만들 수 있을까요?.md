
>[!question]
>GQ1. 고차 함수 사용이 성능에 미치는 영향에 대해 설명하고, 성능 최적화가 필요한 경우 어떤 점을 고려해야 할까요?
>GQ2. 기존 코드를 수정하지 않고, extension과 커스텀 고차 함수를 만들어 반복되는 비즈니스 로직을 어떻게 재사용 가능하게 만들 수 있을까요?

## 주요 기능
### GQ1 
- 고차 함수가 성능에 영향을 줄 수 있는 이유:
	- ✅ 중간 배열 생성: `map().filter().reduce()`를 연달아 쓰면 각 단계에서 새로운 배열이 생성됨.
	- ✅ 클로저 캡처 비용: 고차 함수에 전달하는 클로저가 외부 값을 많이 캡처할수록 메모리와 성능에 영향을 줌.
	- ✅ `lazy` 연산 여부: `lazy` 키워드를 사용하면 중간 배열을 생성하지 않고 지연 평가됨.
	- ✅ 반복문 대체 여부: 경우에 따라 단순한 for 반복문이 더 빠를 수 있음

- 성능을 높이기 위해 고려할 점:
	- `.lazy`를 붙여서 중간 컬렉션 생성을 피함
	- 클로저 안에서 너무 복잡한 연산을 피함
    - `map/filter` 체이닝이 반복문보다 느린지 확인해보고 교체할 수 있음

## 코드 예시 

```Swift 
import SwiftUI

struct ContentView: View {
    @State private var normalTime: TimeInterval = 0
    @State private var lazyTime: TimeInterval = 0

    var body: some View {
        VStack(spacing: 20) {
            Button("일반 고차 함수 테스트") {
                let start = CFAbsoluteTimeGetCurrent()
                let result = (1...10_000_000)
                    .map { $0 * 2 }
                    .filter { $0 % 3 == 0 }
                    .reduce(0, +)
                let end = CFAbsoluteTimeGetCurrent()
                normalTime = end - start
                print("일반 결과: \(result)")
            }

            Button("lazy 고차 함수 테스트") {
                let start = CFAbsoluteTimeGetCurrent()
                let result = (1...10_000_000).lazy
                    .map { $0 * 2 }
                    .filter { $0 % 3 == 0 }
                    .reduce(0, +)
                let end = CFAbsoluteTimeGetCurrent()
                lazyTime = end - start
                print("lazy 결과: \(result)")
            }

            Text("일반 소요 시간: \(normalTime, specifier: "%.4f")초")
            Text("lazy 소요 시간: \(lazyTime, specifier: "%.4f")초")
        }
        .padding()
    }
}
```
- Product > Profile > TimeProfiler 성능 차이 확인 가능 
### GQ2
- 결론 : 반복되는 로직(예: 문자열 유효성 검사)을 뷰에서 반복하지 않고, extension이나 고차 함수로 뽑아내서 재사용 가능하게 만드는 것이 핵심. 

- 예시 : TextField로 이메일을 입력받을 때, 아래 조건들을 반복해서 검사한다고 가정함:
	1. 빈 문자열인지?    
	2. @ 기호가 포함돼 있는지?
	3. .com으로 끝나는지?
- 그러면 각 뷰에서 이런 식으로 검색하게 된다고 생각하면 ? 
```Swift
if !email.isEmpty && email.contains("@") && email.hasSuffix(".com") {
    // 유효한 이메일
}
```
- 같은 로직 중복 -> 유지보수 어려움
- 검사 조건이 바뀔 경우 -> 모든 곳을 수정해야 함
- Unit test 도 어렵고 비효율적임 
	- cf) 유닛 테스트란 ? 하나의 작은 기능(=유닛)이 **제대로 동작하는지 자동으로 검사해보는 테스트 코드**를 말함.
- 그래서 이걸 고차함수 + extenstion 을 사용하면 다음과 같이 수정 가능

```Swift 
extension String {
    func isValid(using rules: [(String) -> Bool]) -> Bool {
        for rule in rules {
            if !rule(self) {
                return false
            }
        }
        return true
    }
}
```

```Swift 
let rules: [(String) -> Bool] = [
    { !$0.isEmpty },
    { $0.contains("@") },
    { $0.hasSuffix(".com") }
]

if email.isValid(using: rules) {
    // ...
}
```

- 로직 분리 : 뷰에서 검증 조건을 직접 안쓰고 String type 확장 안에 숨김
- 재사용성 : 그래서 다른 뷰에서도 isVaild(using:)만 쓰면 됨
- 변경 쉬움 : 조건이 바뀔 때도 rules 배열만 바꾸면 됨
- 테스트 용이 : 유닛 테스트 가능
- 가독성 : 아무튼 보기 쉬워짐. 유효한가? 만 보면 됨 
		결국? 반복되는 코드 제거하고 재사용성 높여서 **중복이 사라진다는 뜻**


## Keywords
+ [[SwiftStandardLibrary - `Sequence`나 `Collection`에서 제공하는 `lazy` 연산은 어떻게 동작하며, 성능 최적화 측면에서 어떤 상황에 `lazy`를 사용하는 것이 적절한가?]]
- [[클로저 (Closure)]]
- [[map/filter]]
- SwiftUI 로직 분리
- 성능 최적화
## References
- [[고차 함수 (Higher-Order Functions)]]
## 작성자
#Zhen 