## Error Handling

### Swift에서의 예외 처리 (Error Handling)

- Swift에서 예외 처리는 프로그램 실행 중 발생할 수 있는 오류 조건에 응답하고, 복구하는 중요한 
프로세스입니다.
- Swift는 런타임에 오류를 던지고, 포착하고, 전파하는 기능을 제공하여 앱 개발을 지원합니다.

<aside>
💡

예외 처리가 필요한 이유

1. 프로그램 안전성 향상: 예상치 못한 상황에서도 앱이 완전히 중단되지 않고 계속 실행 될 수 
있습니다.
2. 사용자 경험 개선: 오류가 발생했을 때 사용자에게 적절한 피드백을 제공할 수 있습니다.
3. 디버깅 용이성: 오류의 원인과 발생 지점을 명확히 파악할 수 있어 문제 해결이 용이합니다.
4. 코드 품질 향상: 예외적 상황을 명시적으로 처리함으로써 코드의 견고성과 가독성이 높아집니다.
</aside>

→ 최선의 노력에도 불구하고 현대 앱의 복잡성과 동적입력에 대한 의존성으로인해 오류는 불가피하기에, 이를 보다 효과적으로 처리하는 것이 중요합니다.

## 에러 정의하기

- Swift에서는 ‘Error’ 프로토콜을 준수하는 타입으로 오류를 표현합니다. *일반적으로 열거형(enum)사용.*

```swift
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}
```

= 자판기에서 발생할 수 있는 세 가지 오류 상황을 정의했습니다.

- `insufficientFunds(coinsNeeded: Int)` 처럼 연관 값 (associated value)을 포함하여 추가 정보를 제공 할 수 있습니다.

## 에러 처리 메커니즘

### `throws` 와 `throw` 키워드

- 함수 자체가 오류를 발생시킬 수 있음을 나타내려면 `throws` 키워드를 사용합니다.
- 실제로 오류를 발생시키려면 함수 내에서 `throw` 키워드를 사용합니다.

```swift
func vend(_ itemNamed: String) throws -> Item {

    guard let item = inventory[itmeNamed] else { // 자판기에 없는 상품이 들어오면 에러
        throw VendingMachineError.invalidSelection
    }
    
    guard item.count > 0 else { // 자판기에 재고
        throw vendingMachineError.outOfStock
    }
    
    guard item.price <= coinsDeposited else { // 넣은 돈 가격보다 적으면 에러
        throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDeposited)
    }
    
    return item
}
```

## `do- catch` 구문

- `do-catch` 구문을 사용하여 발생할 수 있는 오류를 처리할 수 있습니다.

```swift
do {
    try 오류를 발생시킬 수 있는 함수()
} catch 오류 패턴1 {
    // 오류 패턴1 처리
} catch 오류 패턴2 where 조건 {
    // 조건을 만족하는 오류 패턴2 처리
} catch {
    // 기타 모든 오류 처리
}
```

## 특정 오류 타입을 처리하는 예:

```swift
do {
    let machine = try vend(itemNamed: "Candy Bar")
    // 성공적으로 처리 된 경우
} catch VendingMachineError.invalidSelection {
        print("유효하지 않은 선택입니다.")
} catch VendingMachineError.outOfStock {
        print("재고가 부족합니다.")
} catch VendingMachineError.insufficientFunds(let coinsNeeded) {
        print("금액이 \(coinsNeeded)원 부족합니다.")
} catch {
        print("예상치 못한 오류: \(error)")
}
```

## `try` 키워드와 변형

### 오류를 발생시킬 수 있는 함수를 호출할 때는 `try` 키워드를 사용해야며, 
Swfit는 세 가지 형태의 `try`를 제공합니다.

- `try` : 기본 형태로, do-catch 구문 내에서 사용해야 합니다.
- `try?` : 함수가 오류를 발생시키면 nil을 반환하고, 성공하면 옵셔널 값을 반환합니다.
- `try!` : 함수가 오류를 발생시키지 않을 것이라고 확신할 때 사용합니다.
        > 오류가 발생하면 런타임 오류로 앱이 종료됩니다.

