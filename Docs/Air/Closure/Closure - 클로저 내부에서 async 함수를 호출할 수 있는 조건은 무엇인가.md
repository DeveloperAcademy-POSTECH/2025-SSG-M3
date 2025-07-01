>[!question]
>GQ. 클로저 내부에서 async 함수를 호출할 수 있는 조건은 무엇인가

## Description
- Swift에서 클로저는 코드를 그룹화하고 전달하며, 주변 컨텍스트의 변수와 상수를 캡처하고 저장할 수 있는 강력하고 유연한 기능이다.
- 과거에는 `DispatchQueue`를 활용한 GCD나 클로저 기반의 완료 핸들러(completion handler)가 비동기 작업을 처리하는 주된 방식이었다.
- `async/await`와 기존의 클로저 기반의 코드 및 API와의 통합은 중요한 과제이다.

## 주요 기능
#### 클로저 내 `async` 함수 호출 조건
- `async` 함수는 `await` 키워드를 사용하여 비동기 컨텍스트 내에서만 직접적으로 호출될 수 있다.
- `async`가 아닌 클로저에서 `async`함수를 호출 하려면 해당 호출을 `Task` 블록으로 래핑하여 새로운 비동기 컨텍스트를 생성해야 한다.
- `Task {...}`는 새로운 동시성 작업을 생성하고 즉시 실행을 시작하는 역할을 하며, 이 또한 클로저의 한 형태이다.

## 코드 예시
1. 일반 클로저 내 `Task` 블록을 사용하여 `async` 함수를 호출
```Swift
func performLegacyClosure(completion: @escaping (String) -> Void) {
	// 이 클로저는 async 컨텍스트가 아님
	DispatchQueue.global().async {
		// Task 블록 내에서 async 함수 호출
		Task {
			do {
				let data = try await fetchDataAsync()
				completion("Data fetched: \(data.count)bytes")
			} catch {
				completion("Error: \(error.localizedDescription)")
			}
		}
	}
}

func fetchDataAsync() async throws -> Data {
	try await Task.sleep(for:.seconds(1))
	return Data("Hello, Swift Concurrency!".utf8)
}
```
2. `withCheckedContinuation`을 사용하여 클로저 기반 API를 `async/await`로 래핑
```Swift
// async/await로 래핑: 기존 클로저를 async 함수처럼 사용할 수 있게 됨
func fetchDataAsyncWrapped() async -> Data {
	return await withCheckedContinuation { coutinuation in
		fetchData { data in
			coutinuation.resume(returning: data ?? Data()) // 클로저 결과가 나오면 continuation 재개
		}
	}
}

// 기존 클로저 기반 함수
func fetchData(completion: @escaping (Data?) -> Void) {
	DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
		completion(Data("Some data".uft8))
	}
}
```
## Keywords
+ [[클로저 (Closure)]]
+ [[Task]]
+ [[병렬 처리(Swift-Concurrency)]]

## 작성자
- #Air 