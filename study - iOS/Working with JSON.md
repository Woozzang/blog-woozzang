# JSON 다루기



<u>**JSON =  JavaScript Object Notation**</u>



### 🤔사전 지식 -  JSON 이란?

단순하게 데이터를 표현하는 방법 중 하나 (통신 방법도 아니고, 프로그래밍 문법도 아니다. 포맷이다 !!!!)

서버와 클라이언트 간의 데이터 교환에서 일반적으로 많이 사용된다

*클라이언트가 API Request 를 보내면 서버가 응답으로 JSON 데이터를 보내준다.*



**JSON의 포맷은 자바스크립트 객체 표기법을 따른다**

1. `key` - `value` 쌍을 이루어 표현하며, key는 문자열이다. 문자열은  `" "`  쌍따옴표를 사용하여 표기한다.		

2. { } 와 내부에 `key-value` 로 구성된다. 즉 Swift 딕셔너리와 유사한 구조이다.

3. JSON형식에서는 null, number, string, array, object, boolean을 사용할 수 있다.

   

다시 한번 말하지만 JSON은 데이터 표현 방식일 뿐이며, 텍스트형식의 데이터 이다.

그렇기 때문에 가독성이 좋으며, 단순한 텍스트이기 때문에 특정 언어에 종속되지 않는다. 



대부분의 프로그래밍 언어에서 JSON 포맷 데이터를 핸들링 할 수 있는 라이브러리를 제공한다



### 🤔사전 지식 - 기존에 사용하던 XML은?

특징 데이터 값 양쪽으로 태그가 있다. 태그로 표현되는 데이터

(HTML을 근본으로 했기 때문인데, 그 태그를 줄인다 해도 최소한 표현하려면 양쪽에 몇글자씩이 있어야 한다.)

<img src="/Users/woochan/Library/Application Support/typora-user-images/Screen Shot 2021-05-06 at 4.31.19 PM.png" alt="Screen Shot 2021-05-06 at 4.31.19 PM" style="zoom: 67%;" />



참고자료 : https://velog.io/@surim014/JSON



## 본론으로 돌아와서...

보통 `struct` 로 모델을 표현하는데, `struct`  인스턴스를  JSON 으로 바꾸는 것을 encoding, 

JSON 을 struct 인스턴스로 바꾸는 것을  decoding 이라고 한다.



 Encoding  과 Decoding 을 하는 주체는 `JSONEncoder` 와 `JSONDecoder` 타입의 오브젝트이다.

 Encoding  과 Decoding 이 가능하려면 대상 struct 는`Encodable`,` Decodable` 프로토콜을 채택하고 있어야 한다.

만약 Encodable 과 Decodable 을 모두 채택한다면, Codable 프로토콜을 채택한다고 명시하면 된다.



```swift
typealias Codable = Decodable & Encodable // 실제로 이렇게 구현되어 있음 😉
```





```swift
let encoder = JSONEncoder() // 인스턴스에 저장되어 있는 데이터를 JSON 형식으로 Encoding 한다.
encoder.outFormatting = .prettyPrinted
encoder.keyEncodingStrategy = .converToSnakeCase

do {
let jsonData = try encoer.encode(someStruct)
  
  if let str = String(data: jsonData, encoding: .utf8) {
    print( str ) 
  }
} catch {
  print(error)
}
```



encode() 메서드는 Throwing Method 이므로 try 로 처리해준다.

반환 값은 바이너리 형태의 JSON 이다.

바이너리 형태의 문자열은 인간이 읽을 수 없기 때문에 인코딩이 필요하다.

눈으로 확인하려면 String(data:encoding) 생성자로 문자열 인스턴스로 생성해 출력해볼 수 있다.

또한 `outFormatting` 속성을 `.prettyPrinted` 로 변경해 key-value 간 개행을 추가하여 가독성을 높일 수 있다.



## JSON Encoding

 Encoding은 Struct 인스턴스에서 JSON 데이터로 변환하는 과정이다.



### 속성 이름과 key 이름 매칭

JSON 에서 키 네이밍 컨벤션은 `snake_case` 이다.

하지만 Swift 에서는 `lowerCamelCase` 를 사용하기 때문에 수정없이 Encoding 을 진행한다면 key name 이 lowerCamelCase 로 들어가기 때문에 호환성이 떨어질 것이다.

이때는 `keyEncodingStrategy`  속성을 변경하여 대응할 수 있다.

```swift
encoder.keyEncodignStrategy = .convertToSnakeCase
```



### 날짜 형식 매칭

Swift 는 Date 타입의 속성을 JSON 으로 인코딩 할때, Double 값으로 인코딩한다.

이 값은 2001년 1월 1일을 시작점으로 떨어진 만큼의 시간을 표현하는데, 초 단위이다.

가독성이 매우 떨어지므로 많이 사용하는 iso8601 형식을 따른 문자열로 인코딩하게 바꿔줄 수 있다.

인코더 오브젝트의 `dateEncodingStrategy` 속성을 변경하면 된다.

```swift
encoder.dateEncodingStrategy = .iso8601
```



그런데 이 형식을 그대로 따르지 않는 API 들이 있다. (예: Kakao REST API)

이 경우에 해당 Key 혹은 속성은  Encoding 혹은 Decoding 에 실패하게 된다.

