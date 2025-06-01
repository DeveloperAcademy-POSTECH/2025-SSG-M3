1. 참조 타입(Reference Semantics)이 필요한 경우
	- 값 타입(struct)는 복사되고, 클래스는 **참조**로 전달된다. 상태를 공유하거나, 여러 객체가 동일한 **인스턴스를 참조**해야 할 때는 클래스가 필요하다.
```
class Counter {
	var value = 0
}

let a = Counter()
let b = a
b.value += 1
print(a.value)  // 1 (같은 인스턴스를 참조중)
```

2. 객체의 식별성이 중요한 경우
	- 값 타입(struct)은 상태가 같으면 같다고 여긴다.
	- 클래스는 인스턴스 고유성이 중요한 경우에 유리하다 ex) 게임 캐릭터, UI 컴포넌트, 네트워크 세션 등

3. 상속이 필요한 경우
	- Swift의 struct는 상속이 불가능하다
	- 상속을 활용한 공통 기능 확장이나 [[다형성]]을 원할 경우 클래스가 필요하다
```
class Animal {
	func speak() { print("Animal Sound") }
}

class Dog: Animal {
	override func speak() { print("Bark") }
}
```

이외에도 Object-C와의 호환이 필요한 경우이거나, 여전히 클래스 기반인 UIKit/AppKit과의 연동을 원할 때도 클래스가 필요하다.

|구분|클래스|구조체|
|---|---|---|
|타입|참조 타입 (Reference)|값 타입 (Value)|
|상속|O (가능)|X (불가능)|
|식별성|중요 (인스턴스 고유성)|상태 위주|
|메모리 공유|O (공유)|X (복사)|
|주 사용 예시|UIKit, 상태 공유, OOP 모델링|모델, 뷰모델, 데이터 전송 객체 등|
#Nika