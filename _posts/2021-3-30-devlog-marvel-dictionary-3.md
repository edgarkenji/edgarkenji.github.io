---
layout: post
title: Devlog - A Marvel dictionary app - part 3
tags: devlog swiftui swift ios
---

As part of my portfolio, I'll make an open source dictionary app using the wonderful [Marvel API](https://developer.marvel.com/docs), so stay tuned. This will be the general roadmap for what I'm planning to do, but I might more things as I go:

* [Get data from the API](% post_url 2020-12-05-devlog-marvel-dictionary-app %)
* [Decoding JSON data](% post_url 2020-12-05-devlog-marvel-dictionary-app-2 %)
* Show list of character names
* Show details of a character
* Search a specific character
* List pagination
* List cache
* Carrousel of highlight characters

## Show list of character names

We have our data coming from the API so it's time to finally put our hands on some UI code. We'll use a simple list for this for now:

```swift
struct CharactersView : View {
    @State var items: [HeroDictionary.Character]
    var body: some View {
        List {
            ForEach(items, id: \.id) {
                Text($0.name ?? "-")
            }
        }
    }
}

struct CharactersView_Preview : PreviewProvider {
    static var previews: some View {
        CharactersView(items: [
            Character(id: 0, name: "Iron-man", description: "Iron-man", modified: Date(), thumbnail: nil),
            Character(id: 1, name: "Spider-man", description: "Spider-man", modified: Date(), thumbnail: nil),
        ])
    }
}
```

Also `Character` needs to be `Identifiable`:

```swift
extension Character : Identifiable {}
```

It's kinda boring right now, so let's make something more interesting by making a proper view for our list item:

```swift
struct CharactersItemView: View {
    @State var item: HeroDictionary.Character
    var body: some View {
        HStack {
            Image(systemName: "person.fill")
                .resizable()
                .frame(width: 32, height: 32, alignment: .center)
                .padding()
                .background(RoundedRectangle(cornerRadius: 32)
                                .foregroundColor(.secondary))
            Text(item.name ?? "-")
        }
    }
}

struct CharactersItemView_Preview: PreviewProvider {
    static var previews: some View {
        CharactersItemView(item: Character(id: 0, name: "Captain America", description: "Captain America", modified: Date(), thumbnail: nil))
    }
}
```

And updating our list:

```swift
struct CharactersView : View {
    @State var items: [HeroDictionary.Character]
    var body: some View {
        List {
            ForEach(items, id: \.id) { item in
                CharactersItemView(item: item)
            }
        }
    }
}
```

It's still a boring list though, becasue we're not showing the hero's proper image. 

But we can't show the image straight from the URL because it could take a long time to download and show all images, and also it'd refresh every time a new cell is shown.

Since the purpose of this tutorial is to integrate API to SwiftUI and not manage image downloading, we'll use an awesome code from [Yet Another Swift Blog](https://www.vadimbulavin.com/asynchronous-swiftui-image-loading-from-url-with-combine-and-swift/). It handles loading images asynchronously, as well as caching them. Check out their [repo](https://github.com/V8tr/AsyncImage) as well.

So using `AsyncImage`, our new item will look like:

```swift
struct CharactersItemView: View {
    @State var item: HeroDictionary.Character
    var body: some View {
        HStack {
            AsyncImage(url: item.smallThumbnailUrl!, placeholder: {
                Image(systemName: "person.fill")
            })
            .frame(width: 32, height: 32, alignment: .center)
            .clipped()
            .background(RoundedRectangle(cornerRadius: 32)
                            .foregroundColor(.secondary))
            .clipShape(Circle())

            Text(item.name ?? "-")
        }
    }
}
```

Notice we're using `clipShape` to make the image a circle.

Now we need to wrap everything up by actually downloading the data off the API into our list. For that, we make a `CharacterRepository`:

```swift
class CharacterRepository: ObservableObject {
    @Published var characters: [HeroDictionary.Character] = [] {
        willSet {
            objectWillChange.send()
        }
    }
    
    private var cancellable: AnyCancellable? = nil
    
    func load() {
        self.cancellable = URLSession
            .shared
            .decodedDataTaskPublisher(for: MarvelApi.characters.request)
            .receive(on: DispatchQueue.main)
            .print()
            .sink { (completion) in
                print(completion)
            } receiveValue: { (wrapper: CharacterDataWrapper) in
                self.characters = wrapper.data?.results ?? []
                print(self.characters)
            }
    }
    
    func cancel() {
        cancellable?.cancel()
    }
}
```

This will be responsible for loading the character list from the API into a `Published` property. Our view will then use this property to fill up the list. So our content view now looks like:

```swift
struct ContentView: View {
    @ObservedObject var repository = CharacterRepository()

    var body: some View {
        CharactersView(items: self.$repository.characters)
            .onAppear(perform: {
                repository.load()
            })
            .onDisappear(perform: {
                repository.cancel()
            })
    }
}
```

Before running, we have to make sure to add some items into our Info.plist file regarding App Transport Security, since our images come from an `http` server, iOS requires we register a temporary exception to our app:

```plist
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSExceptionDomains</key>
        <dict>
            <key>http://i.annihil.us</key>
            <dict>
                <key>NSIncludesSubdomains</key>
                <true/>
                <key>NSExceptionAllowsInsecureHTTPLoads</key>
                <true/>
                <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
                <true/>
            </dict>
        </dict>
    </dict>
```


Run the simulator and we now have a list with filled items. It's still only the start of the list because we don't have pagination nor infinite scrolling, but it's a start.

You can check the full source code on [Github](https://github.com/edgarkenji/HeroDictionary). Keep in mind that since it might be incomplete as I progress through this series.
