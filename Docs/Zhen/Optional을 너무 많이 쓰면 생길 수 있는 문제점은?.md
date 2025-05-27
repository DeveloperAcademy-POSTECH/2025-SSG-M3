>[!question]
>GQ1. **Optional을 너무 많이 쓰면 생길 수 있는 문제점은?**

## Description
- optional 을 남용했을 때 생기는 문제점을 알아본다. 
-  optional 을 어떻게 사용해야 효율적으로 사용하는 것인지 알아본다. 
	-  optional 대신 사용할 수 있는 방법은? 

## 주요 내용
### Optional 남용

+ 과도한 nil 처리로 코드가 지저분해져 코드를 읽기 어렵고, 중첩이 깊어짐 > 유지보수 난이도 ↗️
```
if let a = a {
	if let b = b {
        if let c = c {
            // ...
        }
	}
}
```

- 실제로 nil이 나올 일이 없는 필드도 습관처럼 optional 을 사용하게 됨
	- > 필요하지 않은 optional 은 오히려 버그 가능성을 키우게 됨. 

- 결국, optional 자체가 나쁜 것은 아니지만, 잘못 쓰면 코드가 복잡해지고 버그가 생길 가능성이 높아진다! 

***

### 어떻게 해야 optional 을 잘 사용하는 걸까? 
>꼭 필요할때만 `?` 붙이기
>값이 항상 존재한다면 non-optional 사용
>가능하면 `guard let`, `??`, `optional chaining` 으로 안전하게 처리 
> SwiftUI에서는 Optional 바인딩이 번거로우니 가능하면 피하기

### cf. optional 을 피하면서 유연하게 설계하는 방법은 없을까? 
1. 필요 없는 optional 은 제거하고 기본값으로 대체
   
```
struct User {
	var id: String = "no id"
}
```

2.  nil 대신 enum 사용해서 상태 분명하게 표시하기 

	`var errorMessage: String?
		이거 대신에 
	```
	enum NetworkState {
	    case loading
	    case success(data: String)
	    case failure(error: String)
	}
	```
`
3.  Result<T, Error> 사용하기 
```
func fetchData() -> Result<String, NetworkError> {
    if success {
        return .success("Hello")
    } else {
        return .failure(.timeout)
    }
}
```




## Keywords
+ [[옵셔널(Optionals)]]

## References
- Blog
	- https://f-lab.kr/insight/safely-handling-optionals-in-swift