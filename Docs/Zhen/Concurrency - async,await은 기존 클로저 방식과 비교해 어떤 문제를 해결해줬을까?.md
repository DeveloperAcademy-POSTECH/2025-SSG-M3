>[!question]
>GQ1. async/await은 기존 클로저 방식과 비교해 어떤 문제를 해결해줬을까?

## Description
- async/await 은 뭘까?
- 기존 클로저의 어떤 문제를 해결한 걸까? 

## 주요 기능
### async/await 은 뭘까
- async 는 이 함수가 비동기적으로 작업을 처리함을 의미함 ` () async -> Image
- 그 함수 안에 특정 명령에 대해서 await 을 사용해서 비동기적 작업이라고 명시적으로 알려줌

```
func process() async -> Image {  
let result = await loadWebResource("dataSource")  
return result  
}  
let image = process()


//1 : process() 실행  
//2 : loadWebResource("dataSource")로 데이터 받아오는 요청함
//3 : 받아온 데이터를 result에 저장함  
//4 : result를 return함
```

### 기존 클로저의 어떤 문제를 해결한 걸까? 
- Async programming with explicit callbacks (also called completion handlers) has many problems, which we’ll explore below.
- #### Problem 1: Pyramid of doom
```
func processImageData1(completionBlock: (_ result: Image) -> Void) {
    loadWebResource("dataprofile.txt") { dataResource in
        loadWebResource("imagedata.dat") { imageResource in
            decodeImage(dataResource, imageResource) { imageTmp in
                dewarpAndCleanupImage(imageTmp) { imageResult in
                    completionBlock(imageResult)
                }
            }
        }
    }
}

processImageData1 { image in
    display(image)
}
```
일명 "들여쓰기의 계단" 현상. 
이 현상이 발생한 경우, 무슨 일이 벌어지고 있는지 트랙을 따라가기 힘들고, 이 클로저 스택은 다음에 이어서 이야기할 2차 효과를 야기할 수 있음. 

- Problem 2: Error handling
```
// (2a) Using a `guard` statement for each callback:
func processImageData2a(completionBlock: (_ result: Image?, _ error: Error?) -> Void) {
    loadWebResource("dataprofile.txt") { dataResource, error in
        guard let dataResource = dataResource else {
            completionBlock(nil, error)
            return
        }
        loadWebResource("imagedata.dat") { imageResource, error in
            guard let imageResource = imageResource else {
                completionBlock(nil, error)
                return
            }
            decodeImage(dataResource, imageResource) { imageTmp, error in
                guard let imageTmp = imageTmp else {
                    completionBlock(nil, error)
                    return
                }
                dewarpAndCleanupImage(imageTmp) { imageResult, error in
                    guard let imageResult = imageResult else {
                        completionBlock(nil, error)
                        return
                    }
                    completionBlock(imageResult)
                }
            }
        }
    }
}

processImageData2a { image, error in
    guard let image = image else {
        display("No image today", error)
        return
    }
    display(image)
}
```
이 경우 
```
// (2b) Using a `do-catch` statement for each callback:
func processImageData2b(completionBlock: (Result<Image, Error>) -> Void) {
    loadWebResource("dataprofile.txt") { dataResourceResult in
        do {
            let dataResource = try dataResourceResult.get()
            loadWebResource("imagedata.dat") { imageResourceResult in
                do {
                    let imageResource = try imageResourceResult.get()
                    decodeImage(dataResource, imageResource) { imageTmpResult in
                        do {
                            let imageTmp = try imageTmpResult.get()
                            dewarpAndCleanupImage(imageTmp) { imageResult in
                                completionBlock(imageResult)
                            }
                        } catch {
                            completionBlock(.failure(error))
                        }
                    }
                } catch {
                    completionBlock(.failure(error))
                }
            }
        } catch {
            completionBlock(.failure(error))
        }
    }
}

processImageData2b { result in
    do {
        let image = try result.get()
        display(image)
    } catch {
        display("No image today", error)
    }
}
```

이렇게됨... 클로저 중첩이 계속 있음 
- #### Problem 3: Conditional execution is hard and error-prone
조건부 실행이 어려워지고, 에러가 발생하기 쉬움 

```
func processImageData3(recipient: Person, completionBlock: (_ result: Image) -> Void) {
    // 1) 먼저 ‘swizzle’라는 이름의 클로저(closure)를 미리 만든다.
    let swizzle: (_ contents: Image) -> Void = { image in
        // …이 안에서 최종적으로 completionBlock(image)를 호출한다고 치자
    }

    // 2) 만약 recipient(사람)가 프로필 사진을 이미 가지고 있으면
    if recipient.hasProfilePicture {
        //   즉시 swizzle에 프로필 사진을 넘겨서 처리를 시작한다.
        swizzle(recipient.profilePicture)
    } else {
        // 3) 그렇지 않으면, decodeImage라는 비동기 함수(예: 네트워크나 디스크에서 이미지 디코딩)를 호출하고,
        //    비동기 완료 시점에 swizzle(image)를 호출한다.
        decodeImage { image in
            swizzle(image)
        }
    }
}

```

그니까 결국 후반부가 전반부보다 먼저 실행되어야만 하는 코드가 완성되게 된다는 뜻임.. 
(let swizzle ~ 여기가 제일 먼저라고 생각되지만, 실제로 따져보면 if/else 구문을 먼저 실행하고, 그 결과에 따라서 1번으로 돌아간다는 뜻)
C(=swizzle)부터 정의해놓고 나중에 A나 B를 거쳐 C를 실행하는 구조가 된다는 의미.
이게 왜 문제냐면? 
가독성이 떨어지고 콜백 지옥이 생기는 것 + swizzle 클로저 안에서 외부 변수를 사용할 때 순환 참조가 일어나지 않도록 `[weak self]` 또는 `[weak recipient]` 를 신경써야 하는데, 이 클로저를 바깥에서 계속 재사용하다보면 클로저 캡처(적절한 위치의 변수를 가져오는거)와 메모리 누수 문제가 생기게 됨. 
케이스가 여러개면 이게 기하급수적으로 더 늘어남... 아무튼 비효율적이고 너무 복잡하다는 뜻. 

이걸 async/await 으로 바꾸면 다음과 같음 

```
func processImageData(recipient: Person) async throws -> Image {
    // 1) recipient에 프로필 사진이 있으면 그대로 리턴
    if recipient.hasProfilePicture {
        return recipient.profilePicture
    }
    // 2) 없으면 비동기로 이미지 디코딩
    let image = try await decodeImage()
    // 3) 디코딩된 이미지에 swizzle 처리(가공)한 뒤 리턴
    return swizzled(image)
}

// 호출 예시
Task {
    do {
        let finalImage = try await processImageData(recipient: somePerson)
        // completionBlock처럼 최종 결과를 처리
    } catch {
        // 에러 처리
    }
}

```

똑같이 비동기함수를 호출하는 방식인데, 동기 코드처럼 깔끔하고 순차적으로 진행되는 것 처럼 보임. 
if - else 구문을 통해 분기를 정해서 > 준비된 이미지를 swizzle 에 리턴하는 순서가 드러나게 되고, 가독성이 높아짐 + 캡처 신경 쓸 필요 없음. 

## Keywords
+ [[병렬 처리(Swift-Concurrency)]]

## References
- Blog
	- https://medium.com/hcleedev/swift-async와-await-등장-배경과-사용법-1d21003fe57f
- Github
	- https://github.com/swiftlang/swift-evolution/blob/main/proposals/0296-async-await.md

## 작성자
	#Zhen 