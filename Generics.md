# Generics

## This chapter covers
- How and when to write generic code
- Understanding how to reason about generics
- Constraining generics with one or more protocols
- Making use of the Equatable, Comparable and Hashable protocols
- Creating highly reusable types
- Understanding how subclasses work with generics

## The benefits of generics

제네릭은 스위프트에서 자주 등장합니다.
다형성을 위해서는 제네릭과 프로토콜은 필수적입니다.

제네릭 없이 여러 타입을 대응하는 함수를 만들기 위해서는 중복되는 코드가 씁니다.
제네릭을 사용한다면 여러 타입을 대응하는 하나의 함수를 만들 수 있습니다.

물론 Any 타입을 사용해 여러 타입에 대응할 수 있지만, Any를 사용하면 런타임에 Any를 특정 타입(String, Int 등)으로 다운 캐스팅해야 합니다.
Any 타입과 함께 AnyObject 타입도 존재합니다. Any 타입은 값 타입, 참조 타입 모두 저장 가능하지만 AnyObject 타입은 클래스 타입만 저장할 수 있습니다.
하지만 Any 타입과 AnyObject 타입 모두 런타임에 타입이 결정됩니다. 
또한 다운 캐스팅도 필수적입니다. 이는 보일러플레이트 코드로 이어집니다.

Any 타입이나 AnyObject 타입보다는 제네릭을 사용하도록 합시다.

제네릭을 사용해 컴파일 타입에 다형성을 이룰 수 있습니다.
아래 코드는 제네릭 없이 구현하던 함수를 제네릭 함수로 고친 코드입니다.

```swift
// 제네릭 X
func firstLast(array: [Int]) -> (Int, Int) {
  return (array[0], array[array.count - 1])
}

// 제네릭 O
func firstLast<T>(array:[T]) -> (T, T) {
  return (array[0], array[array.count -1])
}
```

제네릭 함수를 만들 때는 함수명 뒤에 <T>를 붙이고 함수 내부에서 사용되는 타입도 T로 바꾸면 됩니다.
물론 T 말고도 Wrapped, U, V 등 자유롭게 사용할 수 있습니다.

하지만 T로 제네릭을 구현했다면 함수 내부에서 T 타입을 가진 변수는 모두 동일한 데이터 타입을 따르게 됩니다.
제네릭에 Int, String 등과 함께 커스텀 데이터 타입도 넘길 수 있습니다.

제네릭으로 타입에 제약받지 않는 범용 코드를 만들고 코드 중복을 피할 수 있습니다.

결과적으로 제네릭 함수는 하나의 함수로 여러 타입에 대응할 수 있습니다. 
Any를 사용했다면 추가적인 다운 캐스팅을 해야 하지만, 제네릭으로 컴파일 타임에 모든 타입을 선언할 수 있습니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/e1ac17ec-7377-4599-a458-8efa25af6219)

처음부터 제네릭 함수를 구현하는 건 어려울 수 있습니다. 
먼저 제네릭 없이 함수를 구현해보고 이후 제네릭 함수로 고치는 방식이 더 쉽습니다.

유용한 제네릭이지만 주의해야 할 부분도 있습니다.
아래 코드로 확인해 봅시다.

```swift
func illegalWrap<T>(value: T) -> [Int] {
  return [value]
}
```

illegalWrap 함수는 제네릭 타입의 value를 입력 받고 [Int]를 리턴하고 있습니다.
illegalWrap 함수의 제네릭을 Int로 특정해 리턴하고 있기 때문에 컴파일 에러가 발생합니다. 

제네릭은 타입을 특정할 수 없습니다!
당연하게 생각되지만 주의해야 합니다.

아래 코드로 더 확인해 봅시다.

```swift
// 컴파일 에러
func wrap<T> (value: Int, secondValue: T) -> ([Int], U) {
  return ([value], secondValue)
}

// 컴파일 가능
func wrap<T>(value: Int, secondValue: T) -> ([Int], T) {
  return ([value], secondValue)
}

// 컴파일 에러
func wrap(value: Int, secondValue: T) -> ([Int], T) {
  return ([value], secondValue)
}

// 컴파일 에러
func wrap<T>(value: Int, secondValue: T) -> ([Int], Int) {
  return ([value], secondValue)
}

// 컴파일 가능
func wrap<T>(value: Int, secondValue: T) -> ([Int], Int)? {
  if let secondValue = secondValue as? Int {
    return ([value], secondValue)
  } else {
    return nil
  }
}
```

