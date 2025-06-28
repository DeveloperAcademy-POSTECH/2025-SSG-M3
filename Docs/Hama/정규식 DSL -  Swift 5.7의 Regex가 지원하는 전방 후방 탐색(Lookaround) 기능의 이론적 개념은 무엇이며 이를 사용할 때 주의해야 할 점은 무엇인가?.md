>[!question]
>GQ1. Regex(정규 표현식)이 지원하는 전방 후방 탐색(Lookaround) 기능의 이론적 개념이 뭘까?
>GQ2. Lookaround 기능을 사용할 때 주의해야 할 점은 무엇인가?|

## Description
GQ1. Regex(정규 표현식)이 지원하는 전방 후방 탐색(Lookaround) 기능의 이론적 개념이 뭘까?
* GA1. Lookaround(전방/후방 탐색)은 조건부 매칭을 위한 중요한 도구다. Lookaround 는 정규식에서 어떤 패턴이 앞이나 뒤에 있는지를 확인하되, 그 패턴 자체는 소비하지 않고 조건만 확인하는 기능이다.
## 주의할점
GQ2. Lookaround 기능을 사용할 때 주의해야 할 점은 무엇인가?
* GA2. 
1. 조건으로만 확인하고, 실제 결과에는 나오지 않는다.
   foo(?=bar) 는 foobar 중 foo 만 결과에 나온다.
2. 조건으로 쓴 글자가 결과에 안 나온다고 생각할 수 있다.
   (?<=foo)bar 는 foobar 중 bar 만 결과에 나온다.
3. (?=(a+)+)와 같이 조건이 복잡하면 결과가 오래걸릴 수 있다.
4. 후방 탐색은 복잡한 조건이면, 안 될 수도 있다.
5. Lookaround는 포인터를 움직이지 않아서 다음 패턴과 충돌할 수 있다.
## 코드 예시
* 전방 탐색
```swift
let regex = try! Regex("foo(?=bar)")
let text = "foobar"
text.wholeMatch(of: regex) // 매칭 결과는 "foo"
```
* 후방 탐색
```swift
let regex = try! Regex("(?<=foo)bar")
let text = "foobar"
text.wholeMatch(of: regex) // 매칭 결과는 "bar"
```

## Keywords
+ [[정규식 DSL]]

## 작성자
- #Hama 