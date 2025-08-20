
>[!question]
>GQ1. 어떤 디자인 패턴이 SwiftUI 와 잘 어울릴까? 그리고 어떤 패턴이 어울리지 않을까? 


## UIKit 방식 
- 명령형(Imperactive)+이벤트 중심 : 즉, 무슨 일이 일어나면 화면을 어떻게 바꿀지를 개발자가 다 명령하는 방식 ! 
- View 참조 : 뷰 객체(버튼 같은)를 변수로 잡고(`IBOutlet`) 코드에서 접근해서 속성을 바꿈. 
- 이벤트 처리 : 버튼을 누르거나, 데이터가 변하는 등의 이벤트가 생기면? 개발자가 `delegate`, `target-action`, `callback` 등으로 직접 처리해서 UI를 업데이트 해야 함. 

즉 ! 내가 이벤트를 감지해서 ~ 그거에 맞게 UI를 직접 조작하는 방식 ! 

## SwiftUI 방식 
- 선언형(Declarative)+상태(State) : UI를 어떻게 보이게 할지만 선언해 두고, State 가 바뀌면 화면이 <u>자동으로</u> 갱신
- View 참조 불가 : 특정 뷰를 찾아서 직접 속성을 바꾸지는 못함. 
	- 대신 바인딩된 상태값(`State`, `Binding` 등)을 바꾸면 SwiftUI가 알아서 화면을 다시 그려줌. 
- 이벤트 처리: 스유에서는 `closure`와 `binding` 으로 단순하게 처리

즉 ! 상태를 바꾸면, 그 상태를 참조하는 view 가 자동으로 바뀌는 방식 ! (키오스크랑 비.. )

## 구조적 차이 
- UIKit 의 UIView 는 class 기반. 상속해서 객체를 직접 생성 , 관리 -> 무거움
- SwiftUI의 View 는 struct 기반. body 라는 <u>함수</u>로 UI를 선언 -> 가볍고 빠름
	- 즉 ! SwiftUI의 View는 함수라는 것임 ! 
	- 즉 ! 입력값(state)를 바꾸면 출력(UI)이 자동으로 달라짐 ! 

```Swift
struct ContentView: View {
    @State private var isOn = false

    var body: some View {
        VStack {
            Button("Toggle") {
                isOn.toggle()
            }
            // 이런식으로 미리 선언된 body 안에서 if 문이나 상태값에 따라 보이거나 ! 사라지거나 ~ 알아서 판단해서 처리해준다는 뜻 
            if isOn {
                Text("켜짐")
            } else {
                Text("꺼짐")
            }
        }
    }
}
```

## 실제 패턴은 어떻게 달라져야 하는가? 
### Coordinator 
- UIKit 에서는, 일일히 이벤트를 제어해야 했기 때문에 Coordinator가 필요했음. (손님이 화장실 어디로 가요? 하면 점원(Coordinator)이 어디로 가세요 ~ 하고 안내해야 햇음)
- SwiftUI에서는 상태값만 바꾸면 자동으로 화면이 이동함. 점원이 필요 없어진 것. 
## SwiftUI에서의 클린 아키텍쳐는? 
- 결국 필요한게 무엇인가. 를 생각해보면 Presentation / Business Logic / Data Access 밖에 없음. 
	- Presentation (메뉴판, 어떻게 보여줄 것인가) : View
	- Business Logic (주문처리, 어떤 순서로 어떻게 처리할 것인가) : ViewModel, Interactor 
	- Data Access (창고, 어디서 가져올 것인가) : Repository 

		- 이렇게 구분하면, 주방(Repository)를 바꿔도 메뉴판(View)는 영향이 없게 됨. 
		- 점원(ViewModel)이 있기 때문에 손님(View)은 복잡한 과정을 몰라도 됨. 
		- 즉, 테스트하기 쉽고 유지보수가 편한 구조라는 뜻 
## Interactor, Repository Pattern 
### Interactor Pattern 
- 역할 : 특정 화면(View)에서 필요한 비지니스 로직을 담당하는 중간 관리자, 비지니스 로직 캡슐화 객체
- 특징 
	- 자기 스스로 상태(`State`)를 가지지 않음 -> 대신 `AppState` (전역 상태 참고)를 참조해서 업데이트함. 
	- View 와 직접 얽히지 않고, 결과를 `AppState`나 특정 `Binding`에 넣어줌. 
	- 프로토콜로 추상화됨. 그래서 나중에 Interactor 대신 Mock Interactor 를 넣어서 테스트하기 쉬움. 
- 즉 ! 손님(View)이 뭔가 달라고 하면, 점원(Interactor)은 주방(Repository)에 주문 넣고, 음식이 나오면 메뉴판(AppState)에 업데이트 해주는 것.
```Swift
// MARK: - Domain
struct Todo: Identifiable, Hashable, Equatable {
    let id = UUID()
    var title: String
    var done: Bool = false
}
// 로딩 상태 표현 (전역/로컬 공용)
enum Loadable<Value: Equatable>: Equatable {
    case notRequested
    case isLoading(last: Value?)
    case loaded(Value)
    case failed(String)

    var value: Value? {
        if case let .loaded(v) = self { return v }
        return nil
    }
}
```
 