제네릭을 사용하면 컴파일러가 Value Witness Tables이라 불리는 메타 데이터를 활용하여 
제네릭 함수를 구체화하는 과정에서 구체적인 타입(Int 등)으로 치환된 코드를 컴파일러가 반복적으로 생성합니다.

결과적으로 제네릭을 사용하면 어떤 값을 다룰지 컴파일 타임에 알 수 있다는 장점이 있습니다.

## Constraining generics

지금까지 소개한 제네릭은 별다른 제약 조건이 없는 제네릭이었기 때문에 모든 타입에 대응할 수 있었습니다. 

하지만 별다른 제약 조건이 없는 제네릭으로는 많은 일을 할 수 없습니다.
오히려 프로토콜을 통해 제네릭에 제약을 걸면 더 유용히 제네릭을 쓸 수 있습니다.

아래 예시는 제네릭에 제약을 걸지 않았을 때 문제가 발생하는 상황을 살펴봅시다.
아래 lowest 함수는 입력 데이터 중 가장 작은 값을 리턴하는 제네릭 함수입니다.

```swift
// 아직 미완성된 lowest 함수입니다.
func lowest<T>(_ array: [T]) -> T? {
  let sortedArray = array.sorted { (lhs, rhs) -> Bool in
    return lhs < rhs  // 에러의 원인입니다.
  }
  return sortedArray.first
}

lowest([3, 1, 2]) // Optional(1)
lowest([40.2, 12.3, 99.9]) // Optional(12.3)
lowest(["a", "b", "c"]) // Optional("a")
```

위의 lowest 함수는 에러를 발생시킵니다.

lowest 함수에서 제네릭에 제약을 걸지 않았기 때문에 입력 array에 모든 타입이 들어올 수 있습니다.
하지만 lowest 함수 안에서 비교 연산(<)을 수행하기 때문에 비교 연산이 가능하지 않은 타입이 입력으로 들어올 가능성은 곧 에러의 원인으로 이어집니다.

비교 연산이 가능한 타입만 함수 입력으로 들어오도록 제네릭 제약을 걸어 에러를 해결할 수 있습니다!
우린 프로토콜을 통해 제네릭에 제약을 걸 수 있습니다.

그렇다면 비교 연산과 관련 있는 프로토콜은 어떤 게 있을까요?
Equatable 프로토콜과 Comparable 프로토콜이 대표적으로 비교 연산과 관련된 프로토콜입니다.

먼저 Equatable 프로토콜은 두 값이 같은지 확인하는 데 쓰입니다.
동일한 타입 간의 == 비교연산자를 제공합니다.
따라서 Equatable 프로토콜을 채택했을 때 static == 함수를 필수로 구현해야 합니다.
Equatable 프로토콜의 static == 함수를 통해 동일한 타입 간의 '같다'는 기준을 만들 수 있습니다.

아래 코드는 Equatable 프로토콜 코드입니다.

```swift
public protocol Equatable {
  static func == (lhs: Self, rhs: Self) -> Bool
}
```

그렇다면 Comparable 프로토콜은 어떨까요?

Comparable 프로토콜은 Equatabel을 채택하고 있습니다.
클래스가 Comparable 프로토콜을 채택할 경우 Equatable 프로토콜의 static == 함수의 구현 의무도 지게 됩니다.
Comparable 프로토콜은 Equatable 프로토콜과 마찬가지로 static 함수가 있지만 모든 static 함수 구현을 필수로 요구하지 않습니다.

하지만 적어도 하나의 static 함수는 구현해야 합니다.

```swift
public protocol Comparable: Equatable {
  static func < (lhs: Self, rhs: Self) -> Bool
  static func <= (lhs: Self, rhs: Self) -> Bool
  static func >= (lhs: Self, rhs: Self) -> Bool
  static func > (lhs: Self, rhs: Self) -> Bool
}
```

Comparable 프로토콜을 채택할 경우 static 함수 중 하나를 구현하면 나머지 static 함수들은 스위프트가 유추하기 때문에 추가적으로 구현할 필요가 없습니다.
Int, float, string은 기본적으로 Comparable을 따르는 타입이기 때문에 우리가 자연스럽게 값을 비교할 수 있는것 입니다.

위에서 lowest 함수의 제네릭을 Comparable 프로토콜로 제약을 거는 방식으로 함수 안의 비교 연산에서 발생하는 에러를 고쳐봅시다.

