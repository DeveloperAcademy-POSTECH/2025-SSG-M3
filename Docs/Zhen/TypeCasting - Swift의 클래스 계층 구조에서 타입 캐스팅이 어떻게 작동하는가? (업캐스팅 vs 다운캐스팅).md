>[!question]
>[!question]
>GQ1. Swift의 클래스 계층 구조에서 타입 캐스팅이 어떻게 작동하는가? (업캐스팅 vs 다운캐스팅)
>GQ2. JSON 데이터 디코딩 시 타입 캐스팅이 필요한 상황은 어떤 케이스인가?

# Swift 의 클래스 계층 구조에서 타입 캐스팅이 어떻게 작동하는가? 
## Defining a Class Hierarchy for Type Casting 
-  클래스 및 하위 클래스의 계층 구조와 함께, 타입 캐스팅을 사용하여 특정 클래스 인스턴스의 타입을 확인하고( `is` 사용), 해당 인스턴스를 동일한 계층구조 내의 다른 클래스로 캐스트(Cast)할 수 있음

```swift
class MediaItem {
    var name: String
    init(name: String) {
        self.name = name
    }
}

class Movie: MediaItem {
    var director: String
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
}

class Song: MediaItem {
    var artist: String
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}

let library = [
    Movie(name: "죽은 시인의 사회", director: "피터 위어"),
    Song(name: "창공", artist: "김준석"),
    Movie(name: "인터스텔라", director: "크리스토퍼 놀란"),
    Movie(name: "공범자들", director: "최승호")
]

var movieCount = 0
var songCount = 0

  

for item in library {
    if item is Movie {
        movieCount += 1
    } else if item is Song {
        songCount += 1
    }
}

print("Media library는 \(movieCount)개의 영화와  \(songCount)개의 노래가 있어요!")

//Media library는 3개의 영화와  1개의 노래가 있어요!



```


## DownCasting 
- 특정 클래스 타입의 상수 혹은 변수는 하위 클래스의 인스턴스를 참조할 수 있는데, 이 때 `as!` 또는 `as?` 를 활용해서 서브 클래스 타입으로 다운캐스팅을 시도할 수 있음. 
	왜 optional 이냐면 다운캐스팅이 실패할 수도 있기 때문.. 잘못된 클래스 타입으로 시도하는데 '!' 활용하면 런타임에러 뜸

```Swift
for item in library {
    if let movie = item as? Movie {
        print("Movie: \(movie.name), dir. \(movie.director)")
    } else if let song = item as? Song {
        print("Song: \(song.name), by \(song.artist)")
   }
}

/*

Movie: 죽은 시인의 사회, dir. 피터 위어
Song: 창공, by 김준석
Movie: 인터스텔라, dir. 크리스토퍼 놀란
Movie: 공범자들, dir. 최승호

 */
```
- `as?` 는 옵셔널로 반환해서 `if let` 으로 꺼내온 것. 

## UpCasting 
- 서브 클래스 인스턴스를 super class 의 타입으로 참조함. upCasting은 항상 성공함. as 연산자를 이용할 수도 있다. 
무슨 말이냐면.. 

```Swift
class Human {
    let name: String = "Sodeul"
    init(name: String) {
        self.name = name
    }
}

class Teacher: Human {
    let subject: String = "English"
}

class Student: Human {
    let grade: Int = 1
}
```
이렇게 Teacher 와 Student 의 부모 클래스가 같은 경우에 

```Swift =

let people: [Human] = [
    Teacher.init(name: "teacher kim"),
    Student.init(name: "student kim"),
    Student.init(name: "student cho")
]
```
이렇게 people 이라는 배열에 teacher 랑 student 를 담을 수 있는 것임.. 
- 정확하게는 human 으로 upcasting 을 해서 담을 수 있는 것. 

```Swift 
let human = Teacher.init() as Humann
```
- 이렇게도 사용 가능. 이 의미는 Teacher 타입의 인스턴스를 생성하지만, Human 타입으로 upcasting 해서 human 에 저장하겠다는 뜻 ! 
- 그럼 어떻게 저장되냐면... 
	- 실제 human의 타입은 Human 이지만, 먼저 Teacher 라는 인스턴스를 만들었기 때문에 Teacher 란 인스턴스가 온전히 메모리에 올라감 (그림 참고)

- **human이 Teacher이란 서브 클래스를, Human이란 슈퍼클래스 타입으로 참조하는** **"업캐스팅"을 한 것이기 때문에, human의 접근 범위가 "Human" 멤버로 한정되는 것**임


