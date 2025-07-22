**internal이 기본 접근 레벨인 이유와 장점은?**  
**public API 설계 관점에서 고려해야 할 점들을 이야기해보세요.**

Swift에서는 명시하지 않으면 기본적으로 `internal` 접근 레벨이 적용된다. 이는 **같은 모듈 안에서** 어디서든 접근할 수 있지만, 모듈 외부에서는 접근할 수 없는 제한이다. 


GQ. `public`과 `internal`의 차이가 무엇인가?
`internal`은 같은 모듈 내 어디서든 접근 가능. `public`은 다른 모듈에서도 접근 가능.
**같은 모듈 내의 의미?** -  Swift에서 모듈은 보통 하나의 앱, 프레임워크 단위 ex. 앱 전체가 하나의 모듈
그래서 `internal`이 기본 접근 레벨인 Swift에서는 같은 프로젝트 내 다른 파일 안에 있는 코드를 자유롭게 가져다 쓸 수 있음

public 예시
```swift
// MyLibrary 모듈 (Swift Package나 framework로 빌드됨)
public struct MathKit {
    public static func add(_ a: Int, _ b: Int) -> Int {
        return a + b
    }
}
```

```swift
// 다른 프로젝트(MyApp)
import MyLibrary

let sum = MathKit.add(3, 4)
```


### 왜 `internal`을 기본 접근 레벨로 했을까

1. 은닉화 장려: 내부 구현은 숨기고 외부에 꼭 필요한 것만 공개하도록 유도. 캡슐화 촉진
2. API 오염 방지: 실수로 외부에 노출되는 것을 방지해 API를 깔끔하게 유지할 수 있음
3. 모듈 단위 설계에 적합: Swift는 모듈 기반 컴파일을 지향하므로, 기본적으로 내부 모듈 간 협업은 허용하고 외부에는 숨기려는 철학 반영
4. 사용자가 신중하게 public/open 선언하게 유도: 공개 API는 변경 시 주의가 필요하므로, 기본적으로 `internal`로 제한하고 개발자가 명시적으로 노출 범위를 선택하도록 설계


#Nika 