아래 코드는 lowest 함수의 제네릭을 Comparable 프로토콜로 제약한 코드입니다.
이제 lowest 함수의 입력은 Comparable 프로토콜을 따르는 타입이어야 합니다.

```swift
func lowest<T: Comparable>(_ array: [T]) -> T? {
  let sortedArray = array.sorted { (lhs, rhs) -> Bool in
    return lhs < rhs  // array 입력으로 Comparable을 따르는 타입만 입력으로 들어오기 때문에 비교 연산이 가능합니다.
  }
  return sortedArray.first
}

// lowest 간략한 버전
func lowest<T: Comparable>(_ array: [T]) -> T? {
  return array.sorted().first  // array 입력이 항상 Comparable 프로토콜을 따르기 때문에 내장 함수인 sorted()도 사용 가능합니다.
}
```

그렇다면 lowest 함수에 입력으로 들어올 커스텀 타입은 어떤게 있을까요?
물론 Comparable 프로토콜을 따르도록 구현된 Int, float, string도 가능하겠지만 Comparable 프로토콜을 따르는 커스텀 타입도 입력으로 들어올 수 있습니다.

아래 코드는 Comparable 프로토콜을 따르는 커스텀 타입인 RoyalRank 타입의 코드입니다.
위에서 말했듯이 Comparable 프로토콜이 지원하는 static 함수 중 하나를 구현하여 나머지 static 함수를 컴파일러가 유추 가능하기 때문에 직접 구현하지 않은 static 함수도 사용할 수 있습니다.

```swift
enum RoyalRank: Comparable {
  case emperor
  case king
  case duke

  static func <(lhs: RoyalRank, rhs: RoyalRank) -> Bool {
    switch (lhs, rhs) {
      case (king, emperor): return true
      case (duke, emperor): return true
      case (duke, king): return true
      default: return false
    }
  }
 }

let king = RoyalRank.king
let duke = RoyalRank.duke

duke < king // true
duke > king // false
duke == king // false

let ranks: [RoyalRank] = [.emperor, .king, .duke]
lowest(ranks)  // .duke
```

제네릭의 장점 중 하나는 아직 존재하지 않은 타입에도 대응할 수 있다는 점입니다.

모든 타입이 Comparable 프로토콜을 따르진 않습니다.
예를 들어 Bool 타입은 Comparable 프로토콜을 따르지 않습니다. 따라서 lowest 함수에 Bool 타입은 입력으로 들어갈 수 없습니다.

Comstraining a generic means trading flexibility for functionality. A constrained generic becomes more specialized but is less flexible.

## Multipule constraints

제네릭을 제약해 사용할 때 하나의 프로토콜 만으로는 부족한 경우가 있습니다. 

예를 들어 값을 비교하고 해당 값을 딕셔너리에 저장해야 한다면, 우리는 비교 기능(Comparable)과 딕셔너리에 저장하는 기능(Hashable) 모두 필요합니다.
이를 위해 함수의 입력은 Comparable 프로토콜과 Hashable 프로토콜을 모두 따르는 타입이어야 합니다.

먼저 Hashable 프로토콜을 살펴 봅시다. 아래 코드는 Hashable 프로토콜 코드입니다.

```swift
public protocol Hashable: Equatable {
  func hash(into hasher: inout Hasher) {
    // ...생략
  }
}
```

Hashable 프로토콜도 Equatable 프로토콜을 채택하고 있습니다.
따라서 Hashable 프로토콜을 채택한다면 Equatable 프로토콜의 static == 함수와 Hashable 프로토콜의 hash 함수를 모두 필수로 구현해야 합니다.

구조체와 열거형에서는 Equatable과 Hashable 프로토콜의 필수 구현 함수 중 하나를 구현하면 나머지 함수들을 컴파일러가 유추하지만 클래스에서는
유추하지 못하기 때문에 클래스의 경우 모든 함수를 구현해야 합니다.

Hashable 프로토콜의 hash 함수를 통해 정수 타입인 hash value로 불리는 값을 제공합니다.
Hashable 프로토콜은 어떤 경우 채택해야 할까요?

Set 또는 Dictionary의 Key로 Hashable을 준수하는 모든 타입을 사용할 수 있습니다. 
스위프트 Dictionary 선언부를 보면 Dictionary<KeyType, ValueType> 형태입니다.
여기서 KeyType은 반드시 Hashable 프로토콜을 따르는 Hashable 타입이어야 합니다.
Hashable 프로토콜이 제공하는 hash value는 그 자체로 유일하게 표현이 가능한 방법을 제공합니다.