```Swift
// MARK: - AppState (전역 상태)
@MainActor
final class AppState: ObservableObject {
    @Published var todos: Loadable<[Todo]> = .notRequested
    @Published var isBusy: Bool = false
}

```

```Swift
// MARK: - Interactor (Stateless 비즈니스 로직)
protocol TodosInteractor {
    func loadTodos()
    func addTodo(title: String)
    func toggle(_ todo: Todo)
    func remove(_ indexSet: IndexSet)
    func loadDetail(_ detail: Binding<Loadable<String>>, id: Todo.ID)
} 

@MainActor
struct RealTodosInteractor: TodosInteractor {
    let appState: AppState
  
    func loadTodos() {
        appState.todos = .isLoading(last: appState.todos.value)
        Task {
            // 비동기 흐름 흉내
            try? await Task.sleep(nanoseconds: 300_000_000)
            appState.todos = .loaded([
                Todo(title: "SwiftUI 공부"),
                Todo(title: "10분 산책"),
                Todo(title: "물 마시기")
            ])
        }
    }
  
    func addTodo(title: String) {
        guard !title.trimmingCharacters(in: .whitespaces).isEmpty else { return }
        switch appState.todos {
        case .loaded(var items):
            items.insert(Todo(title: title), at: 0)
            appState.todos = .loaded(items)
        case .notRequested:
            appState.todos = .loaded([Todo(title: title)])
        case .isLoading(let last):
            var items = last ?? []
            items.insert(Todo(title: title), at: 0)
            appState.todos = .isLoading(last: items)
        case .failed:
            appState.todos = .loaded([Todo(title: title)])
        }
    }

    func toggle(_ todo: Todo) {
        guard case .loaded(var items) = appState.todos else { return }
        if let idx = items.firstIndex(of: todo) {
            items[idx].done.toggle()
            appState.todos = .loaded(items)
        }
    }

    func remove(_ indexSet: IndexSet) {
        guard case .loaded(var items) = appState.todos else { return }
        items.remove(atOffsets: indexSet)
        appState.todos = .loaded(items)
    }
  

    func loadDetail(_ detail: Binding<Loadable<String>>, id: Todo.ID) {
        detail.wrappedValue = .isLoading(last: detail.wrappedValue.value)
        Task {
            try? await Task.sleep(nanoseconds: 250_000_000)
            detail.wrappedValue = .loaded("이 Todo(\(id.uuidString.prefix(6)))는 Interactor가 로드했음.")
        }
    }
}
```

```Swift
// MARK: - Views
struct TodosListView: View {
    @EnvironmentObject var appState: AppState
    let interactor: TodosInteractor
    @State private var newTitle: String = ""
  
    var body: some View {
        NavigationStack {
            content
                .navigationTitle("Todos")
                .toolbar {
                    ToolbarItem(placement: .topBarTrailing) {
                        Button("초기 로드") { interactor.loadTodos() }
                    }
                }
                .safeAreaInset(edge: .bottom) {
                    HStack {
                        TextField("할 일을 입력…", text: $newTitle)
                            .textFieldStyle(.roundedBorder)
                        Button("추가") {
                            interactor.addTodo(title: newTitle)
                            newTitle = ""
                        }
                        .buttonStyle(.borderedProminent)
                    }
                    .padding()
                    .background(.ultraThinMaterial)
                }
                .task {
                    if case .notRequested = appState.todos {
                        interactor.loadTodos()
                    }
                }
        }
    }
  
    @ViewBuilder
    private var content: some View {
        switch appState.todos {
        case .notRequested, .isLoading:
            VStack(spacing: 12) {
                ProgressView()
                Text("Loading…").foregroundStyle(.secondary)
            }
            .frame(maxWidth: .infinity, maxHeight: .infinity)
        case let .failed(msg):
            VStack(spacing: 12) {
                Text("에러: \(msg)").foregroundStyle(.red)
                Button("다시 시도") { interactor.loadTodos() }
            }
            .frame(maxWidth: .infinity, maxHeight: .infinity)
        case let .loaded(items):
            if items.isEmpty {
                ContentUnavailableView("할 일이 비어 있음", systemImage: "square.dashed.inset.filled")
            } else {
                List {
                    ForEach(items) { todo in
                        NavigationLink {
                            TodoDetailView(todo: todo, interactor: interactor)
                        } label: {
                            HStack {
                                Image(systemName: todo.done ? "checkmark.circle.fill" : "circle")
                                Text(todo.title)
                                    .strikethrough(todo.done)
                            }
                        }
                        .swipeActions {
                            Button(role: .destructive) {
                                if let idx = items.firstIndex(of: todo) {
                                    interactor.remove(IndexSet(integer: idx)
                                }
                            } label: { Label("삭제", systemImage: "trash") }
                        }
                        .contextMenu {
                            Button(todo.done ? "미완료로 표시" : "완료로 표시") {
                                interactor.toggle(todo)
                            }
                        }
                    }
                }
            }
        }
    }
}

struct TodoDetailView: View {
    let todo: Todo
    let interactor: TodosInteractor

    @State private var detail: Loadable<String> = .notRequested

    var body: some View {
        content
            .navigationTitle(todo.title)
            .task {
                if case .notRequested = detail {
                    interactor.loadDetail($detail, id: todo.id) // 로컬 Binding만 갱신함
                }
            }
            .padding()
    }  

    @ViewBuilder
    private var content: some View {
        switch detail {
        case .notRequested, .isLoading:
            HStack(spacing: 12) {
                ProgressView()
                Text("상세 정보를 불러오는 중…")
            }
        case let .failed(msg):
            VStack(spacing: 12) {
                Text("에러: \(msg)").foregroundStyle(.red)
                Button("다시 시도") { interactor.loadDetail($detail, id: todo.id) }
            }
        case let .loaded(text):
            VStack(alignment: .leading, spacing: 12) {
                Text("상세")
                    .font(.headline)
                Text(text)
                    .foregroundStyle(.secondary)

            }
        }
    }
}

// MARK: - App Entry (Preview 포함)
@main
struct InteractorMinimalApp: App {
    @StateObject private var appState = AppState()

    var body: some Scene {
        WindowGroup {
            let interactor = RealTodosInteractor(appState: appState)
            TodosListView(interactor: interactor)
                .environmentObject(appState)
        }
    }
}
```

