>[!question]
>GQ. SwiftUI 앱에서 여러 데이터 소스를 효율적으로 관리하기 위해 Repository 패턴을 어떻게 적용할 수 있는가?

## Description
- 리포지토리 패턴(Repository Pattern)은 애플리케이션의 비즈니스 로직(ViewModel)과 데이터 소스(원격 API, 로컬 캐시, 데이터베이스 등) 사이에 위치하는 중개자 역할을 한다. 이 패턴의 핵심 목표는 데이터 접근 로직을 추상화하여, ViewModel이 데이터가 어디서 오는지 또는 어떻게 저장되는지에 대해 전혀 알 필요가 없도록 만드는 것이다.
- 특히 원격 API와 로컬 캐시처럼 여러 데이터 소스를 사용하는 SwiftUI 앱에서 유용하다. 리포지토리가 어떤 소스에서 데이터를 가져올지 결정하는 중앙 집중식 관리 지점 역할을 하므로, ViewModel은 단일 인터페이스(프로토콜)를 통해 데이터를 요청하기만 하면 된다. 이로 인해 코드의 관심사가 명확히 분리되어 테스트 용이성과 유지보수성이 크게 향상된다.

## 주요 기능
+ SwiftUI 앱에서 리포지토리 패턴을 활용해 여러 데이터 소스를 관리하는 핵심 기능은 **데이터 흐름을 조율**하는 것이다. 일반적인 아키텍처는 **View → ViewModel → Repository → Data Sources** 순으로 데이터가 흐른다.
1. **단일 진입점 제공**: ViewModel은 `MovieRepository`라는 단일 프로토콜(인터페이스)에만 의존다. 이 프로토콜을 구현한 `MovieRepositoryImpl` 클래스가 내부적으로 원격 API와 로컬 캐시 데이터 소스를 모두 관리한다.
    
2. **캐싱 전략 구현**: 리포지토리는 데이터 요청 시 어떤 소스를 우선할지 결정하는 로직을 포함한다. 가장 일반적인 전략은 다음과 같다 :
    - **데이터 읽기 (Read)**: 먼저 로컬 캐시에서 데이터를 찾아 반환한다. 만약 캐시에 데이터가 없으면(Cache Miss), 원격 API에 데이터를 요청힌다. API로부터 받은 데이터는 로컬 캐시에 저장한 후 ViewModel에 반환하여 다음 요청 시 빠르게 응답할 수 있도록 한다.
        
    - **데이터 쓰기 (Write)**: 새로운 데이터를 추가하거나 수정할 때, 원격 API와 로컬 캐시 양쪽에 동시에 데이터를 써서 일관성을 유지한다 (Write-Through 전략).
        
3. **테스트 용이성 확보**: ViewModel이 구체적인 구현이 아닌 프로토콜에 의존하므로, 단위 테스트 시 실제 네트워크나 데이터베이스에 연결된 리포지토리 대신 가짜 데이터를 반환하는 모의(Mock) 리포지토리를 쉽게 주입할 수 있다.
    

이 모든 과정은 의존성 주입(Dependency Injection)을 통해 이루어진다. ViewModel을 생성할 때 외부에서 리포지토리 구현체를 주입함으로써, ViewModel과 데이터 계층 간의 결합도를 낮추고 유연한 구조를 만든다.

## 코드 예시
+ 영화 목록을 원격 API와 로컬 캐시에서 가져오는 간단한 SwiftUI 앱 예시이다.

**1단계: 모델 및 프로토콜 정의**
```Swift
import Foundation

// 1. Domain Model: 앱의 비즈니스 로직에서 사용할 순수 데이터 모델
struct Movie: Codable, Identifiable, Equatable {
    let id: Int
    let title: String
    let releaseDate: String
}

// 2. Repository Protocol: ViewModel이 의존할 인터페이스
protocol MovieRepository {
    func fetchMovies() async throws -> [Movie]
}

// 3. Data Source Protocols: 각 데이터 소스의 역할을 정의
protocol MovieAPIDataSource {
    func fetchMovies() async throws -> [Movie]
}

protocol MovieCacheDataSource {
    func getMovies() -> [Movie]?
    func saveMovies(_ movies: [Movie])
}
```

**2단계: Repository 구현체 작성** 이 클래스는 API와 캐시 데이터 소스를 받아 데이터 요청을 조율하는 핵심 로직을 담당한다.
```Swift
// 4. Concrete Repository: 여러 데이터 소스를 관리하는 구현체
class MovieRepositoryImpl: MovieRepository {
    private let apiDataSource: MovieAPIDataSource
    private let cacheDataSource: MovieCacheDataSource
    
    init(apiDataSource: MovieAPIDataSource, cacheDataSource: MovieCacheDataSource) {
        self.apiDataSource = apiDataSource
        self.cacheDataSource = cacheDataSource
    }
    
    func fetchMovies() async throws -> [Movie] {
        // 먼저 캐시에서 데이터를 가져오려고 시도
        if let cachedMovies = cacheDataSource.getMovies() {
            print("데이터를 캐시에서 가져옵니다.")
            return cachedMovies
        }
        
        // 캐시에 데이터가 없으면 API에서 가져옴
        print("데이터를 API에서 가져옵니다.")
        let apiMovies = try await apiDataSource.fetchMovies()
        
        // API에서 가져온 데이터를 캐시에 저장
        cacheDataSource.saveMovies(apiMovies)
        
        return apiMovies
    }
}
```

**3단계: ViewModel 및 View 통합** ViewModel은 리포지토리 프로토콜을 통해 데이터를 요청하고, View는 ViewModel의 상태를 표시한다.
```Swift
import SwiftUI

// 5. ViewModel: UI의 상태와 로직을 관리
@MainActor
@Observable
class MovieViewModel {
    var movies: [Movie] =
    var errorMessage: String?
    
    private let repository: MovieRepository
    
    // 의존성 주입: 외부에서 Repository 구현체를 전달받음
    init(repository: MovieRepository) {
        self.repository = repository
    }
    
    func loadMovies() async {
        do {
            movies = try await repository.fetchMovies()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// 6. View: ViewModel의 데이터를 표시
struct MovieListView: View {
    @State private var viewModel: MovieViewModel
    
    init(repository: MovieRepository) {
        _viewModel = State(initialValue: MovieViewModel(repository: repository))
    }
    
    var body: some View {
        NavigationStack {
            List(viewModel.movies) { movie in
                VStack(alignment:.leading) {
                    Text(movie.title).font(.headline)
                    Text(movie.releaseDate).font(.subheadline)
                }
            }
           .navigationTitle("인기 영화")
           .task {
                await viewModel.loadMovies()
            }
           .alert(isPresented:.constant(viewModel.errorMessage!= nil)) {
                Alert(title: Text("오류"), message: Text(viewModel.errorMessage?? ""), dismissButton:.default(Text("확인")))
            }
        }
    }
}
```

## Keywords
+ [[리포지토리 패턴 (Repository Pattern)]]
+ [[디자인 패턴 (Design Pattern)]]

## References
- [Repository design pattern in Swift explained using code examples](https://www.avanderlee.com/swift/repository-design-pattern/)
- [Mastering the Repository Pattern in iOS Development](https://medium.com/@gauravharkhani01/mastering-the-repository-pattern-in-ios-development-f6fe92698873)
- [Implementing the Repository Pattern in Swift](https://medium.com/@asrafulalam2010502cse/implementing-the-repository-pattern-in-swift-32b1534691a4)

## 작성자
- #Air 