>[!question]
>GQ. Apple Foundation Model의 사용법을 알아보자

## Description
- Foundation Model은 iOS 26 이상부터 사용할 수 있는 온디바이스 AI 프레임워크이다.
- Foundation Models 프레임워크는 Apple의 온디바이스 LLM에 대한 액세스를 제공한다.

## 주요 기능
+ 요청을 기반으로 텍스트 생성
+ 요청을 기반으로 Swift 객체를 생성
+ 요청을 기반으로 함수를 호출

## 코드 예시
##### Generative Model
```Swift
import FoundationModels

// 모델이 생성할 데이터의 구조를 정의
@Generable
struct Recipe {
    @Guide(description: "레시피의 이름")
    var name: String
    
    @Guide(description: "조리 시간(분 단위)")
    var cookingTime: Int
    
    var ingredients: [String]
}

// 언어 모델 세션을 시작하고, 구조화된 응답을 요청
#Playground {
    let session = LanguageModelSession()
    let response = try await session.respond(to: "30분 안에 만들 수 있는 간단한 파스타 레시피 알려줘", generating: Recipe.self)

    // Recipe 타입으로 바로 결과를 받아 활용 가능
    let recipe = response.content
    print("레시피 이름: \(recipe.name)")
    print("조리 시간: \(recipe.cookingTime)분")
    print("재료: \(recipe.ingredients.joined(separator: ", "))")
}
```
##### Tool Calling
```Swift
struct WeatherTool: Tool {
	let name = "getWeather"
	let description = "Retrieve the latest weather information for a city"
	
	@Generable
	struct Arguments {
		@Guide(description: "The city to get weather information for")
		var city: String
	}
	
	struct Forecast: Encodable {
		var city: String
		var temperature: Int
	}
	
	func call(arguments: Arguments) async throws -> ToolOutput {
		var temperature = "unknown"
		// Get the temperature for the city by using `WeatherKit`.
		let forecast = GeneratedContent(properties: [
			"city": arguments.city,
			"temperature": temperature,
		])
		return ToolOutput(forecast)
	}
}

// Create a session with default instructions that guide the requests.
let session = LanguageModelSession(
	tools: [WeatherTool()],
	instructions: "Help the person with getting weather information"
)

// Make a request that compares the temperature between several locations.
let response = try await session.respond(
	to: "Is it hotter in Boston, Wichita, or Pittsburgh?"
)
```
## Keywords
+ [[Foundation Models]]

## References
- [Apple Developer: FoundationModels](https://developer.apple.com/documentation/foundationmodels)

## 작성자
- #Air