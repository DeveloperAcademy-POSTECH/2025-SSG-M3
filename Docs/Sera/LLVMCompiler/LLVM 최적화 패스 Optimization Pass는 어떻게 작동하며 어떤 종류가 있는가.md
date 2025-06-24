>[!question]
>GQ1. 1. LLVM 최적화 패스 Optimization Pass는 어떻게 작동하며 어떤 종류가 있는가
## Description
- LLVM = Low-Level Virtual Machine : swift로 작성한 코드를 기계어로 바꾸는 과정에서 중간 단계를 담당
1. Parser (Swift 문법 분석) → 사용자가 쓴 swift 코드를 분석해서 구문 트리(AST)를 만듬
2. SIL 생성 (Swift Intermediate Language) → Swift만의 중간 표현 언어로 변환됨
3. LLVM IR로 변환 (Intermediate Representation) → SIL을 LLVM이 이해할 수 있는 IR로 바꿈
4. LLVM 최적화/코드 생성→ LLVM이 최적화하고, 실제 CPU가 실행 가능한 **기계어(바이너리)** 코드로 바꿔줌
5. 실행 파일 생성 → 결과물을 앱으로 패키징

## 주요 기능
+ Pass는 코드에 어떤 최적화를 적용하는 단계 - LLVM IR을 여러 단계에 걸쳐 점점 더 빠르고, 작고, 효율적인 코드로 만드는 방식

LLVM의 pass는  크게 3가지 유형으로 나뉘고, 그 유형 안에 여러개의 세부 패스들이 있음
1. Transform Passes (변형 패스) → 실제로 코드를 수정하거나 개선하는 패스
2. Analysis Passes (분석 패스) → 코드를 바꾸진 않지만, 다른 pass들이 필요로 하는 정보를 제공하는 패스
3. Utility Passes (도움 도구 패스) → 디버깅, 테스트, 내부 체크 등을 위한 패스

1번 종류

|이름|설명|
|---|---|
|`mem2reg`|변수를 SSA 형태로 바꿔 레지스터 기반 최적화 가능하게|
|`instcombine`|연산 단순화 (`a + 0` → `a`)|
|`reassociate`|연산 순서 재조정 (`(a+b)+c` → `a+(b+c)`)|
|`GVN` (Global Value Numbering)|중복 계산 제거|
|`SCCP` (Sparse Conditional Constant Propagation)|조건 상수 추론|
|`DCE` (Dead Code Elimination)|사용되지 않는 코드 삭제|
|`loop-unroll`|루프 본문 복사로 반복 횟수 줄이기|
|`inline`|함수 호출 제거, 함수 본문 직접 넣기|
2번 종류

|이름|설명|
|---|---|
|`LoopInfo`|반복문 구조 분석|
|`DominatorTree`|기본 블록 간 지배 관계 분석|
|`AliasAnalysis`|두 포인터가 같은 메모리 가리키는지 분석|
|`CallGraphAnalysis`|함수 호출 그래프 생성|
|`ScalarEvolution`|루프 반복 횟수 예측|
3번 종류

| 이름       | 설명                       |
| -------- | ------------------------ |
| `verify` | IR이 규칙 위반 없이 잘 생성되었는지 검증 |
| `print`  | IR 구조 출력                 |
| `lint`   | 잠재적인 오류 검사               |

## 코드 예시

```swift
func demo(x: Int) -> Int {
    let a = x * 1
    let b = x + 0
    let c = a + b
    let d = c * 0
    return d
}
```

| 코드         | 최적화 대상            | 이유                          |
| ---------- | ----------------- | --------------------------- |
| `x * 1`    | `x`               | 곱하기 1은 불필요 → instcombine    |
| `x + 0`    | `x`               | 더하기 0도 불필요 → instcombine    |
| `a + b`    | `x + x` → `2 * x` | 경우에 따라 단순화 가능               |
| `c * 0`    | `0`               | 어떤 수든 0 곱하면 0 → instcombine |
| `return d` | `return 0`        | 모든 게 상수라서 DCE 가능성 존재        |

## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- #Sera 