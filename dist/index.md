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
