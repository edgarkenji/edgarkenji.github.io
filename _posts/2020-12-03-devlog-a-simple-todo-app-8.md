---
layout: post
title: Devlog - A simple to-do app - part 8
tags: devlog swiftui swift ios
---

I'll begin this devlog with a simple to-do app in SwiftUI. The idea is to explain the basic concepts of this language while we learn together. This will be the general roadmap of this series:

* [Create a checkbox](% post_url 2020-11-25-devlog-sample-todo-app %)
* [Add a text to the checkbox](% post_url 2020-11-26-devlog-sample-todo-app-2 %)
* [Use a data model to show a list of items](% post_url 2020-11-27-devlog-sample-todo-app-3 %)
* [Add an item](% post_url 2020-11-28-devlog-sample-todo-app-4 %)
* [Allow editing the text of the item](% post_url 2020-11-29-devlog-sample-todo-app-5 %)
* [Delete an item](% post_url 2020-11-30-devlog-sample-todo-app-6 %)
* [Add a tab bar and another view with a summary of the items](% post_url 2020-12-01-devlog-sample-todo-app-7 %)
* Add persistence to the items

## Add persistence to the items

Our to-do app is working, but not really useful because every time we restart the app, all our data is gone. Let's change this by finally adding persistence. We're going to use SwiftUI's `@AppStorage` property wrapper.

How does it work? It's basically an easy way to store and load data from the apps's standard `UserDefaults`. This is not recommended for storing critical data, so be careful what you add on your to-do list.

We'll start by adding a way to store our `TodoItem` by turning into a `Codable`:

```swift
extension TodoItem : Codable {}
```

Since all the properties of our item are primary, they'll all be automatically encoded into a JSON data. Now we'll change our repository so that we can save and load the data off the storage:

```swift
class TodoItemRepository : ObservableObject {
    @Published var items: [TodoItem] = []
    @AppStorage("items") var storedItems: Data?
    
    private var cancellable: AnyCancellable?
    
    init(items: [TodoItem] = []) {
        if let stored = storedItems {
            self.items = (try? JSONDecoder().decode([TodoItem].self, from: stored)) ?? items
        } else {
            self.items = items
        }
        
        cancellable = $items.sink(receiveValue: { (items) in
            self.storedItems = try? JSONEncoder().encode(items)
        })
    }
    
    deinit {
        cancellable?.cancel()
    }
}
```

Lots of things going on here:

* The `storedItems` is a `Data` because that's how we can represent a list of Codable objects in UserDefaults
* We initialize our items with either the decoded data from the storage by using `JSONDecoder`
* By using Combine, we automatically save new items whenever the published list is updated, by converting it to a JSON data with `JSONEncoder`
* We make sure our cancellable is stored and cancelled when appropriate

This is it. We've implemented basic persistence by just using `@AppStorage` and Combine.

Now this to-do app has very basic functionalities. We can expand it as much as possible, be it by adding multiple collections or other properties, alerts and such.

You can check the full source code on [Github](https://github.com/edgarkenji/SimpleTodo/tree/feature/checkbox). Keep in mind that since it might be incomplete as I progress through this series.
