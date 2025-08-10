## 에러 처리 핵심 연산자들

### A. `catch` — 실패 시 “다른 퍼블리셔로 대체”

- 목적: 업스트림이 실패하면 **새 퍼블리셔를 리턴**해 파이프라인을 계속 진행. 실패 원인별로 분기 가능.
- 특징: 대체 퍼블리셔도 같은 `Output`/`Failure`를 가져야 함(그래서 여전히 실패할 수 있음). throw는 못 함(던지고 싶다면 `tryCatch`).

```Swift
urlSession.dataTaskPublisher(for: aURL)
  .catch { _ in URLSession.shared.dataTaskPublisher(for: fallbackURL) }
  .sink( ... )
```
- 언제 쓰나: 재시도 로직을 직접 구성(지연 후 재시작 등), 백업 API로 대체, 로컬 캐시로 대체 등

### B. `tryCatch` — 실패 시 “대체 + 경우에 따라 새로운 에러 throw”

- 목적: `catch`와 동일한 복구를 하되, 조건에 따라 에러를 다시 던질 수 있음
- 예: 서버가 `Retry-After`를 준 경우엔 일정 시간 딜레이 후 다시 시도, 그 외에는 그대로 실패 전파
```swift
upstream
  .tryCatch { error -> AnyPublisher<Output, Error> in
    if shouldRetry(error) {
      return Just(())
        .delay(for: .seconds(2), scheduler: DispatchQueue.global())
        .flatMap { upstream }   // 파이프라인 재시동
        .eraseToAnyPublisher()
    }
    throw error // 그대로 실패 전파
  }
```

### C. `replaceError(with:)` — 실패를 “값 하나로 바꾸고 정상 완료”

- 목적: 에러 대신 **대체 값 하나**를 방출하고 스트림을 `.finished`로 끝냄.
- 특징: 이후 실패하지 않는 퍼블리셔(`Failure == Never`)로 바뀜. 계속 새로운 값이 와야 하는 입력 스트림에 남용하면, 한 번 실패 후 파이프라인이 끝나서 갱신이 멈출 수 있음(주의)
```swift
searchPublisher           // Output: [Repo], Failure: Error
  .replaceError(with: []) // Output: [Repo], Failure: Never (즉시 정상 완료)
```

### D. `mapError(_:)` — 에러 타입 “변환/정규화”

- 목적: 다양한 업스트림 에러를 도메인 전용 에러로 바꿔서 타입 정합성과 처리 단순화. 동작 자체는 실패를 복구하지 않음(그대로 실패 완료). 
```swift
.decode(type: User.self, decoder: JSONDecoder())
.mapError { MyDomainError.decoding($0) }
```

### E. `retry(_:)` — 실패 시 “자동 재구독”

- 목적: 업스트림이 실패하면 지정 횟수만큼 자동 재시도. 기본은 즉시 재시도(딜레이/백오프는 직접 결합). 지정 횟수 모두 실패하면 원래 실패를 전파
```swift
urlSession.dataTaskPublisher(for: url)
  .retry(3)
```
- 딜레이/백오프가 필요하면 `catch`/`tryCatch` + `delay`/`flatMap`으로 구현(예시 위 `tryCatch` 코드)

### F. `try*` 계열 (`tryMap`, `tryFilter`, …) — 클로저가 throw 가능

- 목적: 변환 중 `throw`를 허용하여, 퍼블리셔의 `Failure`를 자동으로 `Error`로 승격.
- 예: JSON→모델 변환, 검증 실패 등에서 간결. 
```swift
.tryMap { data in
  guard let value = parse(data) else { throw ParseError.invalid }
  return value
}
```

### G. 그 외 알아두면 좋은 것들

- `setFailureType(to:)`: 원래 실패하지 않는 퍼블리셔(예: `Just`, `Empty`)에 명시적 Failure 타입을 부여해 연산자 체인 연결.
- `assertNoFailure()`: 개발 중 “여기선 절대 실패 안 나와야 함”을 디버그 어서션으로 강제(런타임 크래시 가능, 프로덕션 신중).

| 상황                        | 전략/연산자                                        | 이유                                 |
| ------------------------- | --------------------------------------------- | ---------------------------------- |
| “실패해도 다른 소스로 계속”          | `catch` / `tryCatch`                          | 대체 퍼블리셔로 **흐름 유지**, 조건부 재시도/폴백 구현. |
| “일시적 오류 재시도”              | `retry(_:)` (+ 필요시 `delay`, `catch/tryCatch`) | 네트워크 흔들림 등에서 자동 재구독. 지연·백오프는 조합.   |
| “UI는 값이 꼭 필요, 실패를 값으로 대체” | `replaceError(with:)`                         | 기본값으로 마감하                          |
| “에러 타입 통일/도메인화”           | `mapError(_:)`                                | 호출부 하나에서 일관 처리 가능.                 |
| “변환 중 검증/throw 필요”        | `tryMap` 등                                    | 변환 실패를 Combine 실패로 자연스럽게 전파.       |