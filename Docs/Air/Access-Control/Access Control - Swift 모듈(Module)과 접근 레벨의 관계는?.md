>[!question]
>GQ. Swift 모듈(Module)과 접근 레벨의 관계는?

## Description
- Swift에서 모듈(Module)은 컴파일되고 배포되는 코드의 단일 단위를 의미한다. 이는 하나의 프레임워크나 애플리케이션 타겟에 해당하며, 자신만의 독립된 이름 공간(Namespace)을 가진다. `import` 키워드를 통해 다른 모듈의 코드를 가져와 사용할 수 있다.
- 접근 제어(Access Control)는 모듈의 경계를 기반으로 코드의 가시성을 제어하는 메커니즘이다. 이를 통해 내부 구현은 숨기고 외부에 제공할 API만을 선택적으로 노출할 수 있다. Swift는 `open`, `public`, `internal`, `package`, `fileprivate`, `private`의 6가지 접근 레벨을 제공한다.
- 모듈과 접근 제어는 잘 설계된 소프트웨어의 핵심인 캡슐화, 정보 은닉을 가능하게 한다.

## 주요 기능
#### API 설계 및 안정성 확보
+ 프레임워크 개발 시, 외부 개발자가 사용해야 하는 기능은 `public`으로, 상속하여 확장할 수 있도록 허용하는 기능은 `open`으로 선언한다. 이를 통해 API의 의도를 명확히 전달하고, `public`으로 선언된 부분은 상속이 불가능하므로 프레임워크의 내부 동작의 일관성을 보장하여 안정성을 높인다.
#### 내부 구현 보호
+ 외부에 노출할 필요가 없는 대부분의 코드는 기본값인 `internal`로 두어 모듈 내에서만 자유롭게 사용한다. 이를 통해 내부 구현이 외부에 미치는 영향을 최소화하고, 향후 리팩토링 시 유연성을 확보할 수 있다.
#### 읽기 전용 프로퍼티 제공
```Swift
public private(set) var score: Int
```
+ 위와 같이 setter의 접근 수준을 더 낮게 지정할 수 있다. 이를 통해 외부에서는 값을 읽기만 가능하고, 값의 변경은 오직 해당 타입 내부에서만 가능하도록 강제하여 데이터의 무결성을 지킨다.
#### 모듈러 아키텍처 구축
+ 대규모 애플리케이션을 기능별(Feature) 또는 계층별(Layer)로 독립된 모듈로 분리하여 개발한다. 이는 팀 단위 협업을 용이하게 하고, 빌드 시간을 단축하며, 코드의 재사용성을 높이는 현대적인 앱 개발의 핵심 전략이다.

## 코드 예시
```Swift
// DesignSystemFramework.swift

import SwiftUI

// 외부에 공개할 테마 설정 구조체
public struct AppTheme {
    // 외부에서 접근 및 수정 가능한 하이라이트 색상
    public var highlightColor: Color
    
    // 프레임워크 내부에서만 사용되는 기본 패딩 값 (internal)
    let defaultPadding: CGFloat = 8.0
    
    public init(highlightColor: Color) {
        self.highlightColor = highlightColor
    }
}

// 외부에 공개할 ViewModifier
public struct HighlightModifier: ViewModifier {
    let theme: AppTheme
    
    public func body(content: Content) -> some View {
        content
            .padding(theme.defaultPadding) // internal 프로퍼티 접근 (성공)
            .background(theme.highlightColor)
            .cornerRadius(10)
    }
}

// View에 대한 확장을 public으로 제공하여 API를 간편하게 만듦
public extension View {
    func highlight(with theme: AppTheme) -> some View {
        self.modifier(HighlightModifier(theme: theme))
    }
}
```

```Swift
// MyApp의 ContentView.swift

import SwiftUI
import DesignSystem // DesignSystem 프레임워크 임포트

struct ContentView: View {
    var body: some View {
        VStack(spacing: 20) {
            // 1. 프레임워크의 public 타입을 사용하여 테마 생성
            let primaryTheme = AppTheme(highlightColor: .yellow.opacity(0.3))
            
            // 2. 프레임워크가 제공하는 public 확장 메서드 사용
            Text("SwiftUI 모듈 테스트")
                .font(.headline)
                .highlight(with: primaryTheme)

            // 3. internal 프로퍼티는 앱에서 접근 불가
            // let padding = primaryTheme.defaultPadding // 컴파일 에러!
            // 'defaultPadding' is inaccessible due to 'internal' protection level
            
            let secondaryTheme = AppTheme(highlightColor: .blue.opacity(0.2))
            
            Text("모듈화와 접근 제어")
                .font(.title2)
                .highlight(with: secondaryTheme)
        }
        .padding()
    }
}
```
## Keywords
+ [[접근 제어 (Access Control)]]
+ [[모듈 (Module)]]
+ [[캡슐화 (Encapsulation)]]

## References
- [The Swift Programming Language (Swift 6.2) - Access Control](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/accesscontrol/)
- [Swift Docs - Modules](https://github.com/swiftlang/swift/blob/main/docs/Modules.md)
- [swift.org - API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/)

## 작성자
- #Air 