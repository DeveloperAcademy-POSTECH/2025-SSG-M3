
<aside> 📌

**Optional Chaining** 은 “값이 없어도 그냥 넘어간다.” 는 상황에 적합합니다.
**guard let** 은 “값이 없으면, 코드를 실행하지 않고 빠져나간다.” 는 상황에 적합니다.

</aside>

### Optional Chaining

옵셔널 값이 nil 일 수 있고 아닐 수도 있는데, nil 일 경우에도 특별히 처리하지 않아도 되는 경우 사용합니다. 이 때, 마치 체인처럼 아래 코드와 같이 여러 단계의 옵셔널 프로퍼티 접근에 유용합니다.

```swift
let user: User? = getUser()
let email = user?.profile?.email ?? "이메일 없음"

// user 값이 없으면(nil) email 값은 "이메일 없음" 이다
// user 값이 있더라도 profile 이 없으면(nil) email 값은 "이메일 없음" 이다
// user 값이 있고 profile 이 있으면 email 이 존재하기에 그 email 값을 email 에 넣습니다
```

### guard let

옵셔널 값을, 반드시 꺼내야지만 다음 로직을 수행할 수 있게할 경우 사용합니다.

```swift
func showUserInfo(user: User?) {
    guard let user = user else {
        print("사용자가 없습니다.")
        return
    }
    print(user.name)
}
```

---

### Optional Chaining 과 guard let 을 함께 쓴 코드

```swift
func sendEmail(to user: User?) {
    guard let email = user?.profile?.email else {
        print("이메일이 없어서 보낼 수 없습니다.")
        return
    }
    print("이메일을 \\(email)로 보냈습니다.")
}
```

user 안에 profile이 있고, profile 안에 email이 있을 때만 그 email을 사용해서 아래 로직을 실행하고, 그렇지 않으면 함수에서 나가겠다(return) 는 뜻입니다.


#Hama 