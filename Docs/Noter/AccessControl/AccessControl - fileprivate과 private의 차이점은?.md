
>[!question]
>GQ1. fileprivate과 private의 차이점은?


### 1. 개요

private과 fileprivate은 Swift의 **접근 제어자(Access Control)** 중 하나로, 외부에서 코드의 접근을 제한하여 **은닉성과 안정성**을 높이기 위한 도구.

두 키워드는 모두 외부 접근을 막지만 **허용 범위(scope)**가 서로 다르다.

---

### 2. 정의 및 비교

| **키워드**     | **접근 허용 범위**                                                              | **주요 사용 목적**                                                        |
| ----------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| private     | **같은 파일 내 같은 타입**(클래스, 구조체 등)의 범위 안에서만 접근 가능<br>같은 파일에 있어도 타입 바깥에서는 접근 불가 | 내부 구현을 숨김으로써<br>실수로 접근하거나 변경되는 것을 막고 안정성을 높이기 위해                    |
| fileprivate | **같은 파일 내 모든 타입**에서 접근 가능<br>다른 클래스나 구조체, 전역 함수에서도 접근 가능                  | 한 파일 안에서 여러 타입 간의 상호작용이 필요한 경우<br>ex) 테스트 코드, 내부 헬퍼 클래스끼리의 데이터 공유 등 |
예)
 private - 클래스 내부에서 정의한 변수는 그 클래스 안에서만 사용할 수 있음
 fileprivate - 클래스 A에서 만든 변수를 같은 파일의 클래스 B에서 사용할 수 있음
 
---

### 3. 예제 코드 비교

**3.1  private 예시**
```swift
class Dog {
    private var name = "Dooboo"

    func printName() {
        print(name)
    }
}

let dog = Dog()
dog.printName()       // ✅ 가능
// dog.name           // ❌ 에러: name은 private
```
- name은 오직 Dog 내부에서만 접근 가능하다.


**3.2  fileprivate 예시**
```swift
fileprivate var sharedName = "Apple"

class A {
    func printShared() {
        print(sharedName)
    }
}

class B {
    func changeShared() {
        sharedName = "UpdatedName"
    }
}
```
- sharedName은 **같은 파일 안에 있는 A, B 클래스 모두** 접근 가능하다.

---
### 4. 언제 어떤 걸 써야 할까?

| **상황**                           | **적절한 키워드** | **이유**                   |
| -------------------------------- | ----------- | ------------------------ |
| 특정 타입 내부에만 숨기고 싶을 때              | private     | 타입 외부에서 완전히 은닉           |
| 같은 파일 내 여러 타입이 공유하는 내부 데이터를 다룰 때 | fileprivate | 유닛 테스트나 내부 협력이 필요한 경우 유용 |
대부분의 경우에는 private으로 충분하고, fileprivate은 특수한 상황에서만 사용되는 게 일반적.

---

### 5. 정리

- private은 **최소한의 접근 허용**을 보장하고,
- fileprivate은 **한 파일 내 타입 간의 협업을 위한 유연성**을 제공한다.
- Swift의 철학은 **최대한 private을 기본값으로 사용하고, 필요한 경우에만 fileprivate을 사용**하라는 방향이다.

---

### Keyword

- [[Access Control]]


### 작성자

#Noter