```swift
do {
    let item = try vend(itemNamed: "Candy Bar")
    // 성공적으로 상품을 얻었을 때의 처리
} catch {
    // 오류 처리
}

// try? 사용 예
if let item = try? vend(itemNamed: "Candy Bar") {
    // 성공적으로 상품을 얻었을 때의 처리
} else {
    // 오류 처리
}

// try! 사용 예
let item = try! vend(itemNamed: "Candy Bar")
```

- 파일열 열고 쓰는 과정에서 오류 처리를 하는 예제입니다 :

```swift
enum FileError: Error {
    case fileNotFound
    case permissionDenied
}

func writeToFile(named filename: String) throws {
    // 파일 읽기
    if fileNotFound {
        throw FileError.fileNotFound
    }
    
    // 파일 쓰기
    if permissionsDenied {
        throw FileError.permissionDenied
    }
    
    // 정상적 파일 처리
    
}

// 사용 예
do {
    try writeToFile(named: "example.txt")
    print("파일 작성 성공")
} catch FileError.fileNotFound {
        print("파일을 찾을 수 없습니다.")
} catch FileError.permissionDenied {
        print("파일에 쓰기 권한이 없습니다.")
} catch {
        print("알 수 없는 오류: \(error)")
}
```

## 실제 예제: 네트워킹에서의 에러 처리

- 네트워크 요청 시 발생할 수 있는 다양한 오류를 처리하는 예:

```swift
enum NetworkError: Error {
    case invalidURL
    case noData
    case invalidResponse
    case decodingError
}

func fetchData(from urlString: String) throws -> Data {
    guard let url = URL(string: urlString) else {
        throw NetworkError.invalidURL
    }
    
    // 네트워크 요청 로직...
    if response.statusCode != 200 {
        throw NetworkError.invalidResponse
    }
    
    guard let data = data else {
        throw NetworkError.noData
    }
    
    return data
    
}

// 사용
do {
    let data = try fetchData(from: "https://api.example.com/data")
    let decodeData = try JSONDecoder().decode(MyModel.self, from: data)
    // 데이터 처리
} catch NetworkError.invalidURL {
        print("URL 이 유효하지 않습니다.")
} catch NetworkError.invalidResponse {
        print("서버 응답이 유효하지 않습니다.")
} catch NetworkError.noData {
        print("데이터가 없습니다.")
} catch NetworkError.decodingError {
        print("디코딩 오류입니다.")
} catch {
        print("알 수 없는 오류: \(error)")
}

```

<aside>
💡

결론

- Swift의 예외 처리 시스템은 견고하고 안전한 코드를 작성하는 데 필수적인 도구입니다.
- `throws`. `throw`, `try`, `do-catch` 와 같은 키워드를 사용하여 오류를 효과적으로 정의하고, 발생시키고, 처리 할 수 있습니다.
- 적절한 에러 처리는 앱의 안정성을 향상시키고 사용자에게 더 나은 경험을 제공합니다.
</aside>

## ‼️추가로 궁금했던 부분

### why enum?

