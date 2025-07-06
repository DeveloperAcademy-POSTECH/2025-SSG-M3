>[!question]
>GQ. ARC 기반 앱의 메모리 누수를 Instruments(Leaks, Allocations 등)를 사용해 어떻게 진단하고 추적할 수 있는가?

## Description
- ARC(Automatic Reference Counting)가 지원되더라도 순환 참조로 인한 메모리 누수를 완벽히 방지하지는 못한다.
- 이러한 누수를 진단하고 추적하기 위해 Apple은 Instruments라는 강력한 프로파일링 도구 모음과 Xcode에 내장된 메모리 그래프 디버거를 제공한다.
- Instruments의 Leaks와 Allocations 프로파일러를 사용하면 메모리 할당을 추적하고 누수된 객체를 탐지할 수 있다.

## 주요 기능
+ 메모리 누수를 진단하는 과정은 아래의 흐름을 거친다.
	+ 의심 -> 탐지 -> 분석 -> 수정 -> 검증
#### 1. Allocations 프로파일러: 메모리 증가 탐지
- 가장 먼저 전반적인 메모리 사용량을 확인하여 누수 의심 영역을 찾는데 사용
- 예: 특정 기능 실행 후 메모리가 비정상적으로 계속 증가하고 해제되지 않는 현상 포착
- 실제 활용 (Mark Generation 기법)
	1. Xcode에서 `Product` > `Profile`을 선택하거나 `Cmd + I`를 눌러 Instruments를 실행하고 Allocations 템플릿을 선택.
	2. 앱을 안정적인 상태(A)로 만든다. (예: 메인 화면)
	3. Instruments 하단의 Mark Generation 버튼을 클릭한다. (힙 상태 스냅샷 1 생성)
	4. 메모리 누수가 의심되는 기능(B)을 실행한다. (예: 특정 화면 진입 후 빠져나오기)
	5. 다시 안정적인 상태(A)로 돌아온 후, Mark Generation 버튼을 다시 클릭한다. (힙 상태 스냅샷 2 생성)
	6. 두 스냅샷 사이의 Growth(증가량)를 분석한다. 여기서 해제되지 않고 남아있는 객체들이 바로 '버려진 메모리(Abandoned Memory)'이며, 메모리 누수의 원인일 가능성이 높다.
#### 2. Leaks 프로파일러: 직접적인 누수 탐지
- Leaks 프로파일러는 순환 참조로 인해 시스템이 접근할 수 없는, 명확한 '누수' 객체를 직접적으로 찾아준다.
- 실제 활용
    1. Instruments에서 Leaks 템플릿을 선택하고 프로파일링을 시작한다.
    2. 누수가 의심되는 기능을 반복적으로 실행하여 누수를 유발한다.
    3. 빨간색 X 아이콘과 함께 누수된 객체가 목록에 나타나면 해당 객체를 선택한다.
    4. 우측의 스택 트레이스(Stack Trace)를 통해 해당 객체가 어디서 할당되었는지 코드의 위치를 확인할 수 있다.
#### 3. 메모리 그래프 디버거: 순환 참조의 시각적 분석
- 누수 의심 객체를 찾았다면, 그 원인인 순환 참조 구조를 시각적으로 확인하는 데 가장 효과적인 도구이다.
- 실제 활용
    1. Xcode에서 앱을 실행(디버깅) 중인 상태에서 누수가 의심되는 시점에 디버그 영역의 'Debug Memory Graph' 버튼(동그라미 3개가 연결된 아이콘)을 클릭한다.
    2. 잠시 후 런타임의 모든 객체 관계가 그래프 형태로 나타난다.
    3. 좌측 네비게이터에서 보라색 느낌표(!)가 표시된 객체를 찾는다. 이는 Xcode가 순환 참조로 인해 누수되었다고 강하게 의심하는 객체이다.
    4. 해당 객체를 클릭하면 우측 인스펙터와 중앙 그래프에 어떤 객체와 강한 참조(실선 화살표)로 연결되어 순환을 이루고 있는지 명확하게 보여준다.

#### 종합 워크플로우
1. Allocations로 특정 기능 실행 시 메모리가 지속적으로 증가하는지 확인한다.
2. Leaks 또는 메모리 그래프 디버거를 사용하여 해당 메모리 증가가 실제 순환 참조로 인한 누수인지 확인한다.
3. 메모리 그래프 디버거로 순환 참조를 이루는 객체들의 관계(A → B → A)를 시각적으로 파악한다.
4. Leaks의 스택 트레이스 정보로 해당 객체들이 코드의 어느 부분에서 생성되었는지 확인한다.
5. 원인이 된 강한 참조(클로저, 델리게이트 등)를 `weak` 또는 `unowned`로 수정한다.
6. 다시 Allocations의 Mark Generation 기법을 사용해 동일한 기능을 실행했을 때 더 이상 메모리 증가가 없는지 검증한다.

