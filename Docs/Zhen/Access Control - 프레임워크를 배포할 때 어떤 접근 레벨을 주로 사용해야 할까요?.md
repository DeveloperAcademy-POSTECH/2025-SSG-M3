
>[!question]
>GQ1. 프레임워크를 배포할 때 어떤 접근 레벨을 주로 사용해야 할까요? 
>	Swift에서 기본값으로 internal을 채택한 설계 의도를 논의해보세요.
>GQ2. 프레임워크(A 모듈)에서 공개 API 타입을 상속받은 커스텀 뷰를 만들 때, 오버라이드 가능한 메서드를 정의하려면 어떤 접근 레벨을 사용해야 하나요?

## GQ 1
+ - - **프레임워크 배포 시**
        - **public**: 외부에서 인스턴스 생성·메서드 호출을 허용할 API
        - **open**: 추가로 상속·오버라이드를 허용할 확장 지점
        - 내부 구현은 **internal** (기본값), 혹은 더 좁은 fileprivate/private로 감춥니다.
            
        
    - Swift가 기본값으로 **internal**을 택한 이유
        1. **캡슐화 강화**: 외부로 불필요한 구현 세부사항이 유출되지 않도록
        2. **명시적 API 설계**: 개발자가 의도한 공개 지점만 public/open으로 표시
        3. **버전 호환성 관리**: internal 멤버 변경은 외부 사용자에 영향 없음

## 코드 예시

```swift

import SwiftUI

// 1. 내부 구현을 숨기는 internal
// 같은 모듈 내에서만 사용할 수 있어서 외부에 노출되지 않음
internal struct Helper {
    static func fetchData() -> String {
        // 복잡한 네트워크/DB 로직 대신 간단히 반환
        // 여기가 internal 이므로, 여기서 나중에 update 를 해도 외부 호환성 문제가 생기지 않음. 
        "Hello from internal Helper"
    }
}

// 2. 외부에 공개하는 public API
// public으로 선언된 View와 init()만 앱 모듈에서 보이고 사용할 수 있음. = 명시적
public struct PublicDataView: View {
    let content: String

    public init() {
        // internal Helper를 이용해 데이터 로드 = 캡슐화 
        content = Helper.fetchData()
    }

    public var body: some View {
        Text(content)
            .padding()
    }
}

// 3. 상속·오버라이드 지점으로 열어두는 open
// SwiftUI View 자체는 struct여서 open을 사용할 수 없으니,
// UIHostingController를 subclass해 확장 지점을 만듦.
open class BaseHostingController<Content: View>: UIHostingController<Content> {
    public init(root: Content) {
        super.init(rootView: root)
        configureAppearance()
    }
    @objc required public init?(coder: NSCoder) {
        super.init(coder: coder)
        configureAppearance()
    }

    // open 으로 선언한 메서드는 앱 모듈에서 override 가능
    open func configureAppearance() {
        view.backgroundColor = .systemBackground
    }
}

// 4. 앱 모듈에서 override 예시
class CustomHostingController: BaseHostingController<PublicDataView> {
    // open 메서드를 서브클래스에서 오버라이드 할 수 있음. 
    override func configureAppearance() {
        super.configureAppearance()
        view.backgroundColor = .yellow
    }
}
```


## GQ 2

| **접근 레벨**           | **모듈 외부 사용** | **상속 허용** | **오버라이드 허용** | **기본값 여부** |
| ------------------- | ------------ | --------- | ------------ | ---------- |
| open                | ✅            | ✅         | ✅            | 아니요        |
| public              | ✅            | ❌         | ❌            | 아니요        |
| internal            | ❌            | ❌         | ❌            | ✅          |
| fileprivate/private | ❌            | ❌         | ❌            | 아니요        |
|                     |              |           |              |            |
- open 또는 public 으로 된 메서드만 모듈 외부에서 상속 혹은 오버라이드가 가능함. 
- open 은 상속과 오버라이드 모두 허용. 
- public은 모듈 외부에서 사용이 가능. 
	- 따라서, 커스텀 뷰에서 오버라이드 가능한 메서드를 정의하려면 open으로 선언해야 함. 




## Keywords
+ [[Access Control - 프레임워크를 배포할 때 어떤 접근 레벨을 주로 사용해야 할까요?]]
+ [[접근 제어 (Access Control)]]

## 작성자
- #Zhen 