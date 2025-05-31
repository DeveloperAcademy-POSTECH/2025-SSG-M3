>[!question]
>GQ1. Actor는 기존 class와 비교해 어떤 점이 더 안전할까?

## Description
- Swift Concurrency는 동시성(Concurrency)과 병렬성(Parallelism)을  안전하고 읽기 쉬운 코드로 작성할 수 있게 도와주는 Swift의 내장 기능이다. 

## 주요 기능
+  actor은 동시 접근으로 인한 충돌을 방지하기 위해 고안된 타입이다. 내부 상태에 대한 접근은 하나의 Task만 가능하기에 안전한 병렬 처리가능
+ actor은 class보다 동시성에서 안전하다. class는 여러 스레드에서 동시에 접근할 수 있기 때문에 동시성 안전이 자동으로 보장되지 않는다 개발자가 직접 동기화를 해줘야만 동시성 안전이 자동으로 보장됨.  actor은 Swift 언어 수준에서 동시성 안전을 자동으로 보장해주며, 내부 상태에 동시에 접근할 수 없도록 설계되어 있어 데이터 충돌이 발생하지 않는다.
- class는 여러 스레드에서 동시에 접근할 수 있어서  주의 깊게 관리하지 않으면 예기치 않은 동작이나 충돌이 발생할 수 있음. 반대로 `actor`는 내부적으로 직렬 큐(serial queue)로 작동하며, 한 번에 하나의 Task만 내부 상태에 접근할 수 있어서, 안전하게 병렬 처리를 구현할 수 있습니다.
- 또한 class를 사용할 경우, 병렬 환경에서의 동기화를 위해 `DispatchQueue`, `NSLock`, `Semaphore` 등과 같은 동기화 도구를 직접 사용해야함. 반면에 actor는 이러한 동기화 처리를 자동으로 처리(직렬 큐 이용)해 주기 때문에 별도의 도구 없이도 안전하게 사용 가능

## 코드 예시
+ 
```swift
// class 사용 예시
class Counter {
    var value = 0

    func increment() {
        value += 1
    }
}

```
여러 곳에서 increment를 호출 한다면, value += 1가 원자적이 아니기 때문에 값이 꼬일 수 있다. 

```swift
actor SafeCounter {
    var value = 0

    func increment() {
        value += 1
    }
}

```
- actor는 내부적으로 직렬 큐(Serial Queue)처럼 작동해서 동시에 접근되는 일이 없도록 자동 제어함. 따라서 데이터 손상과 충돌 없음
## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- #Sera 