>[!question]
>GQ. 구조화된 동시성이란 무엇이며, Task의 생명주기를 어떻게 관리해주는가?

## Description
- Swift의 구조화된 동시성(Structured Concurrency)은 비동기 코드를 더 안전하고, 읽기 쉬우며, 관리하기 쉽게 만들기 위해 도입된 프로그래밍 모델이다. 이 모델의 핵심은 'Task 트리'라는 계층적 구조이다. 하나의 Task가 다른 Task를 생성하면 부모-자식 관계가 형성되며, 이 관계는 Task의 생명주기, 취소, 우선순위 등을 관리하는 기준이 된다.
- 구조화된 동시성은 Task의 생명주기를 명확한 범위(scope)에 귀속시켜 예측 가능성을 높인다. 부모 Task는 모든 자식 Task가 완료되기 전까지 종료될 수 없으며(그룹 완료 규칙), 부모 Task가 취소되면 해당 신호가 모든 자식에게 자동으로 전파된다(그룹 취소 규칙). 이 자동화된 관리는 리소스 누수(leaked task)를 방지하고, 불필요한 작업을 효율적으로 중단시켜 앱의 안정성과 성능을 향상시킨다.
- `async/await` 구문을 통해 비동기 코드를 동기 코드처럼 직선적으로 표현하여 '콜백 지옥(callback hell)' 문제를 해결하고 코드의 가독성을 크게 높인다.

## 주요 기능
- **자동화된 Task 생명주기 관리**
	- 구조화된 동시성은 Task(작업의 최소 단위)가 언제 시작하고, 언제 끝나야 하는지를 Swift가 자동으로 관리해주는 강력한 기능을 제공한다.
    - **활용 사례 1: 안전한 작업 완료 보장**
	    - 여러 개의 데이터를 서버에서 받아와 화면을 구성한다고 상상해 보자. 구조화된 동시성 모델에서는 모든 데이터 조각이 성공적으로 도착했는지 Swift가 확실히 보장해 준다. 중간에 하나의 작업이라도 누락되는 일 없이, 모든 자식 작업이 끝나야만 부모 작업이 완료되기 때문이다.
    - **활용 사례 2: 효율적인 리소스 관리**
	    - 사용자가 어떤 화면에서 데이터를 로딩하다가 완료되기 전에 다른 화면으로 이동했다고 가정해 보자. 이때 더 이상 필요 없어진 데이터 로딩 작업을 취소해야 한다. 구조화된 동시성에서는 부모 작업 하나만 취소하면, 그와 연결된 모든 자식 작업(예: 여러 이미지 다운로드)들이 연쇄적으로 취소 신호를 받는다. 이로써 불필요한 네트워크와 배터리 소모를 자동으로 막을 수 있다.
- **코드 가독성 및 유지보수성 향상**
	- 과거에는 동시성 코드를 작성할 때 콜백(callback) 함수가 중첩되어 코드가 복잡해지는 '콜백 지옥' 현상이 잦았다. 구조화된 동시성은 `async/await` 문법을 도입하여 이 문제를 해결했다.
    - **활용 사례: 순서대로 읽히는 비동기 코드** `async/await`를 사용하면 마치 일반적인 동기 코드처럼 위에서 아래로 순서대로 코드를 작성할 수 있다. "A 작업을 실행하고(`async`), 그 결과가 올 때까지 기다렸다가(`await`), 그 결과로 B 작업을 해" 와 같이 코드의 흐름을 훨씬 직관적으로 이해하고 관리할 수 있다.
- **향상된 안전성**
	- 여러 작업이 하나의 데이터에 동시에 접근하려 할 때 발생하는 '데이터 경쟁(Data Race)'은 찾기 어려운 심각한 버그의 원인이 된다. 구조화된 동시성은  Actor 및 `Sendable` 프로토콜과 결합하여 데이터 경쟁 문제를 컴파일 단계에서 미리 발견하고 방지할 수 있도록 도와준다.

## 코드 예시
+ **`TaskGroup`을 활용한 여러 이미지 병렬 다운로드**
	+ `TaskGroup`은 동적인 수의 자식 Task를 생성하고 관리하는 데 사용된다. 아래 예시는 여러 URL로부터 이미지를 동시에 다운로드하고, 모든 작업이 완료되면 결과를 배열로 반환한다. 만약 하나의 다운로드라도 실패하면, 그룹 내 다른 모든 Task가 취소되고 오류가 전파된다.
```Swift
enum ImageError: Error {
    case decodingFailed
}

func downloadAllImages(from urls:) async throws -> [UIImage] {
    // 결과를 UIImage 타입으로 반환하는 ThrowingTaskGroup 생성
    try await withThrowingTaskGroup(of: UIImage.self) { group in
        for url in urls {
            // 그룹에 동적으로 자식 Task 추가
            group.addTask {
                // 각 Task는 취소 신호를 확인해야 함
                try Task.checkCancellation()
                
                let (data, _) = try await URLSession.shared.data(from: url)
                guard let image = UIImage(data: data) else {
                    throw ImageError.decodingFailed
                }
                return image
            }
        }

        // 모든 자식 Task가 완료될 때까지 결과를 수집
        var images = [UIImage]()
        // for-try-await 루프는 완료되는 순서대로 결과를 가져옴
        for try await image in group {
            images.append(image)
        }
        
        // group 스코프가 끝나기 전에 모든 자식 Task의 완료가 보장됨
        return images
    }
}
```

## Keywords
+ [[병렬 처리(Swift-Concurrency)]]
+ [[구조화된 동시성(Structured Concurrency)]]

## References
- [Apple Developer Documentation: Explore structured concurrency in Swift (WWDC21)](https://developer.apple.com/videos/play/wwdc2021/10134)
- [# Structured concurrency in Swift explained](https://www.donnywals.com/the-basics-of-structured-concurrency-in-swift-explained/)

## 작성자
- #Air 