이때는 DateFormatter 인스턴스를 직접 만들어서 encoder 에 전달해야한다.

```
// 1. Custom DateFormatter 를 생성한다.
var dateFormatter: DateFormatter = {
	var formatter = DateFormatter()
	formatter.dateFormat = "MMMyyyy" // dateFormat 형식문자열
}()

// 2. encoder 오브젝트에 .formmated case의 연관값으로 전달한다.
encoder.dateEncodingStrategy = .formmatted(dateFormatter)

```



🍯 그런데 원하는 DateFormat  형식문자열을 직접 작성하는 것이 어렵게 느껴질수 있다.

이 때는 https://nsdateformatter.com 에서 원하는 형식문자열을  간편하게 생성할 수 있다.

<img src="/Users/woochan/Library/Application Support/typora-user-images/Screen Shot 2021-05-07 at 5.18.13 PM.png" alt="Screen Shot 2021-05-07 at 5.18.13 PM" style="zoom:67%;" />



## JSON Decoding

Decoding은 서버로부터 전달받은 JSON 데이터를 Struct 인스턴스로 변환하는 과정이다.



```swift
guard let jsonData = jsonStr.data(using: .utf8) else {
   fatalError()
}

//
let decoder = JSONDecoder()
let result = try? decoder.decode(Person.self, from: jsonData)
```



### Struct 속성의 타입

일반 타입이라면  decoding 실패 시 런타임 오류가 발생한다

옵셔널 타입이라면 decoding 실패 시 nil 로 초기화된다.



### JSON key 와 struct 속성이 다른 경우

만약 키 네임과 속성 이름의 Naming convention 만 다르다면 encoding 때와 같은 방식으로 해결해주면 된다.

```swift
decoder.keyDecodingStrategy = .convertFromSnakeCase
```



하지만 이름이 아예 다른경우는 custom key mappping 을 해주어야 한다.

```swift
struct iPhone: Codable {
   var SerialNumber: String
   var Storage: Int
   var generation: Int
   var ramSize: Int
   
  enum CodingKeys: String, CodingKey {
    case SerialNumber
    case Storage
    case generation = "model_number"
    case ramSize
  }
}
```

예를 들어 JSON 데이터에서 `model_number` 키에 저장된 값을 `generation` 속성에 받아오고 싶을 때, custom Key mapping 을 하지 않으면 decoding error 가 발생할 것이다.  Int  타입인 `generation` 에 매핑된 값이 없기 때문이다.



위의 코드와 같이 형식 내에 enum 을 선언하고, raw Value 로 String 로 선언하고, CodingKey 프로토콜을 채택한다음 매핑하고 싶은 case 만 rawValue 설정을 해주면 해결된다. rawValue 에 들어가는 값은 매핑하고 싶은  JSON 의 key name 이다.



## 🍯 Custom Encoding & Custom Decoding

보통 인코딩과 디코딩에 제약을 넣고 싶을때 커스텀으로 구현한다.

Encodable & Decodable 프로토콜의 메서드를 직접 구현하면 된다.



🥳 원리를 제대로 이해했으니, 커스텀이 필요할 때 내용을 더 추가하자!



💡 **기본 원리:** protocol method 를 직접 구현하고, container 의 unkeyed-data 를 적절하게 encode/decode 하자!!!!



Encodable 의 경우 encode(to encoder:)  Decodable 의 경우 init(from decoder:) 메서드를 직접 구현하면 된다.

```swift
  func encode(to encoder: Encoder) throws {
    <#code#>
  }
  
  init(from decoder: Decoder) throws {
    <#code#>
  }
```



우선 커스텀 인코딩을 하려면 CodingKeys [ 컨벤션이지만 임의의 이름으로, CodingKey  프로토콜을 타입] 가 생성되어야 하기 때문에, 형식 내부에서  enum 으로  CodingKey 프로토콜을 채택하여 선언하여야한다.

```swift
struct Person: Codable {
   var firstName: String
   var lastName: String
   var age: Int
   var address: String?
    
  enum CodingKeys: String, CodingKey {
    case firstName
    case lastName
    case age
    case address
  }
```



왜? container(keyedBy:) 의 파라미터로 CodingKey Protocol 타입을 받는다.

![Screen Shot 2021-05-07 at 6.37.36 PM](/Users/woochan/Library/Application Support/typora-user-images/Screen Shot 2021-05-07 at 6.37.36 PM.png)

```swift
  func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)
    
    try container.encode(firstName, forKey: .firstName)
    
    try container.encode(lastName, forKey: .lastName)
    
    guard (30...60).contains(age) else { // 제약을 추가하고 조건에 맞지 않는다면 에러를 던진다.
      throw DecodingError.invalidRange
    }
    
    try container.encodeIfPresent(address, forKey: .address)
  }
```



컨테이너의 정의 : " an container appropriate for holding multiple <u>unkeyed values.</u>"

그리고 `encode(_, forKey:)` 메서드로 container 내부의 데이터(unkeyed values)들을 인코딩하여 키와 매핑해준다.

encode 는  `throws` 키워드에서 볼수 있듯이 Throwing Method 로, 내부 블럭에서 에러를 던질 수 있다.

위 코드에서는 `age` 속성의 값이 30~60 사이가 아니라면 미리 정의된 에러를 던지고 있다.