>[!question]
>GQ. 데이터 모델링 단계에서 발생할 수 있는 성능 병목 현상(예: 조인 비용, 인덱스 선택 오류 등)을 어떻게 예측하고 완화할 수 있는가?

## Description
- 데이터 모델링은 성능에 직결되는 핵심요소이다. 관계 설정, 인덱싱, 데이터 접근 방식에 따라 병목 현상이 발생할 수 있다.

## 주요 기능
#### 1. 조인 비용 최소화
- 관계형 모델에서 Many-To-Many 혹은 깊은 관계 트리 접근 시 N+1 문제 방지
	- N+1 문제
		- 데이터베이스에서 자주 발생하는 비효율적인 쿼리 패턴
		- 하나의 쿼리를 실행한 추가적으로 N개의 쿼리가 각각 실행되어 총 N+1개의 쿼리가 발생하는 상황
- 프리페치(`relationshipKeyPathsForPrefetching`)를 활용해 lazy loading 최소화
#### 2. 인덱스 기반 성능 최적화
- predicate 쿼리 대상 필드에 인덱스를 설정하여 검색 시간 단축
- CoreData에서 인덱스 지정
#### 3. 배치 Fetch와 메모리 관리
- `fetchBatchSize`, `returnsObjectsAsFaults` 등 설정으로 메모리 사용 최적화
- 대용량 데이터 처리 시 부분 로딩 적용
#### 4. 쓰기 충돌 방지
- Background context를 활용한 비동기 저장
- 메인 쓰레드의 UI 차단 없이 안정적인 데이터 처리 가능
#### 5. 모델 구조 단순화
- 트리 형태 대신 플랫 구조 + 참조 ID 방식 고려
- 불필요한 관계 제거로 데이터 접근 간소화


## 코드 예시
#### 1. 조인 비용 최소화 (관계 데이터 프리페치)
- `Author`와 `Book`은 1:N 관계
- 모든 저자와 각 저자의 책 목록을 출력하고 싶음
```Swift
let authors = try context.fetch(Author.fetchRequest())  // 1개의 쿼리

for author in authors {
	let books = author.books  // 각 author마다 추가 쿼리 발생 (N개의 쿼리)
	print("\(author.name): \(books.count) books")
}
```
- 쿼리 횟수가 많아지면서 네트워크 지연, 디스크 I/O, 메모리 사용량 증가
- 특히 데이터 수가 많을수록 성능 저하가 지수적으로 증가
```Swift
let request = NSFetchRequest<Author>(entityName: "Author")
request.relationshipKeyPathsForPrefetching = ["books"]
let authors = try context.fetch(request)  // 저자 + 책을 한 번에 가져옴
```
- `relationshipKeyPathsForPrefetching`: 연관된 `books` 데이터를 미리 로드 -> N+1 문제 방지, 관계 접근 시 추가 fetch 요청 최소화

-> 대규모 관계 데이터를 효율적으로 읽고 표시할 수 있다.

#### 2. 인덱스 기반 성능 최적화
```Swift
let request = NSFetchRequest<Employee>(entityName: "Employee")
request.predicate = NSPredicate(format: "email CONTAINS[c] %@", "@example.com")
request.fetchLimit = 20

do {
    let result = try context.fetch(request)
} catch {
    print("Indexed fetch failed: \(error)")
}
```
- Core Data 설정과 연관. 모델 설정에서 email 속성을 Indexed로 설정했다면
	- 인덱스를 통해 쿼리 속도 대폭 향상

-> 자주 검색되는 필드에 인덱스를 설정해 쿼리 속도를 개선할 수 있다.

#### 3. 배치 Fetch와 메모리 관리
```Swift
request.fetchBatchSize = 50}
```
- `fetchBatchSize`: 한 번에 메모리에 로드할 갯수 지정 -> 메모리 절약, 성능 향상
```Swift
let request = NSFetchRequest<Employee>(entityName: "Employee")
request.returnsObjectsAsFaults = true

do {
    let results = try context.fetch(request)
    // 이 시점에서 results는 fault 상태 객체들로 구성되어 있음
    print(results[0]) // 객체는 있지만, 속성은 아직 메모리에 없음
    print(results[0].name) // 이 순간 실제 속성이 메모리에 로드됨 (fault 해제)
} catch {
    print("Fetch failed: \(error)")
}
```
- `returnsObjectsAsFaults = true`: Core Data가 엔티티를 "fault" 상태로 반환
	- 실제 속성에 접근하기 전까지 메모리에 올리지 않음 -> 메모리 사용량 감소
```Swift
request.includesPropertyValues = false // 값 불러오지 않음 (ID만 사용 시)
```
- `includesPropertyValues = false`: 실제 속성값을 로드하지 않음 (ID만 필요할 때 유용)

->
- 대량의 데이터를 fetch할 때 메모리 사용량을 크게 줄일 수 있다.
- 예를 들어 ID만 필요한 목록에서 세부 정보 없이 표시하거나 삭제할 때 유용하다.

#### 4. 쓰기 충돌 방지 (Background Context를 통한 비동기 저장)
```Swift
persistentContainer.performBackgroundTask { backgroundContext in
    let newEmployee = Employee(context: backgroundContext)
    newEmployee.name = "Air"
    newEmployee.email = "air@example.com"

    do {
        try backgroundContext.save()
        print("Saved in background")
    } catch {
        print("Background save error: \(error)")
    }
}
```

-> 
- UI 응답성을 유지하면서 데이터 저장 가능
- 다수의 저장 작업을 비동기로 분리해 충돌 위험 감소
- 멀티 스레딩 환경에서도 안정적으로 Core Data 작업 수행

## Keywords
- [[데이터 모델링 (Data Modeling)]]
- [[Core Data]]

## 작성자
- #Air