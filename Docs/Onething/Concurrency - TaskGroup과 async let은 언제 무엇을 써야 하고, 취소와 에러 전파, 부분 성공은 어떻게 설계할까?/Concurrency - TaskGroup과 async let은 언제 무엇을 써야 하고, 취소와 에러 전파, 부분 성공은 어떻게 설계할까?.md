
> [!question]
> **GQ**: TaskGroup과 async let은 언제 무엇을 써야 하고, 취소와 에러 전파, 부분 성공은 어떻게 설계할까?

## 개요
동시성 코드를 빨리 돌게 만드는 것보다 중요한 건 예측 가능성과 정책이다.  
이 글은 `async let`과 `TaskGroup`을 **언제 고르는지**를 한눈에 정리하고, **취소·에러 전파·부분 성공**을 **어떤 정책으로 설계**해야 안정적인 앱이 되는지 정리한다.

---

## 1) 한눈 비교 — 역할과 선택 기준
### 언제 무엇을 쓸까

| 상황                          | 권장 도구                          | 이유                         |
| --------------------------- | ------------------------------ | -------------------------- |
| **작업 개수 고정, 소수**(2~3개)      | `async let`                    | 문법 짧고 명확, 스코프 안에서 깔끔하게 수거  |
| **작업 개수 가변/많음**(데이터에 따라 N개) | `TaskGroup`                    | 동적으로 추가/회수 가능              |
| **첫 성공만 쓰고 나머지 취소**(race)   | `TaskGroup`                    | `cancelAll()` 등 정책 제어 용이   |
| **부분 성공 허용**(실패 스킵)         | `TaskGroup`(non-throwing)      | 결과를 `Optional/Result`로 모델링 |
| **동시 실행 개수 제한 필요**          | `TaskGroup`                    | 수동 rate limit 패턴 적용 쉬움     |
| 입력 순서대로 결과 필요               | `async let` 또는 그룹에서 **키로 재정렬** | 순서 보존이 목표면 간단한 쪽 선택        |
  
### 공통 안전망(Structured Concurrency)
- 부모 스코프가 자식 태스크 수명 소유
- 스코프 종료 시 자식 정리
- 부모 취소 → 자식 취소 **전파**

---
## 2) 대표 사용 패턴
### `async let` — 고정 병렬

```swift

func loadProfile() async throws -> Profile {
  async let user  = api.fetchUser()
  async let prefs = api.fetchPreferences()
  return try await Profile(user: user, prefs: prefs) // 스코프에서 반드시 await로 수거
}
```
- 간단 병렬에 최적. 수십 개 남발은 금물(그럴 땐 그룹).

### **포인트**

- **스코프 말미에 미수거 금지**: async let 변수는 **반드시 await**해야 한다.
- **에러 전파**: try await 하는 시점에 **throw**가 튄다. 다른 async let이 **아직 돌고 있다면** 남은 것들도 **결국 기다려 정리**한다.
- **취소**: 부모가 취소되면 자식도 취소. 자식 작업 안에서 **cooperative** 하게 반응해야 빨리 끝난다.
```swift
try Task.checkCancellation() // 또는 if Task.isCancelled { return }
```

### **안티패턴**
- async let를 수십 개 남발 → 스레드 폭주 위험. 이런 건 TaskGroup에서 **수 제한**을 걸자.

---

##  TaskGroup — 정책을 가진 병렬 처리

### **도착 순 수집(부분 성공 허용)**
```swift
struct Photo { let id: String }
func fetchPhoto(id: String) async throws -> Data { /* ... */ }
// 실패는 스킵, 성공만 모으는 non-throwing 그룹
func downloadPhotos(ids: [String]) async -> [Data] {
  await withTaskGroup(of: Data?.self) { group in
    for id in ids {
      group.addTask {
        do { return try await fetchPhoto(id: id) }
        catch { return nil } // 실패는 값으로 표현(nil)
      }
    }
    var result: [Data] = []
    for await data in group {
      if let data { result.append(data) } // 부분 성공만 누적
    }
    return result
  }
}
```
> “부분 성공 허용”은 **non-throwing** 그룹 + Optional/Result가 깔끔.
> “하나라도 실패하면 전체 실패”는 **withThrowingTaskGroup**이 적합.
    
### **첫 성공만 채택하고 나머지 취소(race)**

```swift
// 여러 엔드포인트 중 가장 빠른 성공만 채택
func firstReachableURL(_ urls: [URL]) async -> URL? {
  await withTaskGroup(of: URL?.self) { group in
    for url in urls {
      group.addTask { await Ping(url).isOK ? url : nil }
    }
    for await candidate in group {
      if let ok = candidate {
        group.cancelAll()   // 정책: 첫 성공 이후 즉시 나머지 취소
        return ok
      }
    }
    return nil
  }
}
```
> 네트워크 다중 소스 탐색, 이미지 CDN fallbacks 같은 곳에 유용.

### **동시 실행 개수 제한(간단 rate limit)

```swift
func fetchAll(_ urls: [URL], limit: Int = 4) async -> [Data] {
  await withTaskGroup(of: Data?.self) { group in
    var iterator = urls.makeIterator()
    var inflight = 0
    var out: [Data] = []

    // 초기 시드 채우기
    while inflight < limit, let u = iterator.next() {
      inflight += 1
      group.addTask { try? await api.fetch(u) }
    }

    // 하나 끝날 때마다 하나 더 넣기
    while let data = await group.next() {
      inflight -= 1
      if let data { out.append(data) }
      if let u = iterator.next() {
        inflight += 1
        group.addTask { try? await api.fetch(u) }
      }
    }
    return out
  }
}
```
> 폭주 방지. 썸네일 수백 장 로딩 같은 곳에서 **메모리/스레드 압력**을 안정화.



