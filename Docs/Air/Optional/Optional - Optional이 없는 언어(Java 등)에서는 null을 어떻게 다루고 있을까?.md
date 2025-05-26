>[!question]
>GQ1. Optional이 없는 언어(Java 등)에서는 null을 어떻게 다루고 있을까?

## Description
- Optional을 지원하지 않는 언어에는 C/C++, Java(제한적 사용) JavaScript, Python, Go, Ruby 등이 있다.
- Optional이 없다는 것은 "없음(null, nil)"을 표현할 수는 있지만, 언어가 이를 안전하게 다루도록 강제하지 않는다는 뜻이다.

## C++
- Optional 없다. (C++17부터는 `std::optional<T>`이 표준으로 도입됐다.)
- 모든 포인터 타입에서 NULL 또는 nullptr를 사용한다.
- 프로그래머가 직접 null 체크를 해야 한다.
- 실수하면 Segmentation Fault가 발생할 수 있다.
```cpp
char* name = nullptr;
if (name != nullptr) {
    printf("%s\n", name);
}
```

## Java
- 모든 객체 타입은 기본적으로 null을 할당할 수 있다.
- 런타임에 체크를 하기 때문에 런타임 에러 발생 가능성이 높다.
- null인 값을 사용하려고 하면 NullPointerException이 발생한다.
- 프로그래머가 명시적으로 null 체크를 해줘야 한다.
- Java 8부터는 `Optional<T>`이 도입됐다.
	- 하지만 주로 리턴값에만 권장되며, 필드나 파라미터에서는 권장되지 않는다.
```java
if (user != null && user.getName() != null) {
    System.out.println(user.getName().length());
}
```

```java
Optional<String> name = Optional.ofNullable(user.getName());

name.ifPresent(n -> System.out.println(n.length()));

String safeName = name.orElse("기본값");
```

## Keywords
+ [[옵셔널(Optionals)]]
+ [[Null-Safety]]