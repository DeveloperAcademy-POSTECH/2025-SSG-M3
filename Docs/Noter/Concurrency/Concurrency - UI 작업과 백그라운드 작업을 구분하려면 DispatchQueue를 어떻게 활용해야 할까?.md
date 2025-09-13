
> [!question]
> GQ. Concurrency - UI 작업과 백그라운드 작업을 구분하려면 DispatchQueue를 어떻게 활용해야 할까?

### 0. 키워드 개념 정리

1. **Main Thread (메인 스레드)**
    앱이 실행되면 시작되는 기본 실행 흐름으로, 대부분의 UI 업데이트 및 사용자 인터랙션이 이 스레드에서 처리된다. iOS는 단일 UI 스레드 모델을 따르기 때문에, UI 변경은 반드시 메인 스레드에서 이루어져야 한다.
    
2. **Background Thread (백그라운드 스레드)**
    사용자 인터페이스와 직접적 연관이 없는 작업(네트워크 요청, 파일 읽기, 이미지 처리 등)은 메인 스레드의 부담을 줄이기 위해 백그라운드 스레드에서 실행하는 것이 일반적이다.
    
3. **DispatchQueue (디스패치 큐)**
    Apple의 GCD(Grand Central Dispatch) 시스템의 핵심 구성 요소. 작업을 큐에 등록하면 시스템이 알맞은 스레드에서 실행해준다.
    
    → DispatchQueue.main은 메인 스레드, DispatchQueue.global()은 백그라운드 스레드용 큐.
    
4. **동기(Synchronous)와 비동기(Asynchronous)**
- sync: 현재 스레드를 차단(block)하고 작업이 끝날 때까지 기다림
- async: 현재 스레드를 차단하지 않고, 작업을 다른 스레드에서 병렬로 수행

### 1. 개요

DispatchQueue는 스레드 직접 관리 없이도 작업을 다양한 스레드에서 안전하게 실행할 수 있도록 도와주는 GCD(Grand Central Dispatch)의 도구이다. 특히 iOS에서는 UI는 메인 스레드, 비용이 큰 연산은 백그라운드 스레드에서 처리해야 하는데, 이 구분을 명시적이고 안전하게 수행하게 해주는 수단이 바로 DispatchQueue이다. 

### 2. 왜 DispatchQueue를 사용해야 할까?

|**필요성**|**이유**|
|---|---|
|UI 응답성 유지|무거운 작업(예: 이미지 로딩, 네트워크 요청)이 메인 스레드를 점유하면 앱이 **멈춘 듯 느껴질 수 있음**|
|스레드 간 작업 분리|UI는 **메인 스레드**, 그 외 작업은 **백그라운드 스레드**로 나눠야 충돌 방지|
|스레드 관리 최소화|직접 스레드를 생성/해제하는 복잡한 코드 없이 **간결한 문법**으로 병렬 처리 가능|

### 3. 사용 예시

**3.1 백그라운드에서 무거운 작업 실행 + UI 업데이트**

```swift
DispatchQueue.global().async {
    let data = fetchDataFromDisk() // 디스크에서 데이터 읽어오는 함수. 시간이 오래 걸리는 작업
    
    DispatchQueue.main.async {
        self.label.text = data.title // UI는 반드시 메인 스레드에서
    }
}
```

메인 스레드는 UI 전용. 느려지면 앱이 버벅이거나 멈춘 것처럼 보임.
fetchDataFromDisk()는 시간이 걸리는 작업 (예: 디스크에서 파일 읽기, 이미지 로딩, 큰 JSON 파싱 등)
→ 백그라운드에서 실행해야 앱이 느려지지 않음
작업이 끝난 후에는 UI를 메인스레드에서 안전하게 업데이트해야 함.

`DispatchQueue.global()`
 : GCD가 제공하는 시스템 전역(global) 큐 중 하나를 반환
 큐들은 우선수위에 따라 나뉘어져 있다.
 ```swift
DispatchQueue.global(qos: .userInitiated)
DispatchQueue.global(qos: .background)
DispatchQueue.global(qos: .default)
DispatchQueue.global(qos: .utility)
 ```

QoS(Quality of Service)란?
QoS는 이 작업이 얼마나 중요한지를 시스템에 알려주는 등급!

.userInteractive: 가장 높은 우선순위. UI 반응 즉시 필요할 때 ex) 스크롤, 버튼 애니메이션
.userInitiated: 사용자 액션의 결과가 UI에 바로 반영돼야 할 때 ex) 사진 필터 처리
.default: 특별한 우선순위가 필요 없는 일반 작업 ex) 디폴트
.utility: 오래 걸리지만 급하지 않은 작업 ex) 파일 다운로드, 로그
.background: 사용자와 관계 없는 작업 ex) 백업, 데이터 정리

아무 옵션 없이 DispatchQueue.global()만 쓰면 .default QoS가 적용된다.