### 실제로 xcode 에서 돌려보면? 
```Swift
**import** Foundation

**import** SwiftUI

  

**struct** TypeCasting: View {

    **var** body: **some** View {

        Text("TypeCastingView")

            .onAppear{

                runTypeCast( )

            }

    }

}

// 1. 클래스 계층 정의

**class** Animal {

    **func** makeSound() {

        print("🦠 Some generic animal sound")

    }

}

  

**class** Dog: Animal {

    **override** **func** makeSound() {

        print("🐶 Woof!")

    }

    **func** fetch() {

        print("🎾 Dog is fetching the ball.")

    }

}

  

**class** Cat: Animal {

    **override** **func** makeSound() {

        print("🐱 Meow!")

    }

    **func** scratch() {

        print("😼 Cat is scratching.")

    }

}

  

**func** runTypeCast (){

    // 2. 업캐스팅 (Sub → Super)

    **let** dog = Dog()

    **let** animal: Animal = dog       // implicit upcast

    animal.makeSound()             // 콘솔: "🐶 Woof!"

  

    //    명시적 업캐스팅

    **let** animal2 = dog **as** Animal

    animal2.makeSound()            // 콘솔: "🐶 Woof!"

  

  

    // 3. 다운캐스팅 (Super → Sub)

    // 3.1. 옵셔널 다운캐스트 (안전)

    **if** **let** dogAgain = animal **as**? Dog {

        dogAgain.fetch()           // 콘솔: "🎾 Dog is fetching the ball."

    } **else** {

        print("❌ animal을 Dog로 다운캐스팅 실패")

    }

  

    **if** **let** cat = animal **as**? Cat {

        cat.scratch()

    } **else** {

        print("❌ animal을 Cat로 다운캐스팅 실패")

    }

  

  

    // 4. 타입 체크

    **if** animal **is** Dog {

        print("✅ animal is a Dog")   // 이 줄이 출력됩니다

    }

    **if** animal **is** Cat {

        print("✅ animal is a Cat")

    }

  

}

```






# JSON 데이터 디코딩 시 TypeCasting이 필요한 경우는 어떤 경우인가? 

# JSON 파일 ?
- 쉽게 설명하자면, 안에 뭐가 있을지 모르고 받는 상자같은 것. 그래서 타입을 확인하고 꺼내야(decoding) 한다. 

## 1. JSONSerialization 은  → Any 로 받을 때
- `JSONSerialization` 은 무조건 JSON을 `Any` 로 받음.
- 투명하지 않은 상자에 뭐가 들어 있는데? 뭐가 들어있는지 모르는 상태. 


```Swift 
let raw = try JSONSerialization.jsonObject(with: data)
guard let dict = raw as? [String: Any] else { return }

//꺼내면서 확인해야 함
let name = dict["name"]  as? String  
let age  = dict["age"]   as? Int     
```

## 2. Coderable 을 써도 서버가 약속을 어길 때 (에러)
- Coderble은? 보통 구조체에 딱 맞게 매핑되는데, 서버 오류가 가끔 생길 때가 있고, 이럴 땐 실제로 매핑해줘야 함.  
```Swift
struct User: Codable {
  let id: Int

  init(from decoder: Decoder) throws {
  //decoder.container(keyedBy:) 로 JSON 객체에 들어 있는 키("id")별 데이터를 꺼낼 수 있는 상자를 만듭니다.
    let c = try decoder.container(keyedBy: CodingKeys.self)
    // 숫자 디코딩 시도 → 실패하면 문자열 디코딩 후 Int 변환
    // 성공하면 id = intValue 로 끝!
    if let i = try? c.decode(Int.self, forKey: .id) {
      id = i
    }
    else if let s = try? c.decode(String.self, forKey: .id),
            let i2 = Int(s) {
            //문자열이면? "123"이면 StringValue에 "123"을 넣고, Int("123")으로 숫자를 바꿔서 id = 123 으로 만듦.
      id = i2
    }
    else {
    //만약 문자열에 숫자가 아예 없거나 ("가나다") bool 같이 아예 다른 타입이면 throw 에러
      throw DecodingError.dataCorruptedError(...)
    }
  }
}
```


## 3. 배열이나 Dictionary 에 여러 타입이 섞여 있을 때. 
- 예를 들어 `["hello", 123, ["flag": true] ]`
- 여기서는 꺼낼 때마다 물어봐줘야 함. 
```Swift 
let arr: [Any] = try JSONSerialization.jsonObject(with: data) as! [Any]
for item in arr {
  if let s = item as? String {
    print("문자열:", s)
  }
  else if let n = item as? Int {
    print("숫자:", n)
  }
  else if let d = item as? [String:Any] {
    print("딕셔너리:", d)
  }
}
```


## Keywords
+ [[TypeCasting]]
- [[Codable]]
- [[Dictionary]]
- [[JSON]]
## References
- [https://zeddios.tistory.com/265](https://zeddios.tistory.com/265) [ZeddiOS:티스토리]
- https://babbab2.tistory.com/127

## 작성자
- #Zhen 