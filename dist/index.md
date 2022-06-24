### Learning Swift and iOS as a Web Developer
#### 勉強会 (benky&#333;kai) by Karol Moroz
#### June 2022


### Motivation

* always wanted to build mobile apps
* React Native is obese
* Flutter is made by Google, and Google is bad
* Android is made by Google and uses Java


### First Attempt

* bought a book 2 years ago
* never read it


### Second Attempt

* built a React Native app for China Steel
* it had to work on the Web, too
* ended up deploying a PWA


### Iron Man Challenge 鐵人賽

* write posts about a topic for 30 days
* no excuses or breaks
* held by iThome, a Stack Overflow clone
* 1st prize is NT$10,000, or about US$330


### My challenge

* posts published at <a href="https://moroz.dev/blog" target="_blank" rel="noopener noreferrer">moroz.dev/blog</a>
* 28 posts
* learning Swift and SwiftUI


### Takeaways

* Swift is nice
* full type safety but not too problematic
* SwiftUI is declarative and looks like Flutter
* Generics, protocols, enums with values, optionals


## Swift
### Type safety


### Type inference #1

```swift
// inferred as Date
let date = Date()

// inferred as String
let productJSON = """
{
  "title": "A book",
  "price":"420.69",
  "description": "This is a JSON object describing a book."
}
"""

// inferred as Optional<Product>
let decoded = try? JSONDecoder().decode(Product.self, from: productJSON)
```


### Type inference with generics

```swift
struct TokenPair: Codable {
  let accessToken: String; let refreshToken: String
}
class KeychainHelper { // ...
  func read<T>(service: String = SERVICE, account: String = ACCOUNT)
      -> T? where T: Codable {
    guard let data = read(service: service, account: account) else { return nil }
    do {
      let item: T = try JSONDecoder().decode(T.self, from: data)
      return item
    } catch {
      assertionFailure("Failed to decode item for keychain: \(error)")
      return nil
    }
  }
}
let tokens: TokenPair? = KeychainHelper.shared.read() // T inferred as TokenPair
```


### Type safety

Only optional types can be nil:

```swift
var caption = "I am a string"
caption = nil // won't compile 

var optionalCaption: String? = "I am an optional string"
optionalCaption = nil // OK
```


### Optional binding

Swift:

```swift
func doSomethingWithOptional(arg: SomeStruct?) {
  // SomeStruct? is shorthand for Optional<SomeStruct>
  guard let arg = arg else {
    return
  }

  // ... do something
  // if let works too
}
```

Rust:

```rust
pub fn do_something(arg: Option<SomeStruct>) {
  if let arg = arg {
    // ...do something
  }
}
```


## iOS


### Different from the Web #1

* no URLs
* no HTML, CSS, JavaScript
* no forms
* no cookie session
* no `JSON.parse` to dynamic structures


### Different from the Web #2

* run loops, threads, `DispatchQueue`
* Objective-C and NeXTStep legacy
* mostly compatible with other Apple OSes
* you can run raw C, if you dare


### NeXTStep legacy

* classes starting with NS
* old but robust standard library
* `NSLocalizableString`
* `NSURLSession` (=> `URLSession`)


### Objective-C legacy

```swift
// Storing a token in system keychain be like

let query: [String: Any] =
  [
    kSecValueData as String: data,
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrService as String: service,
    kSecAttrAccount as String: account,
  ]

// positional arguments
let status = SecItemAdd(query as CFDictionary, nil)

// success check by comparing to a negative int constant
if status == errSecDuplicateItem { // ...
```


### Very powerful locale support

```swift
import Foundation

let date = Date()

let formatter = DateFormatter()
formatter.dateStyle = .full
formatter.timeStyle = .full
formatter.timeZone = TimeZone(identifier: "Asia/Tokyo")!

formatter.locale = Locale(identifier: "ja-JP")
print(formatter.string(from: date))
// 2022年6月24日 金曜日 20時21分58秒 日本標準時
formatter.locale = Locale(identifier: "uk-UA")
print(formatter.string(from: date))
// пʼятниця, 24 червня 2022 р. о 20:21:58 за японським стандартним часом
formatter.locale = Locale(identifier: "es-MX")
print(formatter.string(from: date))
// viernes, 24 de junio de 2022, 20:21:58 hora estándar de Japón
```


### Two frameworks

* UIKit &mdash; old but more comprehensive<br/>MVC, clicking on storyboards, loads of code
* SwiftUI &mdash; bleeding-edge declarative UI<br/>MVVM, stateful views
* fully interoperable


### Persistence

* Realm (cross-platform)
* Firebase (cross-platform)
* SQLite (cross-platform)
* Core Data (Apple only)


### Async I/O

```swift
Network.shared.apollo.fetch(query: CurrentUserQuery()) { [unowned self] result in
  self.loading = false

  guard let data = try? result.get().data else {
    return
  }

  if let user = try? User(from: data.user.fragments.userDetails) {
    withAnimation {
      self.user = user
    }
  }
}
```


### Navigation

```swift
struct IndexView: View {
  @EnvironmentObject private var dataStore: DataStore

  var body: some View {
    Group {
      if dataStore.practices.count == 0 {
        Text("Loading...")
      } else {
        List(dataStore.practices) { practice in
          NavigationLink(practice.name,
            destination: PracticeView(practice: practice)) // <=== 
        }
        .navigationTitle("Practices")
      }
    }
    .onAppear { try? dataStore.loadPractices() }
  }
}
```


## Thank 🦃
