>[!question]
>GQ1. `Sequence`나 `Collection`에서 제공하는 `lazy` 연산은 어떻게 동작하며, 성능 최적화 측면에서 어떤 상황에 `lazy`를 사용하는 것이 적절한가?

## Description
- `lazy`  : 게으른 계산, 즉 **필요할 때까지 계산을 미루는 것**. 
		-예를 들어, 어떤 리스트가 있을 때, 그 안의 모든 요소를 두 배로 바꾸는 코드가 있다고 해자. 일반적으로는 리스트 전체를 한 번에 계산해버리지만 `lazy`를 쓰면, 실제로 **그 값을 꺼낼 때까지는 계산을 하지 않음.**
- 왜 성능 최적화에 도움이 될까? : 불필요한 계산을 안 하게 도와줌. 특히 `filter`, `map`처럼 요소 하나하나를 반복하며 계산할 때, **모든 요소를 계산하지 않고 중간에 멈출 수 있어서**성능이 좋아짐. 
## 주요 기능
| 기능                            | 설명                                       |
| ----------------------------- | ---------------------------------------- |
| `lazy.map { ... }`            | 리스트의 각 요소를 변환하는 작업을 **나중에** 수행           |
| `lazy.filter { ... }`         | 리스트에서 조건에 맞는 요소만 골라내는 작업을 **필요할 때까지** 미룸 |
| `first(where:)` 같은 조합과 궁합이 좋음 | 조건을 만족하는 첫 요소를 찾자마자 연산을 멈출 수 있음          |

## 코드 예시

```
let numbers = Array(1...1_000_000)

// 일반 map: 전부 계산해버림
let doubled = numbers.map { $0 * 2 } // 메모리 낭비 가능성

// lazy map: 실제 값을 사용할 때까지 계산 안 함
let lazyDoubled = numbers.lazy.map { $0 * 2 }

// 예: 딱 하나만 찾을 때는 성능 차이 큼!
if let firstOver1000 = lazyDoubled.first(where: { $0 > 1000 }) {
    print("처음 1000 넘는 수는 \(firstOver1000)")
}

```

- `numbers.map`은 **100만 개 모두**를 곱셈 연산하고 저장함.

- `numbers.lazy.map`은 `first(where:)`를 만나기 전까지는 **계산하지 않음**. 1부터 하나씩 계산하다가 501에서 `> 1000`조건을 만족하자마자 끝!

## Keywords
+ [[Swift Standard Library]]

## References
- - Apple 공식 문서: [Swift Documentation - LazySequence](https://developer.apple.com/documentation/swift/lazysequence)
- Swift Evolution: [SE-0068 - Expanding Swift's `lazy`](https://github.com/apple/swift-evolution/blob/main/proposals/0068-universal-lazy.md)

## 작성자
- #Zhen 