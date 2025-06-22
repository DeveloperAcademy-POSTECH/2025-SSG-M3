
> [!question]
> Target Triple의 개념과 LLVM에서 플랫폼별 코드 생성 과정은 무엇인가?

## 개요
컴파일러가 소스 코드를 목적 코드(object code)로 바꾸는 과정은 단순히 문법 분석을 넘어서, **어떤 플랫폼에서 실행될지를 고려한 코드 생성**까지 포함한다.  
LLVM은 다양한 플랫폼을 지원하기 위해 `Target Triple`이라는 명확한 규약을 사용한다.

> 이 개념을 제대로 이해하면, Swift나 Rust, Clang 같은 언어들이 어떤 방식으로 다중 플랫폼을 지원하는지 구조적 이해가 가능해진다.

---
## 1. Target Triple이란?
**Target Triple**은 LLVM 컴파일러가 어떤 플랫폼을 타겟으로 코드를 생성할지 결정하는 핵심 식별자다.
- `x86_64-apple-darwin`
- `arm64-unknown-linux-gnu`
- `wasm32-unknown-unknown`

---

### 각 구성 요소의 의미
| 구성 요소          | 설명                                                                 |
| -------------- | ------------------------------------------------------------------ |
| `Architecture` | CPU 아키텍처. 예: `x86_64`, `arm64`, `wasm32`                           |
| `Vendor`       | 시스템 제조사 또는 플랫폼 공급자. 예: `apple`, `unknown`, `pc`                    |
| `OS`           | 타겟 운영 체제. 예: `darwin`(macOS), `linux`, `windows`                   |
| `Environment`  | ABI 또는 런타임 환경. 예: `gnu`, `musl`, `msvc`, `android`, 생략 가능          |

---

### Target Triple이 중요한 이유

컴파일러는 단순히 CPU 명령어 집합만 고려하는 게 아니다.  
운영 체제에 따라 시스템 콜, 호출 규약(calling convention), 메모리 정렬, 라이브러리 링크 방식 등이 모두 달라진다.  
**Target Triple은 이 모든 맥락을 압축해서 표현하는 "플랫폼의 DNA"와 같은 역할**을 한다.
예를 들어, `arm64-apple-ios`와 `arm64-unknown-linux-gnu`는 같은 아키텍처를 쓰지만,  
- iOS는 다이나믹 프레임워크 방식,
- Linux는 ELF 바이너리 및 glibc 기반  
으로 완전히 다른 방식으로 실행된다.

  
---
## 2. 왜 필요한가?
Target Triple은 LLVM이 **어떤 플랫폼용 코드**를 생성할지를 결정한다.  
단순히 아키텍처만 맞는다고 해서 같은 기계어를 사용할 수 있는 건 아니다.  
운영체제의 시스템 콜, 호출 규약(calling convention), 링크 방식 등이 다르기 때문이다.
즉, Target Triple은 **코드 생성의 전체 컨텍스트를 명시하는 역할**을 한다.

---

## 3. LLVM에서의 플랫폼별 코드 생성 흐름
1. **Frontend** 단계  
   - 예: Swift → SIL / C → LLVM IR  
   - 이 단계에서 언어 자체 문법을 파싱하고 중간 표현으로 변환한다.

2. **Target 설정**  
   - 컴파일 타겟을 명시 (`--target=x86_64-apple-darwin`)  
   - 이 정보를 기반으로 IR이 최적화되고 기계어로 내려갈 준비를 한다.

3. **Backend CodeGen**  
   - 각 아키텍처별로 존재하는 코드 생성기(Code Generator)가 해당 IR을 변환한다.  
   - 이 때 호출 규약, 레지스터 할당, 스택 프레임 레이아웃 등이 타겟에 맞게 처리된다.

4. **Linker 및 ABI 처리**  
   - 생성된 오브젝트 파일이 링커에 의해 OS에 맞게 묶인다.  
   - 환경에 따라 `glibc`, `musl`, `CoreFoundation` 등 ABI 차이도 고려된다.

---

## 4. Swift에서는 어떻게 활용되는가?
Swift 컴파일러도 LLVM을 기반으로 하고 있어서, SwiftPM이나 `swiftc` 명령어에서도 `--target` 옵션으로 타겟 플랫폼을 설정할 수 있다.

```sh
swift build --target x86_64-apple-macosx13.0
```

Swift는 기본적으로 macOS, iOS, Linux, Windows 등을 타겟으로 하고, 플랫폼 간의 차이를 LLVM Target Triple로 구분한다.