물론 스위프트의 기본 타입(Int, Double, String, Bool, enum, sets 등)들은 Hashable 프로토콜을 채택하기 때문에 Dictionary의 KeyType으로 사용할 수 있습니다.
하지만 커스텀 타입의 경우 Hashable 프로토콜을 채택해야 Dictionary의 KeyType으로 사용할 수 있습니다.

심지어 Set은 Hashable 프로토콜을 채택한 타입만 들어갈 수 있습니다.

그렇다면 Hashable 프로토콜이 제공하는 hash value는 어떤 값일까요?

예를 들어 "Hedgehog"을 hash 함수에 입력하면 43092483과 같은 Int 타입의 hash value가 리턴됩니다.
같은 타입의 인스턴스 a와 b가 있으면 a == b이면 a.hashValue == b.hashValue 입니다.
Hash 함수는 동일한 입력에 동일한 출력을 보장합니다.
하지만 hash value가 같다고 해서 동일한 인스턴스는 아닐 수 있습니다.

또한 hash value는 프로그램의 실행에 따라 달라질 수 있습니다.
따라서 이후 실행에 사용할 hash value 값을 저장하지 않는 게 좋습니다.
과거 실행에 저장했던 hash value와 방금 실행한 코드의 hash value는 달라질 수 있기 때문입니다.

Array와 달리 Set과 Dictionary는 "순서"가 없습니다.
따라서 특정 원소를 찾으려할 때 Hashable 프로토콜이 제공하는 정수 Hash(=hashValue)가 있기 때문에 우리가 찾으려는 원소를 빠르게 찾을 수 있습니다.
여기서 우리는 Set과 Dictionary가 중복을 허용하지 않는 이유도 추측해볼 수 있습니다.
Hash 함수를 통해 원소에 접근하기 때문에 중복을 허용하지 않는것 입니다.

아래 코드와 같이 제네릭 타입을 하나 이상의 프로토콜로 제약할 수 있습니다.

```swift
func lowestOccurrences<T: Comparable & Hashable>(values: [T]) -> [T: Int] {
  //...생략
}
```

제네릭 제약을 여러 프로토콜로 할 때 where 구문을 사용하면 더 간략히 표현 가능합니다.
아래 코드로 확인해 봅시다.

```swift
func lowestOccurrences<T>(values: [T]) -> [T: Int] {
  where T: Comparable & Hashable {
  // ...생략
}
```

## Creating a generic type

지금까지는 제네릭 함수만 살펴보았지만, 제네릭 타입도 만들 수 있습니다.
옵셔널 또한 제네릭 타입으로 구현되어 있습니다.
아래 코드를 살펴봅시다.

```swift
public enum Optional<Wrapped> {
  case none
  case some(Wrapped)
}
```

옵셔널과 배열 등은 모두 제네릭 타입으로 구현되어 있습니다.
커스텀 타입 또한 제네릭 타입으로 만들 수 있습니다.

딕셔너리의 KeyType은 Hashable 타입을 요구합니다.
하지만 딕셔너리의 KeyType으로 두 개의 Hashable 타입을 함께 사용하는 건 불가능합니다.
튜플로 두 개의 Hashable 타입인 String 타입을 묶고 딕셔너리의 키로 사용하는 방법 또한 스위프트에서 허용하지 않습니다.

아래 코드로 살펴봅시다.

```swift
let stringsTuple = ("I want to be part of a key", "Me too!")
let anotherDictionary = [stringsTuple: "I am a value"]  // error: type of expression is ambiguous without more context
```

두 개의 Hashable 타입을 튜플에 넣으면 더 이상 해당 튜플은 Hashable 타입이 아닙니다.
이런 경우 딕셔너리의 키로 두 개의 Hashable 타입을 사용하려면 어떻게 할까요?

Pair 커스텀 타입을 만들어 해결할 수 있습니다.
Pair 타입에 Hashable 타입 두 개를 묶어 Hashable 타입으로 딕셔너리의 키에 사용될 수 있습니다.
아래 코드로 Pair 타입 구현을 살펴봅시다.

```swift
struct Pair<T: Hashable> {
  let left: T
  let right: T

  init(_ left: T, _ right: T) {
    self.left = left
    self.right = right
  }
}  
```

위와 같이 Pair 타입을 구현한다면 Pair의 left와 right는 항상 같은 타입이어야 합니다.
left와 right가 다른 타입이어도 되도록 코드를 개선해 봅시다.

