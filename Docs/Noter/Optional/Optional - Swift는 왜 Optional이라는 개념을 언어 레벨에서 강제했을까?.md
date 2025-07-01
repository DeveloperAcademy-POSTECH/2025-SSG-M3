
>[!question]
>GQ1. Swift는 왜 Optional이라는 개념을 언어 레벨에서 강제했을까?

### 1. 개요

Swift는 **안정성**과 **예측 가능한 동작**을 매우 중시하는 언어다. 기존 언어(C, Object C 등)에서는 변수에 값이 없거나(NULL) 잘못된 참조가 있어도 컴파일러가 잡아주지 못해 런타임에서 앱이 터지는 경우가 많았다.

이 문제를 근본적으로 해결하기 위해 Swift는 Optional 타입을 도입하고,
값이 있을 수도 있고 없을 수도 있는 상황을 타입 시스템 차원에서 명시적으로 표현하도록 강제했다.

### 2. Optional이 등장한 배경

**2.1 런타임 크래시 방지**
- 값이 없을 수 있는 상황을 명확하게 타입으로 표현하므로, nil을 잘못 참조해서 앱이 죽는 현상을 미리 방지할 수 있다.

**2.2 안전한 코드 유도**
- Optional을 사용하면 "이 값이 nil일 수 있다"는 걸 컴파일러가 강제로 알려준다.
- 이를 통해 if let, guard let, nil coalescing 등을 통해 안전하게 값을 처리하도록 유도한다.

**2.3 명확한 의사 표현**
- 값이 항상 존재하는지, 아닐 수 있는지를 함수 시그니처나 모델 정의에서 명확히 표현할 수 있다. (예: var name: String? / var name: String)

### 3. Optional 처리 방식
| 처리 방식      | 설명                                    |
| ---------- | ------------------------------------- |
| 옵셔널 바인딩    | `if let`, `guard let` 등으로 안전하게 언래핑    |
| 강제 언래핑     | `!`로 강제로 꺼내기. 값이 없으면 런타임 에러 발생        |
| 옵셔널 체이닝    | `a?.b?.c`처럼 중간에 nil이면 전체가 nil이 되도록 연결 |
| nil 병합 연산자 | `??`로 기본값을 지정해 nil일 경우 대체             |

### 4. 예제 코드
```swift
var name: String? = "Noter"

// 옵셔널 바인딩*
if let unwrappedName = name {
    print("Hi \(unwrappedName)")
} else {
    print("There is no name")
}

// nil 병합 연산자
let nickname = name ?? "Guest" // nil일 경우 nickname을 "Guest"로 대체
print("Welcome, \(nickname)")

// 강제 언래핑
print(name!) // name이 nil이면 앱 터짐

```

바인딩*
: 어떤 값을 이름(변수/상수)에 할당해서 연결해두는 행위!
= 값과 이름을 바인딩(묶음)

```swift
let name = "Yeonsoo" // 이것도 "Yeonsoo"라는 값을 name이라는 상수에 바인딩한 것
```

그럼 옵셔널 바인딩은?

옵셔널 값이 nil이 아닌 경우 즉 값이 들어있는 경우
들어있는 값을 새로운 상수/변수에 '바인딩'해서 사용할 수 있게 하는 문법

위 예제 코드에서 nickname은 String? 타입.
이때 if let unwrappedName = nickname 구문은
"nickname이 nil이 아니면 그 안의 값을 꺼내서 unwrappedName에 바인딩해서 쓰자."는 의미

### 5. Optional이 없던 시절의 문제점

```objective-c
NSString *name = nil;
printf("%lu", name.length); // 크래시 발생 가능
```

- Objective-C에서는 값이 nil인지 아닌지 체크하지 않고 호출할 수 있었기 때문에, 런타임에서 예기치 못한 크래시가 자주 발생했다.
- Swift는 이런 오류를 아예 **컴파일 단계에서 차단**하고자 Optional을 도입했다.

### 6. 정리

Optional은 Swift가 지향하는 **안전하고 명확한 코드**를 실현하기 위한 핵심 개념이다.

> "값이 없을 수 있다는 사실"을 타입으로 강제하고, "그에 따른 안전한 처리 방식"을 언어 문법으로 제공한다.

이를 통해:
- 런타임 크래시를 줄이고
- 명확한 코드 흐름을 유지하며
- 개발자가 실수하지 않도록 컴파일러가 가이드라인을 제공할 수 있게 된다.

## Keyword

- [[Docs/Common/Keyword/Optional|Optional]]

#Noter