`Codable` = `Encodable` + `Decodable`

즉, 어떤 타입이 `Codable`을 채택하면 JSON 같은 외부 데이터 + Swift 객체 간의 변환을 할 수 있다는 뜻이다.

JSON 외부 데이터

```
{
	"name": "Alice",
	"age": 30
}
```

이것을 Swift에서 다루려면

```
struct Person: Codable {
	let name: String
	let age: Int
}
```

`Encodable` -> Swift 객체를 JSON으로 포장 `Decodable` -> JSON을 Swift 객체로 해석해서 꺼냄

#Nika