```swift
struct Pair<T: Hashable, U: Hashable> {
  let left: T
  let right: U

  init(_ left: T, _ right: U) {
    self.left = left
    self.right = right
  }
}

let pair = Pair("Tom", 20)
let pair2 = Pair("Tom", "Jerry")
```

이제는 Pair 타입의 left와 right가 Hashable을 따르는 서로 다른 타입이어도 좋습니다.

하지만 아직 Pair 타입은 Hashable 타입이 아닙니다.
Pair 타입이 hash value를 가지는 Hashable 타입이 되기 위해서는 추가적인 조치가 필요합니다.

첫 번째 방법은 Swift에 맡기는 것입니다. 
Swift 버전 4.1부터 Pair와 같은 객체의 두 프로퍼티(left, right)가 모두 Hashable일 경우 Pair 타입을 Hashable로 자동으로 만듭니다.
하지만 해당 방법은 열거형과 구조체에만 적용됩니다.

클래스의 경우 두 번째 방법을 사용하여 Hashable 타입을 명시해야 합니다. 
또한 클래스가 Hashable 프로토콜을 채택하면 hash 함수와 static == 함수를 모두 구현해야 합니다.

두 번째 방법은 직접 Pair 타입을 Hashable 프로토콜을 채택하여 Hashable 타입으로 만드는 것입니다.
아래 코드로 직접 Hashable 타입으로 만드는 방법을 살펴봅시다.

```swift
struct Pair<T: Hashable, U: Hashable>: Hashable {
  let left: T
  let right: U

  init(_ left: T, _ right: U) {
    self.left = left
    self.right = right
  }

  // Hashable 프로토콜을 직접 따르도록 구현했기 때문에 Hashable 프로토콜의 hash 함수를 구현해야 합니다.
  func hash(into hasher: inout Hasher) {
    hasher.combine(left)
    hasher.combine(right)
  }

  // Hashable 프로토콜을 직접 따르도록 구현했기 때문에 Hasahable 프로토콜이 따르는 Equatable 프로토콜의 static == 함수를 구현해야 합니다.
  static func ==(lhs: Pair<T, U>, rhs: Pair<T, U>) -> Bool {
    return lhs.left == rhs.left && lhs.right == rhs.right
  }
}

let pair = Pair<Int, Int>(10, 20)
print(pair.hashValue) // 52893198438921

let set: Set = [
  Pair("Laurel", "Hardy"),
  Pair("Harry", "Llody")
]
```

Pair 구조체에 Hashable 프로토콜을 명시적으로 채택하여 hash value를 가진 Hashable 타입으로 Pair 타입을 만들 수 있습니다.
이제 Pair 타입은 Hashable 타입이기 때문에 hasher를 넘길 수 있습니다.

아래 코드로 확인해 봅시다.

```swift
let pair = Pair("Madonna", "Cher")

var hasher = Hasher()
hasher.combine(pair) // pair.hash(into: &hasher)와 동일한 의미입니다.
let hash = hasher.finalize()
print(hash)  // 491240970192719
```

그렇다면 제네릭을 활용해 Hashable 키를 통해 값을 저장하는 캐시를 만들어 봅시다.
아래 코드로 확인해 봅시다.

```swift
class MiniCache<T: Hashable: U> {
  var cache = [T: U]()

  init() {}

  func insert(key: T, value: U) {
    cache[key] = value
  }

  func read(key: T) -> U> {
    return cache[key]
  }
}

let cache = MiniCache<Int, String>()
cache.insert(key: 100, value: "Jeff")
cache.insert(key: 200, value: "Miriam")
cache.read(key: 200)  // Optional("Miriam")
cache.read(key:99)  // Optional("Jeff")
```

## Generics and subtypes

서브 클래싱과 제네릭을 같이 쓸 때 다소 복잡한 상황이 발생할 수 있습니다.
서브 클래싱 상황에서 제네릭이 어떻게 동작하는지 알아 봅시다.

온라인 교육 서비스의 데이터를 모델링을 예로 살펴봅시다.
아래는 간단한 온라인 교육 모델입니다.

```swift
class OnlineCourse {
  func start() {
    print("Starting online course")
  }
}

class SwiftOnTheServer: OnlineCourse {
  override func start() {
    print("Starting Swift course.")
  }
}

var swiftCourse: SwiftOnTheServer = SwiftOnTheServer()
var course: OnlineCourse = swiftCourse // 가능합니다.
course.start  // "Starting Swift course."
```

