
>[!question]
>GQ1. 1. LLVM Pass Manager 구조와 커스텀 패스를 추가 또는 비활성화하는 방법은 무엇인가

## 정의
- **LLVM**  : Low Level Virtual Machine의 줄임말로, 컴파일러를 만들기 위한 프레임워크임. Swift, Clang 같은 언어가 LLVM을 이용해 소스코드를 기계어로 번역함.
- **IR (Intermediate Representation)**  : LLVM 내부에서 사용하는 중간 언어. 사람이 읽을 수 있고, 최적화와 분석을 하기 좋게 설계된 코드 형태임.
- **Pass (패스)**  : IR을 읽어서 분석하거나(IR 분석 패스), 수정하는(IR 변환 패스) 작은 프로그램 단위임.  
		예: Dead Code Elimination (쓸모 없는 코드 제거), Constant Folding (상수 계산 미리 처리)
- **Pass Manager (패스 매니저)**  : 여러 패스를 순서대로 실행해주는 관리자.
## Description
- LLVM Pass Manager는 **패스들을 모아서 실행하는 컨트롤러**
- Pass Manager는 다음과 같은 역할을 수행함:
	- 어떤 패스를 실행할지 등록
	- 실행 순서 관리
	- 중간 결과 전달
	- 불필요한 패스 실행 방지

- 근데? LLVM은 전통적으로 **Legacy Pass Manager**와 **New Pass Manager** 두 가지를 제공함.
- 요즘은 (swift) New Pass Manager 가 기본 제공. 
- 커스텀 패스를 추가하거나 비활성화한다는 것은? 
	- **추가** → 내 코드 분석/변환용 패스를 Pass Manager에 등록해 실행되게 함
	- **비활성화** → 기존 최적화 패스를 끄거나 특정 패스만 빼고 실행되게 함
## 주요 기능
+ **커스텀 패스 추가**
	-   New Pass Manager에서는 `PassBuilder`를 통해 커스텀 Pass를 Pass Pipeline에 추가함.
	-  Pass는 ModulePass / FunctionPass / LoopPass 등의 형태로 작성됨.

- **패스 비활성화**
	- Pass Pipeline 생성 시 특정 최적화를 끄거나 제외할 수 있음.
    - `opt` 툴이나 Clang 빌드 시 `-disable-llvm-passes` 등의 플래그를 사용함.

- 무슨 말인지 모르겠다... 면?  (사실 저도 잘 모르겠어요)
	- SwiftUI 개발자가 LLVM Pass Manager를 직접 작성하거나 관리할 일은 거의 없음
	- 근데? **Swift 컴파일러 내부는 LLVM 기반**임.
	- Swift 코드 → Swift Intermediate Language (SIL) → LLVM IR → 기계어로 변환됨.
	- 이 과정에서 **LLVM Pass Manager가 최적화 패스를 돌려 코드 크기·속도를 개선**함. 
	- (즉 내부적으로 swift가 알아서 해준다~ )

	- 빌드 설정에서 약간 관여할 수 있는데, **릴리스 빌드**는 Pass Manager가 돌면서 dead code 제거, 상수 계산, 루프 최적화 등을 함 , **디버그 빌드**는 Pass Manager 최소한만 돌려 디버그 정보 보존
```
# 릴리스 빌드 (최적화 Pass 실행됨)
swift build -c release

# 디버그 빌드 (최적화 Pass 거의 없음)
swift build -c debug

# LLVM 최적화 Pass 끄기 (학습/실험용)
swift build -Xswiftc -Xllvm -Xswiftc -disable-llvm-passes
```

결론 ! 
- 직접 Swift 코드에서 LLVM Pass를 끄거나 추가하지는 못함. (Swift 컴파일러 자체를 포크하거나 LLVM IR을 출력(`-emit-ir`) 후 직접 `opt` 명령어로 Pass를 돌려야 함... 즉 LLVM opt 도구로 직접 관여해야 한다는 것)
- Swift 소스 코드나 Xcode에서 LLVM Pass를 직접 추가하거나 삽입할 수는 없음

그러면? 
실용적으로 XCode 에서 LLVM Pass 동작에 영향을 주는 세팅을 알아보자 ! 

- 최적화 수준 (Optimization Level)
	- Build Settings > Swift Compiler - Code Generation > Optimization Level
	- - **None (-Onone)**  → 디버그용, LLVM Pass 거의 없음 (디버그 심볼 보존, 최적화 비활성화)
	- **Fast (-O)**  → 기본 최적화, LLVM Pass들이 실행됨
	- **Fastest, Smallest [-Os] → (<u>Release 모드 디폴트</u>)** 크기에 맞게 최적화(일반적으로 코드 크기를 늘리지 않는 모든 최적화 활성화)
	- **Fast, Whole Module Optimization (-O -whole-module-optimization)**  → 모듈 전체 최적화, 더 aggressive
    - **Aggressive (-Ofast)**  → 가장 빠른 최적화 + aggressive optimization 수행 (표준을 위반할 수 있기 때문에 잘 정의된 코드에서만 해야하는 위험성이 존재)

- 추가 빌드 플래그 (Other Swift Flags / Other C Flags)
	- Build Settings > Swift Compiler - Custom Flags > Other Swift Flags
	-  `-Xllvm -disable-llvm-passes`  
	    → LLVM Pass를 강제로 꺼서 IR이 거의 가공되지 않음. 학습용·디버그용
	- `-Xllvm -print-after-all`  
	    → Pass 실행 직후 IR을 모두 출력 (터미널에 덤프)
## Keywords
+ [[LLVM 컴파일러]]

## References
- [https://developer.apple.com/documentation/xcode/build-settings-reference?changes=_9](https://developer.apple.com/documentation/xcode/build-settings-reference?changes=_9)
- https://ios-development.tistory.com/1218
- https://developer.apple.com/forums/thread/751099
- https://chibest.tistory.com/92


## 작성자
- #Zhen 