
> [!question]
> 도메인 주도 설계(DDD) 관점에서 데이터 모델을 정의할 때 Value Object, Entity, Aggregate의 개념을 어떻게 구분하고 설계해야 하는가?

---
도메인 주도 설계 DDD 관점에서,  현실 세계의 복잡한 비즈니스 규칙을 코드로 잘 반영하기 위한 전략이다. 
그 중심에는 Entity, Value Object, Aggregate라는 세 가지 핵심 개념이 있다.
이들은 단순히 “데이터를 담는 그릇”이 아니라, **도메인을 설명하는 단위이자, 설계의 방향을 제시하는 추상화 도구**다.

---
#### **“현실 세계의 복잡한 비즈니스 규칙”이란?**

**도메인 전문가가 일하는 비즈니스 환경**, 즉 **업무 세계**를 의미한다.
다시 말해, “현실 세계” = 사용자가 실제로 겪고 있는 문제 상황과 그에 대한 업무 처리 방식

이 “현실 세계” 안에는 다양한 **비즈니스 규칙**과 **업무 흐름**이 존재한다.
이것들이 소프트웨어에 잘 녹아들어야 실제로 작동하는 시스템이 될 수 있다.

- **은행 도메인**:
    - 하루 이체 한도 초과 시 승인 필요
- 쇼핑몰 도메인:
	- 주문 후 7일 내에만 반품 가능
- 교육 플랫폼 도메인:
	- 특정 과목은 선수 과목 수강 이력이 필요

그래서 DDD는 이런 **업무 규칙과 흐름을 정확히 반영하는 모델**을 만드는 것을 목표로 한다.
그 출발점이 바로 “현실 세계의 복잡한 비즈니스 규칙”이다.

---

# **1. Value Object**
값 자체로 의미가 있는 불변 객체.
**ID나 생애주기가 아닌, 값의 조합으로 동일성(equality)을 판단**한다.

- 식별자(ID)가 없다.
- 불변성(immutability)을 지닌다.
- 값이 같으면 같은 것으로 간주된다.
- 주로 수치, 날짜, 범위, 이름, 상태 등으로 쓰인다.

```swift
struct Email {
    let value: String
}
```
이메일은 사용자1의 이메일, 사용자2의 이메일처럼 ID로 구분하지 않는다.
값만 같으면 같은 것으로 간주한다. 따라서 Email("a@b.com") == Email("a@b.com") 은 참이다.
→ DB에 별도 테이블로 분리하지 않고, User 안에 inline 형태로 사용해도 된다.

---
# **2. Entity**
**고유한 ID로 식별되는 객체**.
값이 같아도 ID가 다르면 다른 것으로 간주한다.

- 식별자(ID)를 가진다.
- 상태는 변할 수 있지만, ID가 동일하면 같은 객체로 간주한다.
- 생애주기가 있다 (언제 생성되고, 어떻게 수정되며, 언제 소멸되는가).

```swift
struct User {
    let id: UUID
    var name: String
    var email: Email
}
```
사용자의 이름이나 이메일이 바뀌더라도 id가 같으면 동일한 User로 취급한다.

---
#  **3. Aggregate**
도메인의 일관성을 보장하기 위한 트랜잭션 경계 단위.
여러 Entity와 Value Object들을 묶어서 **한 덩어리(Aggregate Root)** 로 관리한다.

- 하나의 Aggregate에는 반드시 하나의 **Root Entity**가 존재한다.
- 외부에서는 이 Root를 통해서만 Aggregate 내부에 접근할 수 있다.
- 변경은 Aggregate Root를 통해서만 발생해야 한다.
- 변경 시 도메인 규칙을 함께 보장해야 한다.

```swift
struct Order {
    let id: UUID
    var items: [OrderItem]
    var shippingAddress: Address

    mutating func addItem(_ item: OrderItem) {
        items.append(item)
    }
}
```
Order는 Aggregate Root이고, OrderItem, Address는 내부 구성 요소이다.
→ 외부에서 OrderItem을 직접 수정하지 않고, Order의 메서드를 통해 수정해야 한다.
→ 이렇게 해야 **비즈니스 규칙(예: 주문은 반드시 하나 이상의 상품을 포함해야 함)**을 Root에서 통제할 수 있다.

---

#### Aggregate는 트랜잭션 경계이다
- 단일 Aggregate 내부의 변경은 한 트랜잭션에서 이뤄져야 한다.
- 다른 Aggregate를 변경해야 한다면, 이벤트 발행 또는 명시적 커뮤니케이션을 고려해야 한다.
- 하나의 요청에서 하나의 Aggregate만 변경하는 게 기본 원칙이다.

#### 명시적 커뮤니케이션 방식
- 직접 다른 Aggregate를 건드리지 않고,
  **도메인 서비스 또는 Application Layer에서 호출**로 처리한다.

```swift
// Application Service
func completeOrderAndTriggerPayment(orderId: UUID) {
    let order = orderRepository.findById(orderId)
    order.complete()

    let payment = paymentRepository.findByOrderId(orderId)
    payment.prepare()
}
```
→ Aggregate는 **오직 자기 책임만** 수행하고,
외부와의 조율은 도메인 서비스 같은 **상위 계층**에서 담당하게 하는 방식.


---

### GQ
- Aggregate 간 관계는 어떤 방식으로 연결해야 할까?
- Entity의 변경 이력을 추적하고 싶을 때 어떤 설계가 필요할까?

---
### Keywords
- [[데이터 모델링 (Data Modeling)]]
- DDD

### 작성자
- #Onething 