위의 경우 제네릭을 사용하지 않은 상태에서 평범한 서브 클래싱입니다.

하지만 만약 Container<OnlineCourse>와 Container<SwiftOnTheServer> 같이 상속 관계의 클래스를 제네릭 타입 안으로 넣을 경우
Container<OnlineCourse>와 Container<SwiftOnTheServer>는 더 이상 상속 관계를 유지하지 않습니다.

![image](https://github.com/hongjunehuke/swift-in-depth/assets/83629193/4951f891-846a-44fe-8037-81830d02cc0f)

다시 말해, 제네릭 안에서는 클래스끼리의 상속 관계를 잃습니다.
따라서 아래 코드를 보면 부모 클래스로 자식 클래스를 참조하려 할 때 제네릭으로 인해 상속 관계가 깨지며 부모 타입으로 자식 타입을 참조할 수 없습니다.

```swift
struct Container<T> {}

var containerSwiftCourse: Container<SwiftOnTheServer> = Container<SwiftOnTheServer>()
// error: cannot convert value of type 'Container<SwiftOnTheServer>' to specified type 'Container<OnlineCourse>'
var containerOnlineCourse: Container<OnlineCourse> = containerSwiftCourse
```

만약 데이터를 저장하는 제네릭 타입 Cache가 있을 경우를 생각해 봅시다.
아래 코드는 Cache를 간략히 구현한 코드입니다.

```swift
struct Cache<T> {
  // ...생략
}

func refreshCache(_ cache: Cache<OnlineCourse>) {
  // ...생략
}

refreshCache(Cache<OnlineCourse>)  // 가능합니다.
refreshCache(Cache<SwiftOnTheServer>)  // 에러가 발생합니다. 
```

위에서 refreshCache 함수는 입력으로 Cache<OnlineCourse> 타입을 받고 있습니다.

물론 제네릭 타입이 아니고 일반적인 상속관계를 가진 OnlineCourse라면 refreshCache 함수에 SwiftOnTheServer 타입을 전달할 수 있습니다.
하지만 제네릭 타입인 Cache<OnlineCourse>이기 때문에 Cache<SwiftOnTheServer>는 Cache<OnlineCourse>와 상속 관계를 가지지 않기 때문에 전달될 수 없습니다.

제네릭 타입으로 클래스를 받을 경우 해당 클래스의 상속 관계를 깨트립니다.
하지만 상속 관계가 깨지는 제한 사항은 커스텀 타입에만 해당합니다.
Swift Standard Library의 Array와 Optional의 경우에는 상속 관계가 깨지지 않습니다.
Array와 Optional 모두 제네릭 타입이지만 제네릭에 들어가는 클래스의 상속 관계는 유지됩니다.

아래 코드로 확인해 봅시다.

```swift
func readOptionalCourse(_ value: Optional<OnlineCourse>) {
  //... 생략
}

readOptionalCourse(OnlineCourse())
readOptionalCourse(SwiftOnTheServer())
```

위의 readOptionalCourse 함수의 매개변수 Optional<OnlineCourse>는 Optional이기 때문에 제네릭의 상속 관계가 유지됩니다.
따라서 OnlineCourse()를 비롯해 OnlineCourse의 자식 클래스인 SwiftOnTheServer 타입도 전달할 수 있습니다.

다른 예로 Int?를 매개변수 타입으로 받는 함수에서 Int 값도 전달할 수 있는 이유도 옵셔널 타입이기 때문입니다.
(Int 타입은 옵셔널 Int의 자식 클래스입니다.)

옵셔널과 제네릭은 함께 주로 등장합니다.

제네릭을 통한 추상화는 더 복잡해지고 해석하기 어렵다는 비용이 들지만, 유연성을 얻을 수 있습니다.
어디든 trade-off는 존재합니다.

## Summary
- Adding an unconstrained generic to a function allows a function to work with all types.
- Generics can't be specialized from inside the scope of a function or type.
- Generic code is converted to specialized code that works on multipule types.
- Generics can be constrained for more specialized behavior, which may exclude some types.
- A type can be constrained to multiple generics to unlock more functionality on a generic type.
- Swift can synthesize implementations for the Equatable and Hashable protocols on structs and enums.
- Synthesizing that you write are invariant, and therefore you cannot use them as subtypes.
- Generic types in the standard library are covariant, and you can use them as subtypes. 
