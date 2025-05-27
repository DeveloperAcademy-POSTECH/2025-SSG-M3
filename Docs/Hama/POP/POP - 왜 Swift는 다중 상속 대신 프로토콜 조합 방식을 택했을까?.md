
<aside> 📌

프로토콜 지향 프로그래밍(POP)은 기존 객체지향(OOP)의 단점들을 보완하기 위해 Swift에서 제안된 설계 철학이다.

</aside>

OOP는 클래스를 중심으로 상속을 통해 기능을 확장하지만, 이 방식은 상속 관계가 복잡해지고, 유연성이 떨어지는 단점이 있다. 반면, POP는 프로토콜 + 익스텐션 + 합성(composition) 을 통해 필요한 기능만 골라 조합할 수 있고, 특히 구조체, 클래스, 열거형 등 모든 타입에 적용할 수 있다. 또한 기능 단위로 역할을 분리하기 때문에 유지보수성과 테스트 용이성이 훨씬 좋아진다.

---

### **OOP의 한계**

- **상속 중심**: 클래스 기반 계층 구조로 인해 설계가 경직되고, 다중 상속이 불가능해 유연성이 떨어짐.
    
    GQ: 왜 클래스 기반 계층 구조는 설계가 경직이 되는걸까?
    
    GA 1: Swift에서는 한 클래스가, 하나의 부모 클래스만 상속받을 수 있기 때문이다.
    
    ```swift
    class Animal {
		func move() {
    		print("움직인다")
    	}
    }
    
    class SoundMaking {
    	func makeSound() {
    		print("소리를 낸다")
    	}
    }
    
    class Dog: Animal, SoundMaking {
    // Error: Multiple inheritance from classes 'Animal' and 'SoundMaking'
    // Swift에서는 다중 상속이 불가능하기 때문에, 이 코드는 에러가 난다!
    }
    ```
    
    GA 2: 그러나, POP는 프로토콜 여러 개를 동시에 채택해서, 다중 상속처럼 다양한 역할을 조합할 수 있습니다. 이것이 POP의 가장 큰 장점이지요.
    
    ```swift
    protocol Animal {
        func move()
    }
    
    protocol SoundMaking {
        func makeSound()
    }
    
    struct Dog: Animal, SoundMaking {
        func move() {
            print("네 발로 움직인다") //개에 맞게 커스텀 가능!
        }
        
        func makeSound() {
            print("멍멍!") //개에 맞게 커스텀 가능!
        }
    }
    ```
    
- **코드 중복**: 공통 부모가 없으면, 중복 구현이 발생하기 쉬움.
    
    GQ: 왜 공통 부모가 없으면, 중복 구현이 발생하기 쉬울까?
    
    GA 1: 아래 코드와 같이, 둘 다 fly() 메서드를 갖고 있지만, 공통된 부모 클래스가 없기 때문에, fly()를 각 클래스에서 따로 중복해서 구현해야만 합니다.
    
    ```swift
    class Bird {
        func fly() {
            print("날개를 퍼덕이며 난다")
        }
    }
    
    class Airplane {
        func fly() {
            print("엔진을 사용해 난다")
        }
    }
    ```
    
    GA 2: 그렇다해서 부모 클래스를 억지로 만들게 되면, 논리적으로도 맞지 않는 ‘is-a’ 관계를 강제로 만들게 되어 부자연스러워 집니다. Bird 는 Machine 이 아니지만, 부모 클래스를 억지로 끼워맞춘 것이지요.
    
    ```swift
    class FlyableMachine {
    func fly() {
	    print("난다") // 너무 추상적이라 의미가 없다
		}
    }
    
    class Bird: FlyableMachine {}   // 말이 안 되는 상속 구조다
    class Airplane: FlyableMachine {}
    ```
    
- **런타임 결정**: 동적 디스패치(dynamic dispatch)에 의존하여, 런타임 비용이 증가됨.
    
    GQ: 동적 디스패치에 의존하면, 왜 런타임 비용 증가하는 걸까?
    
    GA: OOP에서 클래스는 메서드를 오버라이드(override) 할 수 있는데, 어떤 메서드를 실제로 실행해야 할지 컴파일 타임에 정할 수 없기 때문에 컴파일 타임이 아닌, 런타임에서 결정된다. 그렇기 때문에, CPU 명령 수가 증가하고, 고성능, 반복적인 연산에서는 특히나 문제가 될 수 있다.
    
    GQ: 컴파일 타임은 뭐고, 런타임은 뭘까?
    
    GA: 컴파일 타임은 내가 코드를 작성하고 ‘빌드 버튼’을 눌렀을 때 벌어지는 일이고(한번만 일어남, 정적 디스패치), 런타임은 사용자가 화면을 터치하거나 어떤 동작을 해야 일어나는 일(동적 디스패치)이다.
    
    ```swift
    class Animal {
    func speak() {
	    print("...")
	    }
    }
    
    class Dog: Animal {
	    override func speak() {
		    print("멍멍!")
		}
    }
    
    let pet: Animal = Dog()
    pet.speak() // 어떤 speak()를 호출할지는 런타임에 결정됨
    ```
    
    ```swift
    protocol Speaker {
        func speak()
    }
    
    struct Dog: Speaker {
        func speak() {
            print("멍멍!")
        }
    }
    
    let pet = Dog()
    pet.speak() // 어떤 speak()를 호출할지 컴파일타임에 이미 결정됨
    ```
    

### **POP의 핵심 요소**

- **프로토콜(Protocol)**: 역할과 기능을 정의하는 추상적인 인터페이스
- **익스텐션(Extension)**: 기존 타입에 기능을 추가해, 코드 재사용성 향상
- **합성(Composition)**: 여러 개의 프로토콜을 조합해 유연한 구조 설계

### **POP의 장점**

- **재사용성 향상**: 중복 최소화 및 기능 분리
- **유연한 설계**: 필요한 기능만 조합해 타입 설계 가능
- **테스트 용이성**: 역할이 잘 나뉘어 있어, 이것저것 테스트가 간단
- **모든 타입에 적용 가능**: Class 뿐만 아니라 Struct, enum 에도 적용 가능
- **정적 디스패치**: 컴파일 타임 결정 → 성능 향상


#Hama