---
## 취소와 에러 전파 — 정책으로 설계하기


### 전파 규칙 요약**
- **부모 취소 → 자식 자동 취소**(둘 다)
- **Throwing TaskGroup**: 어떤 자식이 던지면 → **나머지 취소 + rethrow**
- **async let**: await 시점에 throw. **다른 자식도 정리**해야 스코프가 닫힌다.
  
### **Cooperative cancellation**

자식은 **직접** 취소 신호를 체크해야 빨리 끝난다.
```swift
func decode(_ data: Data) async throws -> Model {
    try Task.checkCancellation()
    // 무거운 파싱...
    return model
}
```

### **부분 성공을 의도적으로 허용**

- Non-throwing group으로 만들고, **오류는 값으로 싣는다**(Result, Optional)
- 최종 단계에서 **필터링/집계**:
```swift
var oks: [Output] = results.compactMap { try? $0.get() }
```

## **테스트 전략 — 결정론을 높이는 요령**

- **가짜 비동기 작업**으로 지연·실패·취소를 조합
```swift
struct Stub {
    var delay: UInt64
    var result: Result<Int, Error>
    func run() async throws -> Int {
        try await Task.sleep(nanoseconds: delay)
        try Task.checkCancellation()
        return try result.get()
    }
}
```
- **race 시나리오**: 느린 성공 vs 빠른 실패 → **정책대로** 동작하는지
- **취소 후 UI 미업데이트**: 취소되면 ViewModel가 **상태 변경 안 함** 검증
- **부분 성공**: 실패 섞여도 결과 배열이 **비어 있지 않은지** 확인


## **자주 터지는 함정과 회피법**

> [!warning] 안티패턴
> - withTaskGroup 안에서 또 Task {} 생성(구조 깨짐) → **그룹 밖에서** 만들거나 **그룹으로 통일**
> - async let 수십 개 생성(폭주) → **그룹 + rate limit**
> - 취소 신호 무시하고 마지막에 UI 업데이트 → **Task.isCancelled 체크 후 조용히 탈출**
> - @MainActor 아닌 컨텍스트에서 UI 변경 → await MainActor.run { ... } or @MainActor 메서드로 격리
> - 결과 순서가 중요할 때 도착 순으로 바로 append → **키로 재정렬**(원본 인덱스 보관)

## **검색 자동완성(최신만 유지)**
```swift
// 1) 사용자가 입력할 때마다 네트워크 요청
@MainActor
final class SearchVM: ObservableObject {
    @Published var query = ""
    @Published private(set) var results: [Item] = []
    private var currentTask: Task<Void, Never>?

    func onQueryChanged(_ q: String) {
        currentTask?.cancel()                 // 이전 작업 취소
        guard q.count >= 2 else { results = []; return }

        currentTask = Task { [q] in
            do {
                let items = try await fetchMergedSources(q) // TaskGroup 내부에서 race/switch
                try Task.checkCancellation()
                await MainActor.run { self.results = items } // 최신 쿼리만 반영
            } catch is CancellationError {
                // 조용히 종료
            } catch {
                await MainActor.run { self.results = [] }
            }
        }
    }
}

// 2) 내부 구현: 여러 소스 동시 조회, 최신 것만 채택
func fetchMergedSources(_ q: String) async throws -> [Item] {
    try await withThrowingTaskGroup(of: [Item].self) { g in
        g.addTask { try await apiA.search(q) }
        g.addTask { try await apiB.search(q) }
        var merged: [Item] = []
        for try await part in g {
            merged = merge(merged, part) // 중복 제거/정렬
        }
        return merged
    }
}
```
- 입력이 바뀌면 **이전 Task 취소**(UI “뒤늦은 응답” 방지)
- 내부는 그룹으로 멀티 소스 병렬화. 실패는 throw → 상위에서 처리.


## 최종 체크리스트
- 고정 개수면 async let, 가변/정책 필요면 TaskGroup
- **취소 전파**를 디자인에 포함(부모 취소, 첫 성공 후 cancelAll)
- **부분 성공**은 값으로 모델링(Non-throwing + Optional/Result)
- **동시 실행 제한**이 필요하면 rate limit 패턴
- UI 업데이트는 **MainActor** 보장
- 테스트에서 지연·실패·취소 시나리오 **필수**



---

## **Finding & Synthesis**

- 성능보다 중요한 건 **정책**이다: “첫 성공만”, “부분 성공 허용”, “최신만 반영” 같은 규칙을 코드로 만들면 동시성은 안정적이 된다.
- async let은 **간단병렬**의 왕도, TaskGroup은 **정책 병렬**의 도구. “언제 무엇을”만 잘 고르면 설계가 쉬워진다.
- **취소는 자동이 아니다**: 전파는 자동이지만, **빠른 종료**는 작업 안에서의 **협력**으로 완성된다.
- UI와 동시성을 섞을 땐 “**최신만 반영**”과 @MainActor 보장이 기본 안전장치다.

## **Next GQ**
- **Cancellation을 UI·네트워크 계층 전반으로 일관되게 설계하려면 어떤 인터페이스가 좋은가?**
    (예: Cancelable 프로토콜, ViewModel의 최신 요청만 유지, 네트워크 타임아웃 정책)

#Onething 