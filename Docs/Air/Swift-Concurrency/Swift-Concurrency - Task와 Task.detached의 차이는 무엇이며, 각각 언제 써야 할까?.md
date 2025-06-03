>[!question]
>GQ. Task와 Task.detached의 차이는 무엇이며, 각각 언제 써야 할까?
## Description
- `Task`와 `Task.detached`는 Swift에서 비동기 작업을 생성할 때 사용합니다.
- 둘 다 비동기적을 작업을 수행하지만, context와 상속된 환경의 처리 방식에서 차이가 있다.

| 항목       | Task                     | Task.detached              |
| -------- | ------------------------ | -------------------------- |
| 부모-자식 관계 | 부모 Task의 context를 상속     | 완전히 독립적인 Task 생성           |
| actor 상속 | 현재 actor context를 계승     | actor context를 계승하지 않음     |
| 사용 목적    | 구조화된 비동기 흐름, actor 내부 작업 | 독립 실행이 필요한 경우, actor 외부 작업 |
| 취소 전파    | 부모의 취소 상태를 전파받음          | 취소 상태를 전파받지 않음             |

## 주요 기능
| 상황                        | Task 사용                | Task.detached 사용          |
| ------------------------- | ---------------------- | ------------------------- |
| 현재 actor, view model 내부에서 | 적합 (context를 따라감)      | 위험 (actor 외부로 나까니까 충돌 위험) |
| 사용자와 연결된 UI 작업            | 적합                     | 부적절                       |
| 독립적인 백그라운드 작업 필요 시        | 부적합 (불필요한 context 상속됨) | 적합 (깨끗한 환경에서 실행됨)         |
| 취소, 에러 전파 필요              | 적합 (부모 Task와 함께 처리됨)   | 부적합 (독립적이므로 직접 처리해야 함)    |

## 코드 예시
#### Task
```Swift
actor Counter {
    var value = 0

    func increase() {
        // actor context 안에서 안전하게 실행됨
        Task {
            value += 1  // actor가 보장해줌
            print("Increased to \(value)")
        }
    }
}
```

#### Task.detached 사용 예시
```Swift
func performHeavyTaskInBackground() {
    Task.detached {
        // 완전히 독립된 환경에서 실행
        let data = await downloadBigData()
        print("Downloaded data: \(data)")
    }
}

func downloadBigData() async -> String {
    // 비동기 네트워크 작업
    return "Gigabytes of data"
}
```

## Keywords
+ [[병렬 처리(Swift-Concurrency)]]
+ [[Task]]
+ [[Task.detached]]

## References
- 

## 작성자
- #Air 