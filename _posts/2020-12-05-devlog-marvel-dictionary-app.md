---
layout: post
title: Devlog - A Marvel dictionary app - part 1
tags: devlog swiftui swift ios
---

As part of my portfolio, I'll make an open source dictionary app using the wonderful [Marvel API](https://developer.marvel.com/docs), so stay tuned. This will be the general roadmap for what I'm planning to do, but I might more things as I go:

* Get data from the API
* Decoding JSON data
* Show list of character names
* Show details of a character
* Search a specific character
* List pagination
* List cache
* Carrousel of highlight characters

## Get data from the API

First and foremost you need to create an account at [Marvel Dev Portal](https://developer.marvel.com/docs) in order to get the API key. After that we'll go through some hoops to actually make authenticated requests to their API. 

Upon reading through their [authentication doc](https://developer.marvel.com/documentation/authorization), it's a bit confusing because they mention client-side browser based apps don't need to send a hash, but don't mention native apps at all. We'll have to rely on the same way server-side authentication works. First, let's find how to convert a string into MD5 from [StackOverflow](https://stackoverflow.com/questions/32163848/how-can-i-convert-a-string-to-an-md5-hash-in-ios-using-swift):

```swift
import Foundation
import CryptoKit

func MD5(string: String) -> String {
    let digest = Insecure.MD5.hash(data: string.data(using: .utf8) ?? Data())

    return digest.map {
        String(format: "%02hhx", $0)
    }.joined()
}
```

Now we need to store thos API keys somewhere. There's a nice way to get those values from config files, provided by [NSHipster](https://nshipster.com/xcconfig/).

The idea is to have those keys on a `.xcconfig` file. Our `default.xcconfig` will look like this:

```
MARVEL_API_PRIVATE_KEY = YOUR_PRIVATE_KEY
MARVEL_API_PUBLIC_KEY = YOUR_PUBLIC_KEY
```

Following NSHipster's post, we'll have both `MARVEL_API_PRIVATE_KEY` and `MARVEL_API_PUBLIC_KEY` entries on our `Info.plist`, respectively with values `${MARVEL_API_PRIVATE_KEY}` and `${MARVEL_API_PUBLIC_KEY}`.

Don't forget to change the keys above to yours and apply the configuration file to the targets that will use them (via the Project Info tab).

The `Configuration` from NSHipster's post:

```swift
enum Configuration {
    enum Error: Swift.Error {
        case missingKey, invalidValue
    }

    static func value<T>(for key: String) throws -> T where T: LosslessStringConvertible {
        guard let object = Bundle.main.object(forInfoDictionaryKey:key) else {
            throw Error.missingKey
        }

        switch object {
        case let value as T:
            return value
        case let string as String:
            guard let value = T(string) else { fallthrough }
            return value
        default:
            throw Error.invalidValue
        }
    }
}
```

Now let's make a data structure to help us out whenever we need to send an authenticated request:

```swift
public struct MarvelApiAuth {
    private static var privateKey: String = try! Configuration.value(for: "MARVEL_API_PRIVATE_KEY")
    private static var publicKey: String = try! Configuration.value(for: "MARVEL_API_PUBLIC_KEY")
    
    private static var timestamp: String {
        String(Int(Date().timeIntervalSinceReferenceDate))
    }
    
    private static var hash: String {
        MD5(string: "\(timestamp)\(privateKey)\(publicKey)")
    }
    
    static var params: [String: String] {
        [
            "apikey": publicKey,
            "ts": timestamp,
            "hash": hash
        ]
    }
}
```

Now onto our API access specification. Back in the days of Alamofire and Moya, I really liked how using `enum` cases made it easy for us to create requests for each endpoint with specific parameters. So we'll make one for our API:

```swift
public enum MarvelApi {
    case characters
    
    internal var baseUrl: String { "https://gateway.marvel.com:443/v1/public" }
    
    internal var endpoint: String {
        switch self {
        case .characters:
            return "/characters"
        }
    }
    
    internal var method: String {
        switch self {
        default:
            return "GET"
        }
    }
        
    internal var params: [String : String] {
        return MarvelApiAuth.params
    }
    
    public var request: URLRequest {
        var urlComponents = URLComponents(string: "\(baseUrl)\(endpoint)")
        if method == "GET" {
            let queryItems = params.map { (key, value) -> URLQueryItem in
                URLQueryItem(name: key, value: value)
            }
            urlComponents?.queryItems = queryItems
        }
        guard let requestUrl = urlComponents?.url else {
            fatalError("Invalid URL")
        }
        var request = URLRequest(url: requestUrl)
        request.httpMethod = method
        return request
    }
}
```

IRL you may want to extract the `baseUrl` into the configuration file you created and separate it for debug and production environment as they might be different. Here, since it's the same URL, we're hardcoding because we're lazy.

How to make requests using the `enum` above? Well, we can simply call:

```
URLSession.shared.dataTask(with: MarvelApi.characters.request)
```

But to see it in action we'll update ContentView to call that request when showing anything. Check Xcode's console for the ouput:

```swift
struct ContentView: View {
    @State var cancellable: AnyCancellable?
    var body: some View {
        Text("Hello, world!")
            .padding()
            .onAppear(perform: {
                self.cancellable = URLSession
                    .shared
                    .dataTaskPublisher(for: MarvelApi.characters.request).sink { (completion) in
                        
                    } receiveValue: { (value) in
                        let (data, response) = value
                        print(String(data: data, encoding: .utf8))
                    }
            })
            .onDisappear(perform: {
                cancellable?.cancel()
            })
    }
}
```

You can check the full source code on [Github](https://github.com/edgarkenji/HeroDictionary). Keep in mind that since it might be incomplete as I progress through this series.
