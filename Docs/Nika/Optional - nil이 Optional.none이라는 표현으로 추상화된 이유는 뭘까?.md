Swift에서는 `nil`이 단순히 '비어있음'의 의미가 아니라 `Optional.none`이라는 구체적인 타입 값으로 추상화되어 있다. 이는 Swift의 철학인 **타입 안정성과 안전한 프로그래밍**을 위해서이다.
Swift에서는 **값이 없을 수 있다**는 사실을 명시적인 타입으로 모델링한다.

Optional은 다음과 같이 정의되어 있다:
```
enum Optional<Wrapped> {
	case none
	case some(Wrapped)
}
```

그래서 `nil`은 그냥 '포인터 없음(`null`)'이 아니라 `Optional.none`이라는 **enum case**로 해석된다.
```
let x: String? = nil
// 내부적으로는: x = Optional<String>.none
```
