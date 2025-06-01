race condition은 여러 스레드(혹은 task)가 동시에 같은 자원(변수 등)에 접근해서 예상치 못한 결과가 나오는 상황을 말한다. 즉, 누가 접근하느냐에 따라서 결과가 달라지는 예측 불가능한 상황을 이른다.

예)
```
var counter = 0

DispatchQueue.concurrentPerform(iterations: 1000) { _ in
    counter += 1
}
```
여기서 counter는 1000이 나오지 않을 수 있다. 여러 스레드가 동시에 counter를 읽고 수정하고 쓰기 때문이다. `counter += 1`를 동시에 연산하다가 중간 상태에서 충돌할 경우 일부 연산이 덮어씌워져서 값이 손실될 수 있다.


Swift는 이러한 race condition을 막기 위해서 여러 도구를 제공한다.
1. Serial Queue 사용(GCD 방식)
```
let queue = DispatchQueue(label: "my.serial.queue")

for _ in 0..<1000 {
    queue.async {
        counter += 1
    }
}
```
- 한 번에 하나의 작업만 실행한다. 동시에 접근하지 않도록 serialize(직렬화)시켜준다.

2. Lock 사용(NS lock, os_unfair_lock 등)
```
let lock = NSLock()

for _ in 0..<1000 {
    DispatchQueue.global().async {
        lock.lock()
        counter += 1
        lock.unlock()
    }
}
```
- 직접 락을 걸어서 한 번에 한 스레드만 접근 가능하게 만든다.
- 사용법은 단순하지만, 락을 잘못 사용하면 데드락 등의 위험이 있다. 비추천

3. Swift concurrency: `Actor` 사용
```
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }

    func getValue() -> Int {
        return value
    }
}
```
사용:
```
let counter = Counter()

Task {
    await counter.increment()
    let value = await counter.getValue()
    print(value)
}
```
- `actor`는 내부 상태를 **자동으로 동기화**함 
- 다른 코드가 동시에 접근하지 못하게 보장
- Race Condition 방지에 매우 안전한 방법

4. Task-safe 자료 구조 사용
```
@Sendable func update(shared: UnsafeMutablePointer<Int>) {
    shared.pointee += 1
}
```
- `@MainActor`, `Sendable`, `@State`, `@ObservedObject` 등의 **SwiftUI/Concurrency 보조 도구**를 이용해 상태를 안전하게 관리할 수도 있다.

|방법|설명|특징|
|---|---|---|
|Serial Queue|작업 순차 처리|쉬움, GCD 기반|
|Lock|직접 락 걸기|세밀한 제어, 데드락 주의 필요|
|Actor|Swift Concurrency 방식|안전하고 현대적인 방식|
|@MainActor 등|상태 접근을 MainQueue에 제한|UI 상태 관리에 적합|

#Nika