- Swift의 GCD(Grand Central Dispatch)는 Apple이 제공하는 저수준의 멀티스레딩 API로, 앱에서 동시성을 쉽게 다룰 수 있게 해준다.
	- 우리가 직접 스레드를 만들지 않아도 GCD가 스레드 풀을 관리
	- 작업을 큐에 넣기만 하면 적절한 스레드에 자동으로 분배해줌
	- 성능, 안전성, 효율성이 좋음
- 시스템이 제공하는 글로벌 백그라운드 큐를 호출
- 일반적으로 CPU 작업이나 I/O 작업을 수행하는 데 사용

`async { ... }`
- 큐에 작업을 비동기로 제출, 즉 현재 흐름을 차단하지 않고 바로 다음 코드로 넘어감.
- 작업을 큐에 제출만 하고 바로 다음 줄(async{} 블록 바깥)로 이동한다는 뜻~

```swift
DispatchQueue.global().async {
```
이 부분은 이런 흐름인 것!
1. DispatchQueue.global() → 글로벌 큐를 반환
2. .async { ... } → 해당 큐에 작업을 보냄
어떤 큐에 작업을 보낼지 정한 다음 거기에 작업을 보낸다.


**3.2 특정 작업 지연 실행**

```swift
DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
    print("2초 뒤 실행되는 코드")
}
```

`DispatchQueue.main.asyncAfter(deadline: .now() + 2.0){ ... }`
: "지금으로부터 2초 후에 메인스레드에서 이 코드를 실행해."
- 내부적으로는 타이머처럼 동작
- 특정 시간 후에 큐에 작업을 넣음

실행 순서 흐름
```swift
print("A")

DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
    print("B")
}

print("C")
```

이렇게 작성하면 실제 출력 순서는?
A
C
(2초 후)
B

왜냐면?
- A → 바로 출력
- asyncAfter → 2초 후에 실행 예약만 하고 넘어감
- C → 바로 출력
- B → 2초 지난 후 메인 스레드에서 실행


**3.3 사용자 정의 큐 사용**

```swift
let myQueue = DispatchQueue(label: "com.noter.mySerialQueue")
myQueue.async {
    // 순차적으로 실행될 작업 - 비동기로 등록
}
```

"내가 만든 직렬 큐(serial queue)를 통해 작업을 순차적으로 실행시켜줘"
즉, 나만의 이름을 가진 큐를 만들고 그 큐에 작업을 비동기로 등록해 하나씩 차례대로 실행하게 하는 것!

- `DispatchQueue(label:)`→ 커스텀 큐(사용자 정의 큐) 만들기
- label은 큐 이름. 디버깅하거나 로깅할 때 유용. 일반적으로 `com.회사이름.기능` 형식 사용
- 기본적으로 이 큐는 serial 큐로 생성된다. 큐에 들어온 작업을 하나씩, 순서대로 처리한다는 뜻
- 이렇게 만든 큐는 여러 번 재사용할 수 있다.

예시
```swift
let loggingQueue = DispatchQueue(label: "com.noter.logging")

func logEvent(_ message: String) {
    loggingQueue.async {
        // 로그는 순차적으로 출력돼야 보기 좋으니께
        print("[LOG]: \(message)")
    }
}
```
→ 여러 이벤트가 동시에 발생하더라도 logginQueue는 한 줄씩 차례로 로그를 찍는다.

### 4. 실무 팁과 주의점

- **UI 업데이트는 항상 메인 스레드에서**
    → DispatchQueue.main.async { ... }로 감싸는 습관 들이기
    
- **비싼 작업은 무조건 백그라운드에서**
    → 이미지 처리, JSON 파싱, 파일 입출력 등은 DispatchQueue.global().async 사용
    
- **스레드 전환은 최소화**
    → 자주 스레드를 전환하면 오히려 **오버헤드**가 생길 수 있으므로 최소화


### 5. 느낀 점과 한계

- DispatchQueue는 Swift에 내장된 만큼 강력하고 성능 최적화된 도구다.
    
- 그러나 구조화된 흐름 제어에는 불편함 있다.
    → 비동기 코드가 중첩되면 콜백 지옥(callback hell)처럼 복잡해지기도 함.
    
- 최근의 async/await 구조에 비해 **가독성과 유지보수성이 떨어질 수 있음**
    → 예: 중첩된 DispatchQueue.main.async / DispatchQueue.global().async 구조는 흐름 파악이 어렵다.
    
- 복잡한 비동기 로직이 많아질수록 DispatchQueue만으로는 흐름을 추적하고 디버깅하기 어려움

### 정리

- DispatchQueue는 Swift의 GCD 기반 병렬 처리 시스템의 핵심이다.
- UI 작업은 DispatchQueue.main, 무거운 작업은 DispatchQueue.global()에 분리해서 실행하자.
- async, asyncAfter를 활용하면 비동기 코드나 지연 실행도 쉽게 구현 가능하다.
- 가독성과 흐름 제어가 중요하다면 Swift의 async/await, Task 구조와 병행해 사용하는 것이 좋다.

#### Keyword
[[Concurrency]]


#Noter