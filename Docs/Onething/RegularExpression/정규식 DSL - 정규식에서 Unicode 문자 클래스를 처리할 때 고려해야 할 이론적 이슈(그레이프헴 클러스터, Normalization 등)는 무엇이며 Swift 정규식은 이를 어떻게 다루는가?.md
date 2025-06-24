
> [!question]
> 정규식에서 Unicode 문자 클래스를 처리할 때 고려해야 할 이론적 이슈(그레이프헴 클러스터, Normalization 등)는 무엇이며 Swift 정규식은 이를 어떻게 다루는가?

## 1. 문제의 출발점 - Unicode는 왜 다르게 취급되는가?

ASCII 문자만 다루던 시절의 정규식은 "한 글자 = 한 바이트 = 한 의미 단위"라는 단순한 모델이었다.  
하지만 유니코드는 전 세계의 문자, 기호, 이모지 등을 다루기 위해 **복합 문자 구성**, **조합형 문자**, **이모지 시퀀스** 같은 개념을 도입했다.
그래서 생기는 핵심 질문은 이거다.  
**정규식에서 '문자 하나'는 정확히 무엇을 의미할까?**

---
## 2. 이슈 1 - Grapheme Cluster
**Grapheme Cluster**는 사용자 눈에 '한 글자처럼 보이는 단위'를 의미한다.  
Swift에서도 `Character`는 유니코드 스칼라가 아니라 이 Grapheme Cluster를 기준으로 동작한다.
예를 들어:
```swift
let a1 = "a\u{0301}" // 'a' + Combining Acute Accent (U+0301)
let a2 = "\u{00E1}"  // 'á' precomposed (U+00E1)
print(a1.count) // 1
print(a2.count) // 1
print(a1 == a2) // false
```
정규식에서도 단순히 ., \w, . 등의 패턴으로는 Grapheme Cluster를 정확히 처리하지 못할 수 있다.
왜냐하면 .는 기본적으로 유니코드 스칼라 단위로 동작하고, 조합 문자는 둘 이상의 스칼라로 이루어질 수 있기 때문이다.

---

## **3. 이슈 2 - Normalization**
**Normalization(정규화)**는 동일한 시각적 결과를 갖는 유니코드 시퀀스를 하나의 통일된 형태로 바꾸는 과정이다.
예시:
- `é` → 단일 코드 포인트 U+00E9
- `e + ◌́`  → U+0065 U+0301 (조합형)
두 표현은 시각적으로 같지만, 바이트 수준이나 코드 포인트 비교에서는 다르다.
따라서 정규식을 사용하기 전, 텍스트가 동일한 정규화 방식(NFC, NFD 등)으로 정리되어 있어야 정확한 매칭이 가능하다.

---
## **4. Swift Regex는 이 문제를 어떻게 다루는가?**
Swift 5.7 이상에서 도입된 **RegexBuilder**와 Swift 정규식 리터럴은 Character 단위를 기본 단위로 삼는다.
즉, ., \w, \b 등은 Grapheme Cluster 기준으로 매칭된다.
이는 String도 Grapheme Cluster 기반으로 정의되어 있기 때문에 가능한 접근이다.

```swift
let regex = /á/   // Swift에서는 grapheme cluster 단위로 처리
"á".contains(regex) // true
```

또한, Swift는 String을 기본적으로 **NFC 정규화 상태**로 유지한다.
따라서 대부분의 실무에서는 별도의 Normalization 없이도 매칭이 일관되게 동작한다.

---

## **5. 주의할 점**
- 외부에서 가져온 문자열(JSON, DB 등)은 **정규화 상태가 보장되지 않을 수 있다**.
    → 비교 또는 정규식 매칭 전에 string.precomposedStringWithCanonicalMapping 같은 함수를 활용할 수 있다.
- 이모지처럼 ZWJ(Zero-Width Joiner)로 연결된 문자들도 하나의 Character로 판단된다.
    → Swift 정규식은 이런 경우도 Character 단위로 인식한다는 점에서 강력하다.

## **6. 정리**
- **Grapheme Cluster**: Swift에서는 문자(Character) 단위를 사용해 정규식에서도 사람이 인식하는 ‘글자’ 단위 매칭이 가능하다.
- **Normalization**: Swift의 String은 NFC로 유지되어 있어 대부분의 경우 별도 처리 없이도 정규식이 안정적으로 동작한다.
- **Swift Regex**는 유니코드 대응이 매우 강력하며, 대부분의 복잡한 문자 처리도 Character 단위로 해결 가능하다.


## GQ
- Grapheme Cluster 기반 정규식 매칭 성능은 어떻게 최적화될 수 있을까?
- Swift 정규식에서 이모지 시퀀스 또는 ZWJ 시퀀스를 정확히 필터링하려면 어떤 패턴을 사용해야 할까?

## Keywords
- [[정규식 DSL]]


#Onething 