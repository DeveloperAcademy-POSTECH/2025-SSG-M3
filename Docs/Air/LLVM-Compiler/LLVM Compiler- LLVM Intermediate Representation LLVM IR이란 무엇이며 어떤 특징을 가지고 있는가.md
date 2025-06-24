>[!question]
>GQ. 1. LLVM Intermediate Representation LLVM IR이란 무엇이며 어떤 특징을 가지고 있는가

## Description
- LLVM Intermediate Representation(IR)은 LLVM 컴파일러 인프라에서 사용하는 중간 표현 언어
- 고수준 언어(C, C++, Swift 등)를 기계어로 번역하는 과정의 중간 단계에서 사용되며, 플랫폼 독립적이고 최적화에 유리한 구조로 설계된다.
- Swift는 컴파일 과정에서 소스 코드를 먼저 SIL(Swift Intermediate Language)로 변환한 후, 이를 다시 LLVM IR로 변환하여 최종적으로 네이티브 코드(machine code)로 컴파일한다.
- 따라서 LLVM IR은 Swift 코드가 실행 파일로 변환되기까지의 중요한 전환점이며, 성능 최적화와 플랫폼 간 이식성에 크게 기여한다.

## 주요 기능
#### 플랫폼 독립성
- LLVM IR은 추상화된 명령어 집합을 사용하여 여러 CPU 아키텍처에 대응 가능하다. Swift는 이를 활용하여 iOS, macOS, Linux 등 다양한 플랫폼에서 동일한 코드를 실행 가능하게 한다.
#### 최적화
- LLVM IR 수준에서 다양한 최적화가 이루어짐. 예를 들어, Swift의 `@inline`, `@inlinable` 속성은 LLVM 최적화 과정에 영향을 주어 런타임 성능을 향상시킨다.
#### 타겟 코드 생성 지원
- LLVM IR은 다양한 백엔드(실제로 기계가 실행할 수 있는 기계어 코드를 만드는 단계, x86, ARM 등 CPU 종류에 떠러 다른 명령어 체계)로 타겟 코드를 생성할 수 있어 Swift가 여러 플랫폼에 배포 가능한 바이너리를 만들 수 있게 한다.
#### 디버깅 및 분석 기능
- Swift 컴파일러에서 `-emit-ir` 옵션을 통해 IR 코드를 출력할 수 있어 코드 분석 및 성능 디버기에 유용하다.

## 코드 예시
#### Swift 코드
```Swift
func add(_ a: Int, _ b: Int) -> Int {
	return a + b
}
```
#### Swift 컴파일 시 LLVM IR 출력
터미널 명령어 (IR 출력):
```bash
swiftc -emit-ir -O add.swift -o add.ll
```
생성된 LLVM IR 중 add(a:b:) 부분
```llvm
define hidden swiftcc i64 @"$s3addAAyS2i_SitF"(i64 %0, i64 %1) local_unnamed_addr #1 {
entry:
  %2 = tail call { i64, i1 } @llvm.sadd.with.overflow.i64(i64 %0, i64 %1)
  %3 = extractvalue { i64, i1 } %2, 1
  br i1 %3, label %6, label %4, !prof !17

4:                                                ; preds = %entry
  %5 = extractvalue { i64, i1 } %2, 0
  ret i64 %5

6:                                                ; preds = %entry
  tail call void asm sideeffect "", "n"(i32 0) #4
  tail call void @llvm.trap()
  unreachable
}
```

#### 최적화
`@inline`, `@inlinable`
```Swift
@inlinable func add(_ a: Int, _ b: Int) -> Int {
	return a + b;
}
```
- `@inlineable` 키워드가 붙은 add 함수는 LLVM IR로 변환될 때, 호출된 함수 안에 그 내용이 직접 복사되어 들어간다.
- 호출 비용이 줄어들고 더 빨라질 수 있다.

## Keywords
- [[LLVM 컴파일러]]
+ [[LLVM IR]]

## 작성자
- #Air 