>[!question]
>GQ1. async let 가 뭘까?
>GQ2. TaskGroup 가 뭘까?
>GQ3. async let과 TaskGroup의 차이는 뭘까?
>GQ4. async let과 TaskGroup 각각의 장단점은 뭘까?

## Description
- GQ1. async let 가 뭘까?
- GA1. async let은 동시에 여러 작업을 실행하고, 나중에 결과를 기다릴 수 있게 해주는 문법이다.
쉬운 예시
```swift
async let 라면 = 끓이기(라면)
async let 계란 = 삶기(계란)

let 식사 = await (라면, 계란)
```
예시
```swift
func fetchImage() async -> String {
    // 시간이 오래 걸리는 작업 시뮬레이션
    return "🖼️"
}

func fetchTitle() async -> String {
    return "📄"
}

func loadContent() async {
    async let image = fetchImage()
    async let title = fetchTitle()

    // 동시에 시작된 두 작업의 결과를 기다림
    let (img, ttl) = await (image, title)
    print(img, ttl)
    // 출력: 🖼️ 📄 
}
```
- GQ2. TaskGroup 가 뭘까?
- GA2. TaskGroup은 동시에 여러 작업을 실행하고, 그 결과를 모아서 처리할 수 있게 해주는 기능이다.
쉬운 예시
```swift
let gimbap = await makeGimbap()
let eggRoll = await makeEggRoll()
let fruit = await sliceFruit()

await withTaskGroup(of: String.self) { group in
    group.addTask { await makeGimbap() }
    group.addTask { await makeEggRoll() }
    group.addTask { await sliceFruit() }

    for await item in group {
        print("완성된 도시락 구성: \(item)")
    }
}
// 실행 타이밍에 따라 순서가 달라짐.
// 완성된 도시락 구성: 과일
// 완성된 도시락 구성: 김밥
// 완성된 도시락 구성: 계란말이
```
예시
```swift
func fetchImage(id: Int) async -> String {
    return "🖼️ \(id)"
}

func loadImages() async {
    await withTaskGroup(of: String.self) { group in
        for i in 1...3 {
            group.addTask {
                await fetchImage(id: i)
            }
        }

        for await image in group {
            print("받은 이미지:", image)
        }
    }
}
// 실행 타이밍에 따라 순서가 달라짐.
// 받은 이미지: 🖼️ 2
// 받은 이미지: 🖼️ 1
// 받은 이미지: 🖼️ 3
```

## 차이
+ GQ3. async let과 TaskGroup의 차이는 뭘까?
+ GA3. 
|     항목     |          async let          |       TaskGroup       |
| 작업 개수 |    고정 (보통 2~4개)    |  동적 (반복문 가능)  |
| 선언 위치 |      코드 상에서 직접      |  반복문 등 유연하게  |
| 결과 수집 |  튜플 또는 각각 await  | for await 또는 배열 수집 |
| 병렬 처리 |             간단함             |   복잡한 상황에 적합)   |
| 에러 처리 |             제한적             |  try/catch 유연하게 가능  |

## 장단점
+ GQ4. async let과 TaskGroup 각각의 장단점은 뭘까?
+ GA4.
- async let 의 장점은, 한줄로 비동기 작업을 시작할 수 있어 간단하고 직관적이다.
그래서 코드 가독성이 좋고, 튜플로 결과를 받을 수 있어 깔끔하다.
적은 작업에 적합하여 2-4개 병렬작업을 처리할 때 매우 편하다.
* async let 의 단점은, 반복문으로 동적으로 만들 순 없다.
작업 취소가 어렵고, 튜플로 묶여있다면 결과 순서도 조절하기 어렵다.
* TaskGroup 의 장점은, 반복문과 조건문으로 자유롭게 작업 생성이 가능하다.
먼저 끝난 작업부터 바로 처리할 수 있으며, 에러 처리에도 유연하다.
* TaskGroup 의 단점은, 클로저() 안에서 작성해야하고 코드가 길어질 수 있다.
때문에 초보자에게 생소하며, 익숙하지 않으면 헷갈릴 수 있다.

## Keywords
[[병렬 처리(Swift-Concurrency)]]

## 작성자
- #Hama 