- ??
    
    ### Swift의 에러 처리 규칙
    
    - Swift에서 throw하려면 에러는 반드시 Error 프로토콜을 채택한 타입이어야 한다.
    
    ```swift
    enum MyError: Error {
        case somethingWrong
    }
    ```
    
    ❗️꼭 enum을 안 써도 되긴 하지만 추천되는 방식은 아니다
    
    ### 그럼 왜 enum을 쓸까?
    
    1. 에러는 대부분 “케이스” 로 구분됨
        - 네트워크 오류
        - 파일 없음
        - 권한 없음
        - 입력값 오류 등
    
    >> 이런 여러가지 상태(case)가 있을 땐 enum이 가장 자연스럽다.
    
    ```swift
    enum LoginError: Error {
            case invalidUsername
            case invalidPassword
            case networkFailure
    }
    ```
    
    1. 패턴 매칭이 쉽다 (switch, catch)
    
    ```swift
    enum LoginError: Error {
            case invalidUsername
            case invalidPassword
            case networkFailure
    }
    
    func login() throws {
            throw LoginError.invalidUsername
    }
    
    do {
            try login()
    } catch {
            switch error {
            case LoginError.invalidUsernamse:
                    print("사용자 이름이 틀립니다.")
            
            case LoginError.invalidPassword:
                    print("비밀번호가 틀립니다.")
                    
            case LoginError.networkFailure:
                    print("네트워크 오류로 정보를 불러오지 못했습니다.")
                    
            default:
                    print("알 수 없는 오류입니다. 고객센터에 문의하세요.")
            }
    }
    ```
    
    1. 연관값(associated values)으로 추가 정보 전달 가능
    
    ```swift
    enum PurcahseError: Error {
            case insufficientFunds(coinsNeeded: Int)
    }
    ```
    
    - 케이스에 데이터를 붙일 수 있다.
    - `.insufficientFunds(200)`  처럼 상태 + 데이터를 한번에 전달 가능하다.
    

### 상황별 조건 분기와 예외 흐름처리?

- if, guard, guard let?
    
    
    | 쓰는 문법 | 언제 쓰는가 |
    | --- | --- |
    | `if` | 조건이 다양하거나, 단순 분기만 할 때 |
    | `guard` | 실패 조건에서 조기 탈출(Early Exit) 하고, 이후 코드를 깔끔하게 진행할 때 |
    | `guard let` | 옵셔널 값을 안전하게 꺼내야 할 때 |
    1. `if` 를 쓴 경우
    
    ```swift
    if fileNotFound {
            throw FileError.fileNotFound
    }
    
    if permissionDenied {
            throw FileError.permissionDenied
    }
    ```
    
    - 옵셔널이 없으며, 단순한 불리언 조건 검사만 하고 있다.
    - 조건이 만족되면 에러를 `throw`하고, 아니면 계속 아래 코드로 진행
    
    👉 이럴 땐 `if`가 간단하고 명확하다.
    
    1. `guard let` 을 쓴 경우
    
    ```swift
    guard let item = inventory[name] else {
            throw VendingMachineError.invalidSelection
    }
    ```
    
    - `inventory[name]` 이 옵셔널 값이고 > 키가 없으면 nil 리턴
    - 그래서 `guard let`으로 옵셔널 바인딩해서 안전하게 꺼낸 후, 아래에서 계속 item을 사용해 나감
    
    👉 `guard let` 은 옵셔널 처리 + 조기 탈출 이라는 두 가지 목적을 동시에 만족한다.
    
    1. `guard` 만 쓴 경우
    
    ```swift
    guard item.count > 0 else {
            throw VendingMachineError.outOfStock
    }
    ```
    
    - 여긴 옵셔널이 아니라 그냥 Int 타입이라서 `guard let`이 아니라 `guard` 만 쓴다.
    - 조건이 틀리면 빠져나가고, 맞으면 코드 계속 진행.
    
    👉  `guard` 는 “조건 실패 시 나가고 싶다” 라는 명확한 의도 표현.
    
    <aside>
    💡
    
    - `guard` 는 실패 조건을 먼저 처리하고 코드 흐름을 납작하게(flat 하게) 만들어줌
    - `if` 는 여러 경우를 처리하거나 복잡한 분기가 필요한 경우에 유용
    - `guard let` 은 옵셔널을 안전하게 벗겨내고, 이후 코드에서 안심하고 사용할 수 있도록 해줌.
    </aside>
    
    👉 ” 이 값이 없으면 이 함수 그냥 탈출해버려! “ → `guard let`
    
    👉 ” 이 조건이 아니면 탈출해! “ → `guard`
    
    👉 ” 여러 조건을 각각 다르게 처리해줘야 해 “ → `if`
