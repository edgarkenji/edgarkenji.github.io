---
layout: post
title: Devlog - A simple to-do app - part 4
tags: devlog swiftui swift ios
---

I'll begin this devlog with a simple to-do app in SwiftUI. The idea is to explain the basic concepts of this language while we learn together. This will be the general roadmap of this series:

* [Create a checkbox](% post_url 2020-11-25-devlog-sample-todo-app %)
* [Add a text to the checkbox](% post_url 2020-11-26-devlog-sample-todo-app-2 %)
* [Use a data model to show a list of items](% post_url 2020-11-27-devlog-sample-todo-app-3 %)
* Add an item
* Allow editing the text of the item
* Delete an item
* Add a tab bar and another view with a summary of the items
* Add persistence to the items

## Add an item

So our list is a bit boring if we can't even add to it. We'll start by improving some things first.

Let's update our `TodoItem` to be `Identifiable` so that we can have the list updated correctly later on.

```swift
struct TodoItem: Identifiable {
    var id: UUID = UUID()
    var description: String
    var checked: Bool
}
```

And also make a wrapper for our Checkbox with a text label, so that it holds the state of each item separately:
```swift
struct CheckboxText : View {
    @State var item: TodoItem
    
    var body: some View {
        Checkbox(isChecked: $item.checked) {
            Text(item.description)
        }
    }
}
```

We'll also use a repository to store all the items in memory for now, and make it an `ObservableObject`. That way, SwiftUI will update the list whenever we make changes to the items:

```swift
class TodoItemRepository : ObservableObject {
    @Published var items: [TodoItem] = []
    
    init(items: [TodoItem] = []) {
        self.items = items
    }
}
```

The idea is to have a general purpose view that holds the list and the other components, such as the add button. We'll follow the MVVM pattern and use a View Model to hold the logic for adding items:

```swift
class TodoItemsViewModel : ObservableObject {
    @ObservedObject var repository: TodoItemRepository
    
    
    init(repository: TodoItemRepository) {
        self.repository = repository
    }
        
    func add(items: String...) {
        items.forEach {
            self.repository.items.append(TodoItem(description: $0, checked: false))
        }
    }
}
```

And our view will be as below. Note that I added the add button at the bottom of the screen to replicate how Apple's own to-do list works.

```swift
struct TodoItemsView: View {
    @ObservedObject var viewModel: TodoItemsViewModel
    
    init(viewModel: TodoItemsViewModel) {
        self.viewModel = viewModel
    }
    
    var body: some View {
        NavigationView {
            VStack {
                TodoItemListView(items: viewModel.$repository.items)
                    .listStyle(InsetGroupedListStyle())
                Button {
                    withAnimation {
                        viewModel.add(items: "")
                    }
                } label: {
                    HStack {
                        Image(systemName: "plus")
                        Text("Add new item")
                    }.padding(EdgeInsets(top: 8, leading: 0, bottom: 12, trailing: 0))
                }
            }
            .navigationTitle("To-do")
        }.edgesIgnoringSafeArea(.all)
    }
}
```

Let's update our `ContentView` and the preview to show this view:

```swift
struct ContentView: View {
    @EnvironmentObject var repository: TodoItemRepository
    
    var body: some View {
        TodoItemsView(viewModel: TodoItemsViewModel(repository: repository))
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
            .environmentObject(TodoItemRepository(items: [
                TodoItem(id: UUID(), description: "Teste 1", checked: false),
                TodoItem(id: UUID(), description: "Teste 2", checked: false)
            ]))
    }
}
```

Note that we're using `@EnvironmentObject` for our repository. The repository is actually initialized on our `SimpleTodoApp` and passed over by the environment:

```swift
@main
struct SimpleTodoApp: App {
    @ObservedObject var repository: TodoItemRepository = TodoItemRepository()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(repository)
        }
    }
}
```

Phew, that's it. We can now test the app on a device or simulator and see that the button adds an item to the list.

You can check the full source code on [Github](https://github.com/edgarkenji/SimpleTodo/tree/feature/checkbox). Keep in mind that since it might be incomplete as I progress through this series.
