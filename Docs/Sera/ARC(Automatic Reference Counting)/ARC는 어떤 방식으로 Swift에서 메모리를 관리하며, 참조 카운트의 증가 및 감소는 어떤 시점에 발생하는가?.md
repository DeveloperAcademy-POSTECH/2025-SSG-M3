>[!question]
>GQ1. ARC는 어떤 방식으로 Swift에서 메모리를 관리하며, 참조 카운트의 증가 및 감소는 어떤 시점에 발생하는가?

## Description
- ARC는 앱의 메모리를 자동으로 관리하여 개발자가 메모리 관리에 크게 신경 쓰지 않도록 도와주는 시스템. 더 이상 필요 없는 클래스 인스턴스를 메모리에서 알아서 해제하여 메모리 누수(memory leak)를 막아줌
- 그래서 ARC가 어떻게 알아서 메모리 관리를 해주냐? 메모리의 참조 횟수(RC-Reference Count)를 계산하여, 참조 횟수가 0이 되면 더 이상 사용하지 않는 메모리라 생각하여 해제함(컴파일 시간에 자동으로 처리됨)
- 따라서 모든 인스턴스는 자기만의 RC를 갖고있다. 


## 주요 기능
### 참조 카운트가 증가하는 시점 (Retain)
새로운 강한 참조가 생길 때마다 참조 카운트는 1씩 증가함

1. 새로운 인스턴스를 생성하여 변수나 상수에 할당할 때
```swift
// Person 인스턴스가 생성되고 person 상수가 강한 참조를 시작
// Person 인스턴스의 참조 카운트: 1
let person = Person(name: "김세라")
```

2. 기존 인스턴스를 다른 변수나 상수에 할당할 때 
```swift
// samePerson도 동일한 인스턴스를 강하게 참조합니다.
// Person 인스턴스의 참조 카운트: 2
let samePerson = person
```

3. 클래스 인스턴스를 함수의 인자로 전달하거나, 컬렉션(배열, 딕셔너리 등)에 추가할 때
```swift
var people = [Person]()
// 배열에 추가되면서 인스턴스를 강하게 참조합니다.
// Person 인스턴스의 참조 카운트: 3
people.append(person)
```

### 참조 카운트가 감소하는 시점 (Release)
강한 참조가 사라질 때마다 참조 카운트는 1씩 감소함
1. 참조하던 변수나 상수에 `nil`을 할당할 때 (옵셔널 타입의 경우)
```swift
// aPerson의 참조가 끊어집니다.
// Person 인스턴스의 참조 카운트: 1 감소
var aPerson: Person? = Person(name: "세라")
aPerson = nil
```

2. 참조하던 변수나 상수에 다른 인스턴스를 할당할 때
```swift
var person = Person(name: "원래 인스턴스")
// 새로운 인스턴스를 할당하면, "원래 인스턴스"에 대한 참조가 끊어집니다.
// "원래 인스턴스"의 참조 카운트: 1 감소
person = Person(name: "새로운 인스턴스")
```

3. 참조하던 변수나 상수가 범위를 벗어나 메모리에서 해제될 때
```swift
func createPerson() {
    // aPerson은 createPerson 함수 내에서만 유효합니다.
    let aPerson = Person(name: "이준")
    // 함수가 종료되면 aPerson은 사라지고, 참조가 끊어집니다.
    // "장길산" 인스턴스의 참조 카운트: 1 감소
}
createPerson()
```
## Keywords
+ 파생된 키워드들을 작성

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- 작성한 사람의 닉네임 (예: # Air )