## 코드 예시
#### 1. 클로저에서의 순환 참조
##### 문제 코드
`LeakyViewModel`의 `fetchData` 메서드 내 클로저가 `self`를 강하게 참조하고 있다. 만약 이 View가 데이터를 불러오는 중에 화면에서 사라지면, 비동기 작업이 끝날 때까지 ViewModel은 메모리에 남아있게 된다. 작업이 매우 길거나 끝나지 않으면 영구적인 누수가 된다.
```Swift
import SwiftUI
import Combine

// 대용량 데이터를 생성하고 보관하는 작업자 클래스
class HeavyWorker {
    private var dataChunks: [Data] = []
    let id = UUID()

    init() { print("HeavyWorker Initialized: \(id.uuidString.prefix(4))") }

    func generateDataChunk() {
        // 50MB 크기의 랜덤 데이터 덩어리를 생성
        let chunkSize = 50 * 1024 * 1024
        let data = Data(repeating: 0, count: chunkSize)
        dataChunks.append(data)
        let totalSize = dataChunks.count * 50
        print("Worker \(id.uuidString.prefix(4)) allocated new chunk. Total: \(totalSize) MB")
    }

    deinit { print("HeavyWorker Deallocated: \(id.uuidString.prefix(4))") }
}

  

// 메모리 누수를 유발하는 ViewModel
class LeakyViewModel: ObservableObject {
    @Published var totalAllocatedSizeMB: Int = 0
    private var timerCancellable: AnyCancellable?
    private let worker = HeavyWorker() // ViewModel이 Worker를 강하게 참조
    let id = UUID()
  
    init() { print("LeakyViewModel Initialized: \(id.uuidString.prefix(4))") }
  
    func startHeavyTask() {
        // 클로저가 self를 강하게 참조
        timerCancellable = Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { _ in
                self.worker.generateDataChunk()
                self.totalAllocatedSizeMB += 50
            }
    }
  
    deinit {
        print("LeakyViewModel Deallocated: \(id.uuidString.prefix(4))")
        timerCancellable?.cancel()
    }
}
  
// 상세 페이지
struct LeakyView: View {
    @StateObject private var viewModel = LeakyViewModel()
  
    var body: some View {
        VStack {
            Text("Leaky Version")
            Text("Allocated: \(viewModel.totalAllocatedSizeMB) MB")
                .font(.largeTitle)
                .fontWeight(.bold)
        }
        .onAppear(perform: viewModel.startHeavyTask)
    }
}
```
##### 해결 코드
캡처 리스트에 `[weak self]`를 추가하여 클로저가 ViewModel을 약하게 참조하도록 수정했다. 이렇게 하면 View가 사라졌을 때 ViewModel이 즉시 메모리에서 정상적으로 해제될 수 있다.
```Swift
import SwiftUI
import Combine
  
// HeavyWorker 클래스는 동일하게 사용합니다.
  
// 메모리 누수를 해결한 ViewModel
class FixedViewModel: ObservableObject {
    @Published var totalAllocatedSizeMB: Int = 0
    private var timerCancellable: AnyCancellable?
    private let worker = HeavyWorker()
    let id = UUID()
  
    init() { print("FixedViewModel Initialized: \(id.uuidString.prefix(4))") }
  
    func startHeavyTask() {
        // [weak self]로 순환 참조를 해결
        timerCancellable = Timer.publish(every: 1, on: .main, in: .common)
            .autoconnect()
            .sink { [weak self] _ in
                guard let self = self else { return }
                self.worker.generateDataChunk()
                self.totalAllocatedSizeMB += 50
            }
    }
  
    deinit {
        print("FixedViewModel Deallocated: \(id.uuidString.prefix(4))")
        timerCancellable?.cancel()
    }
}
  
// 해결된 버전의 상세 페이지
struct FixedView: View {
    @StateObject private var viewModel = FixedViewModel()
  
    var body: some View {
        VStack {
            Text("Fixed Version")
            Text("Allocated: \(viewModel.totalAllocatedSizeMB) MB")
                .font(.largeTitle)
                .fontWeight(.bold)
        }
        .onAppear(perform: viewModel.startHeavyTask)
    }
}
```


## Keywords
+ [[ARC (Automatic Reference Counting)]]
+ [[메모리 누수]]
- [[Instruments]]

## References
- Apple Developer Documentation: [Gathering Information About Memory Use](https://developer.apple.com/documentation/xcode/gathering-information-about-memory-use)
- Hacking with Swift: [How to find and fix memory leaks in Swift using Instruments](https://www.hackingwithswift.com/articles/148/how-to-find-and-fix-memory-leaks-in-swift-using-instruments)
## 작성자
- #Air 