### Repository Pattern 
- 실제 데이터 소스에 접근해서 읽고 씀 
- [[Design Pattern - SwiftUI 앱에서 여러 데이터 소스를 효율적으로 관리하기 위해 Repository 패턴을 어떻게 적용할 수 있는가?]] 참고... 
- 즉 ! 재료(Data)를 repository(창고)에서 가져옴. 그니까 창고는 재료만 제공하고 음식을 어떻게 만들지~ 는 분리해서 모름. 

### Interactor + Repository 
1. View -> Interactor 데이터 가져와줘 요청
2. Interactor -> Repository 실제 데이터 요청 
3. Repository -> 외부(서버, DB)에서 데이터 가져옴
4. Interactor -> 가져온 결과를 AppState나 Binding 에 저장
5. View -> 상태 변화를 감지하고 자동으로 UI 업데이트 

```Swift
// 📦 Repository (데이터 공급자)
protocol CountryRepository {
    func fetchCountries() -> AnyPublisher<[Country], Error>
}

struct RealCountryRepository: CountryRepository {
    let session: URLSession
    func fetchCountries() -> AnyPublisher<[Country], Error> {
        // 실제 서버에서 나라 목록 가져오기
        session.dataTaskPublisher(for: URL(string: "https://example.com/countries")!)
            .map { data, _ in try! JSONDecoder().decode([Country].self, from: data) }
            .eraseToAnyPublisher()
    }
}

// 🧑‍🍳 Interactor (중간 관리자)
protocol CountryInteractor {
    func loadCountries()
}

struct RealCountryInteractor: CountryInteractor {
    let repository: CountryRepository
    let appState: AppState
    
    func loadCountries() {
        appState.isLoading = true
        _ = repository.fetchCountries()
            .sink(receiveCompletion: { _ in },
                  receiveValue: { countries in
                      appState.countries = countries
                      appState.isLoading = false
                  })
    }
}

// 📊 AppState (전역 상태)
class AppState: ObservableObject {
    @Published var countries: [Country] = []
    @Published var isLoading: Bool = false
}

// 👀 View (메뉴판)
struct CountryListView: View {
    @EnvironmentObject var appState: AppState
    let interactor: CountryInteractor
    
    var body: some View {
        VStack {
            if appState.isLoading {
                ProgressView()
            } else {
                List(appState.countries, id: \.name) { country in
                    Text(country.name)
                }
            }
        }
        .onAppear { interactor.loadCountries() }
    }
}
```


## 더 생각해보기 
- **SwiftUI에서 MVVM이 꼭 필요하지 않다는 생각은 ?** 
	- 맞말. 소규모일수록, 단순 화면일수록 굳이 MVVM이 필요 없을 것 같음. 뷰의 State/Binding 자체만으로 동작이 훨씬 직관적. 그리고 이미 분리된 형태. 형식적인 뷰모델은 보일러플레이트만 늘릴 수 있음. 
	- 그러나 규모가 커질수록 적어도 로직을 뷰에서 분리할 필요는 있어 보임... 꼭 MVVM이 아니더라도. Interactor 도 괜찮고. 아무튼 역할은 분리되어야 하지 않을까? 
- **그럼 언제가 기준이 될 수 있을까?** 
	- 상태가 전역 공유가 될 필요가 없을 때 (즉, 화면이 지역적이고, 규칙, 방향성이 단순할 경우)
	- 비동기가 한 두 군데일 경우 
		- 이럴 때는 MVVM이 필요 없지 않을까? 

## Keywords
+ [[Interactor]]
+ [[Coordinator]]
+ [[디자인 패턴 (Design Pattern)]]
+ [[리포지토리 패턴 (Repository Pattern)]]


## References
- https://gon125.github.io/posts/SwiftUI를-위한-클린-아키텍처/

## 작성자
- #Zhen 