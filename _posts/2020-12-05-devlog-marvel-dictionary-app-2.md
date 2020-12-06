---
layout: post
title: Devlog - A Marvel dictionary app - part 2
tags: devlog swiftui swift ios
---

As part of my portfolio, I'll make an open source dictionary app using the wonderful [Marvel API](https://developer.marvel.com/docs), so stay tuned. This will be the general roadmap for what I'm planning to do, but I might more things as I go:

* [Get data from the API](% post_url 2020-12-05-devlog-marvel-dictionary-app %)
* Decoding JSON data
* Show list of character names
* Show details of a character
* Search a specific character
* List pagination
* List cache
* Carrousel of highlight characters

## Decoding JSON data

Our request side is done, and now we have to make use of the response. Swift makes it easy for us to decode JSON responses into actual data classes, we just have to make it conform to `Decodable`:

```swift
public enum ImageType: String, Decodable {
    case portrait, standard, landscape
}

public enum ImageSize: String, Decodable {
    case small, medium, large, amazing, xlarge, fantastic, incredible, uncanny
}

public struct ImageUrl: Decodable {
    let path: String
    let `extension`: String
}

public struct CharacterDataWrapper: Decodable {
    let code: Int?
    let status: String?
    let copyright: String?
    let attributionText: String?
    let etag: String?
    public let data: CharacterDataContainer?
}

public struct CharacterDataContainer: Decodable {
    let offset: Int?
    let limit: Int?
    let total: Int?
    let count: Int?
    public let results: [Character]
}

public struct Character: Decodable {
    public let id: Int
    public let name: String?
    public let description: String?
    public let modified: Date?
    public let thumbnail: ImageUrl?
    
    public init(id: Int, name: String?, description: String?, modified: Date?, thumbnail: ImageUrl?) {
        self.id = id
        self.name = name
        self.description = description
        self.modified = modified
        self.thumbnail = thumbnail
    }
}

public extension Character {
    var smallThumbnailUrl: URL? {
        guard let path = thumbnail?.path, let imageExtension = thumbnail?.extension else { return nil }
        let url = "\(path)/\(ImageType.standard.rawValue)_\(ImageSize.small.rawValue).\(imageExtension)"
        return URL(string: url)
    }
}
```

We'll be using URLSession's `dataTaskPublisher` to get the response in a reactive way. It'd be nice if we could get it already decoded, like this:

```swift
extension URLSession {
    func decodedDataTaskPublisher<T: Decodable>(for request: URLRequest,
                                                handleResponse: ((Data, URLResponse) throws -> Data)? = nil) -> AnyPublisher<T, Error> {
        let decoder = JSONDecoder()
        decoder.dateDecodingStrategy = .iso8601
        return dataTaskPublisher(for: request)
            .tryMap({ (params) -> Data in
                return try handleResponse?(params.data, params.response) ?? params.data
            })
            .decode(type: T.self, decoder: decoder)
            .eraseToAnyPublisher()
    }
}
```

This is basically just using a `JSONDecoder` to decode the response as part of the publisher chain. Note that we're also providing a closure as a way to handle response if needed, i.e. when there's an error.

Let's update our `ContentView` to use this new method:

```swift
struct ContentView: View {
    @State var cancellable: AnyCancellable?
    var body: some View {
        Text("Hello, world!")
            .padding()
            .onAppear(perform: {
                self.cancellable = URLSession
                    .shared
                    .decodedDataTaskPublisher(for: MarvelApi.characters.request).sink { (completion) in
                        
                    } receiveValue: { (data: CharacterDataWrapper) in
                        print(data)
                    }
            })
            .onDisappear(perform: {
                cancellable?.cancel()
            })
    }
}
```

Check the log to see we're getting a bunch of data within our `CharacterDataWrapper`. Next in line we'll make a repository that will help us load this data, and finally put it into a list view on our app.

You can check the full source code on [Github](https://github.com/edgarkenji/HeroDictionary). Keep in mind that since it might be incomplete as I progress through this series.
