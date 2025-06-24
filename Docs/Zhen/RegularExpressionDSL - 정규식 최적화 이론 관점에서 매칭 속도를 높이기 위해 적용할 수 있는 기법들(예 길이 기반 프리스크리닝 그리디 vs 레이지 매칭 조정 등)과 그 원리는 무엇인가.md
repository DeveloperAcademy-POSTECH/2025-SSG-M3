>[!question]
>GQ1. 정규식 최적화 이론 관점에서 매칭 속도를 높이기 위해 적용할 수 있는 기법들(예 길이 기반 프리스크리닝 그리디 vs 레이지 매칭 조정 등)과 그 원리는 무엇인가
>

## 정의
- **정규식** : 문자열 패턴을 기술하기 위한 도메인 특화 언어(DSL). 다시 말해 메타문자(`*`, `+`, `?`, `|`, `()`, `[]`, `^`, `$`)를 조합해 “문자열 집합”을 기술하는 언어
- **매칭 속도**: 입력 길이 `n`에 대해 정규식 엔진이 매칭을 완료하는 데 걸리는 시간임.
### **정규식 최적화 이론들** 
> 
- **길이 기반 프리스크리닝(Length-based pre-screening)** : 패턴 전체가 매칭될 최소·최대 길이를 사전에 계산해, 대상 문자열의 길이가 범위 밖이면 본격적인 매칭 시도를 건너뜀
	예를 들어 `^.{5,10}$`이라면 문자열 길이가 5~10이 아닌 경우 곧바로 실패 처리함.
- **고유접두사(prefix) 또는 Boyer–Moore-Horspool 전처리** : 고정 문자열이나 클래스(예: `"error:"`)를 찾아야 하는 경우, Boyer–Moore-Horspool 알고리즘으로 빠르게 스캔한 뒤에야 정규식 엔진을 기동함
	- **Boyer–Moore-Horspool 알고리즘** : 알파벳의 각 기호에 대해 안전하게 건너뛸 수 있는 문자 수를 포함하는 표를 생성하기 위해 패턴을 전처리함... 그니까 '<u>문자열 내에서 특정 패턴을 검색하는 알고리즘</u>인데 ~ 패턴의 오른쪽에서 왼쪽으로 건너뛰면서 ~ <u>불일치시 건너뛰는 거리를 계산해서 검색 결과를 높이는 알고리즘</u>' 임. 
- **Greedy vs Lazy 매칭 조정** : `.*` 같은 “탐욕적(greedy)” 수량자 대신 `.*?` 같은 “지연(lazy)” 수량자를 사용해 불필요한 백트래킹을 줄임
		여기서 '탐욕적' 이라는 뜻은 백트래킹 알고리즘을 사용한다는 뜻. 예를 들어, 
		- 문자열 전체를 한 덩어리로 보고 검사한 다음
		- 매치되지 않으면 끝에서 한 문자를 빼고 다시 검사. 
		- 이렇게 하나씩 빼가면서 맞을때까지 검사. 
		- 즉, 정규표현식과 가장 잘 매치되는 가장 큰 덩어리를 찾으려고 하기 때문에 탐욕적이라는 이름이 붙은 것. 
	- lazy 는 그냥 앞에서부터 검사하는 것. 원본 문자열의 길이가 길면 탐욕적 수량자는 검사할 양이 많아지는 것. 
- **앵커(anchor) 사용** : `^…$`, `\b` 등을 적극 활용해 가능한 매칭 범위를 크게 좁힘
- **문자 클래스 최적화** : `[A-Za-z0-9_]`처럼 포괄 클래스 대신 `\w`나 특정 리터럴 집합을 사용해 내부 구현을 단순화함.

등... (너무 많음)


****## Description
- 정규식 최적화를 위해 **매칭 전처리**, **수량자 조정**, **그룹 전략** 등을 적용함으로써 **백트래킹 지연** 및 **불필요한 탐색 공간**을 줄여 매칭 속도를 높이는 기법이 있음. 

## 주요 기능
- **길이 기반 프리스크리닝**
    - 패턴이 매칭할 수 있는 최소·최대 길이를 계산하여 범위 밖 문자열은 곧바로 실패 처리함.
        
- **고정 리터럴 전처리 (Boyer–Moore-Horspool)**
    - 패턴 내 고정된 문자열을 빠르게 스캔하여 가능성 없는 위치를 건너뜀.
        
- **Greedy vs Lazy 수량자 조정**
    - 탐욕적(`*`, `+`) 수량자를 지연(`*?`, `+?`) 수량자로 바꿔 백트래킹 비용을 줄임.
        
- **앵커(Anchor) 활용**    
    - `^`, `$`, `\b` 등을 써서 검사 범위를 제한함.
        
- **패턴 단순화**
    - 중복·불필요 그룹 제거, 분기 최소화로 컴파일 시간 최적화함.

## 코드 예시
```
import Foundation

/// 길이 기반 프리스크리닝 + lazy 수량자(lazy quantifier) 예시
func fastRegexMatch(_ input: String) -> Bool {
    guard input.count >= 10 && input.count <= 100 else {
        return false
        // → 애초에 매칭 시도하지 않고 false 반환
    }
    
	//사용할 패턴을 정의
    // ^ERROR:  → 문자열의 시작에서 "ERROR:"가 와야 하고
    // \s+       → 한 칸 이상의 공백
    // (\d+?)    → 숫자(digit)가 1개 이상, lazy(게으른) 기법으로 최소 매칭
    // \b        → 단어 경계
    let pattern = #"^ERROR:\s+(\d+?)\b"#
    
     //NSRegularExpression 객체 생성
    //    - 패턴을 컴파일해서 내부 효율적인 형태로 변환
    let regex = try! NSRegularExpression(pattern: pattern, options: [])
    
   // 전체 문자열 범위를 NSRange 로 변환
    let range = NSRange(input.startIndex..<input.endIndex, in: input)
    
    // 실제 매칭 시도
    //    - firstMatch: 첫 번째 일치 결과를 찾아서
    return regex.firstMatch(in: input, options: [], range: range) != nil
}

// 사용 예
let logs = [
    "WARN: Disk almost full",
    "ERROR: 12345 unexpected end-of-file",
    "ERROR: no-number-here"
]

for line in logs {
    print("\(line) → \(fastRegexMatch(line))")
}
// 출력:
// WARN: Disk almost full → false
// ERROR: 12345 unexpected end-of-file → true
// ERROR: no-number-here → false

```


## Keywords
+ [[정규식 DSL]]

## References
- - Russ Cox, _Regular Expression Matching Can Be Simple And Fast_ [swtch.com](https://swtch.com/~rsc/regexp/regexp1.html?utm_source=chatgpt.com)
- RexEgg, _Regular Expression Optimizations_ [rexegg.com](https://www.rexegg.com/regex-optimizations.php?utm_source=chatgpt.com)
- Loggly Blog, _Five Invaluable Techniques to Improve Regex Performance_
- https://junstar92.tistory.com/202

## 작성자
#Zhen 