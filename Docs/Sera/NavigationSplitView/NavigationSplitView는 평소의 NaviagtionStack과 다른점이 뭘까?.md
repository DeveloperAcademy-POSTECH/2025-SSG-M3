>[!question]
>GQ1. NavigationSplitView는 평소의 NaviagtionStack과 다른점이 뭘까?

사이드바를 구현하고 싶었는데 -> navigationsplitview를 쓰면 자동으로 구현된다는 사실을 알고.. ^^ 한번 파보고 싶었습니다.
## Description
- `NavigationSplitView`는 여러 개의 열(column)을 갖는 내비게이션 인터페이스이다. 넓은 화면에서 여러 정보를 동시에 보여주고 싶을 때 사용함


구성 요소
`NavigationSplitView`는 최대 3개의 주요 부분으로 구성됨
1. 사이드바 : 가장 왼쪽에 위치하는 최상위 탐색 레벨. 주로 앱의 주요 카테고리나 메일함 목록처럼 큰 분류를 표시함
2. 콘텐츠 : (선택 사항) 중간 열. 사이드바에서 특정 항목을 선택했을 때 그에 해당하는 중간 단계의 목록을 보여줌
3. 상세 뷰 : 가장 오른쪽에 위치하며, 최종 콘텐츠를 표시하는 뷰입니다. 사이드바나 콘텐츠 뷰에서 특정 항목을 선택했을 때 그 상세 내용을 보여줌


| 구분     | NavigationSplitView      | NavigationStack       | NavigationView                     |
| ------ | ------------------------ | --------------------- | ---------------------------------- |
| 핵심 목적  | 다중 열(2-3개) 레이아웃          | 단일 열, 스택 기반 내비게이션     | 구 버전의 스택 기반 내비게이션                  |
| 주요 사용처 | iPad, Mac, 넓은 화면의 iPhone | iPhone 및 모든 기기의 단일 경로 | iOS 16 이전 (현재는 NavigationStack 권장) |
| 구조     | 사이드바, 콘텐츠, 상세 뷰 등        | 루트 뷰에서 다음 뷰로 쌓아 올림    | 스택과 유사하나 API가 덜 직관적                |
| 동작 방식  | 각 열이 독립적으로 뷰를 표시         | PUSH / POP 방식         | PUSH / POP 방식                      |



### 2단 구성(Sidebar - Detail)
```swift
struct ContentView: View {
    @State private var selectedCategory: Category?

    var body: some View {
        NavigationSplitView {
            // Sidebar: 카테고리 목록
            List(Category.allCases, selection: $selectedCategory) { category in
                Text(category.name)
            }
            .navigationTitle("카테고리")
        } detail: {
            // Detail: 선택된 카테고리의 상세 내용
            if let category = selectedCategory {
                Text("\(category.name)의 상세 정보")
            } else {
                Text("카테고리를 선택하세요.")
            }
        }
    }
}
```

### 3단 구성(Sidebar - Content - Detail)
```swift
struct MailView: View {
    @State private var selectedMailbox: Mailbox?
    @State private var selectedMessage: Message?

    var body: some View {
        NavigationSplitView {
            // Sidebar: 메일함 목록
            List(mailboxes, selection: $selectedMailbox) { mailbox in
                Text(mailbox.name)
            }
            .navigationTitle("메일함")
        } content: {
            // Content: 선택된 메일함의 메시지 목록
            if let mailbox = selectedMailbox {
                List(mailbox.messages, selection: $selectedMessage) { message in
                    Text(message.subject)
                }
            } else {
                Text("메일함을 선택하세요.")
            }
        } detail: {
            // Detail: 선택된 메시지의 본문
            if let message = selectedMessage {
                Text(message.body)
            } else {
                Text("메시지를 선택하세요.")
            }
        }
    }
}
```
![[Pasted image 20250713141924.png]]


## 주요 기능
+ 실제 활용을 작성

## 코드 예시
+ 실제 코드 예시를 작성

## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- #Sera 