>[!question]
>GQ. String 타입이 유니코드 스칼라(Unicode Scalar)와 그래플렘 클러스터(grapheme cluster)를 어떻게 표현하며, 문자를 조작할 때 발생할 수 있는 함정은 무엇인가?

## Description
#### 유니코드 스칼라와 그래플렘 클러스터란?
- 유니코드 스칼라 값 (Unicode Scalar Value)
	- `U+0000` ~ `U+D7FF`, `U+E000` ~ `U+10FFFF` 범위의 코드 포인트
	- 예시
		- "A" -> `U+0041`
		- "에" --> `U+C5D0`
- 그래플렘 클러스터 (Grapheme Cluster)
	- 사람이 인식하는 하나의 문자 단위
	- 여러 유니코드 스칼라(코드 포인트)를 조합해서 하나의 문자처럼 보이는 것
	- 예시
		- e + `U+0301` -> é
	    - 🇰 + 🇷 → 🇰🇷

## 주요 기능
#### Swift에서의 String 구성
- Swift의 `String`은 완전한 유니코드 지원을 목표로 설계되었다. 즉, 사람이 한 글자로 인식하는 문자 단위인 그래플렘 클러스터를 기준으로 작동한다.
- 유니코드 스칼라
	- `Unicode.Scalar` 타입
	- 하나의 유니코드 코드 포인트
- 그라플렘 클러스터
	- `Character` 타입
	- 여러 개의 Unicode Scalar로 구성될 수 있음

## 코드 예시
#### Swift에서 문자 조작 시 주의할 점
##### 1. .count는 Character 단위
```Swift
let s = "🧑‍🧑‍🧒‍🧒" // 가족 이모지
print(s.count) // 1 —> 하나의 Character
print(s.unicodeScalars.count) // 7 —> 조합된 스칼라 수
```

##### 2. 인덱싱은 String.Index로, 정수 인덱싱 불가
```Swift
let str = "에어"
let index = str.index(str.startIndex, offsetsBy: 1)
print(str[index]) // "어"
```
- 직접 인덱스를 잘못 다루면 그래플렘 클러스터 경계를 깨뜨릴 수 있기 때문에, Swift는 정수 인덱싱을 아예 금지한다.
- 슬라이싱도 마찬가지

##### 3. == 는 정규화(nfc/nfd)를 고려 
```Swift
let precomposed = "é"           // U+00E9
let decomposed = "e\u{301}"     // U+0065 + U+0301
print(precomposed == decomposed) // true
```
- 유니코드에서는 같은 문자를 여러 방식으로 표현할 수 있는데, NFC는 가능한 한 단일 문자로 결합된 형태로 나타내고, NFD는 가능한 한 분해된 형태로 표현한다.
- Swift의 `==` 비교 연산자는  `String` API에서 자동으로 정규화를 고려하여 작동하기 때문에 결과는 같게 처리된다.
- 하지만, 내부 표현은 다를 수 있으므로 파일 저장이나 네트워크 전송 시 주의가 필요하다.
- 표현에 따라 Byte 수도 다르다.

## Keywords
- [[Swift Standard Library]]
+ [[String]]

## 작성자
- #Air 