---
layout: post
title: Devlog - A simple to-do app - part 3
tags: devlog swiftui swift ios
---

I'll begin this devlog with a simple to-do app in SwiftUI. The idea is to explain the basic concepts of this language while we learn together. This will be the general roadmap of this series:

* [Create a checkbox](% post_url 2020-11-25-devlog-sample-todo-app %)
* [Add a text to the checkbox](% post_url 2020-11-26-devlog-sample-todo-app-2 %)
* Use a data model to show a list of items
* Add an item
* Allow editing the text of the item
* Delete an item
* Add a tab bar and another view with a summary of the items
* Add persistence to the items

## Use a data model to show a list of items

It's time to show our checkbox into a list of items. For that, we'll need a data model to hold the state of our items, as well as a view to show the list.

To start things off, let's create our `TodoItem` data model:

```swift
struct TodoItem {
    var description: String
    var checked: Bool
}
```

Our list will look like:

```swift
struct TodoItemListView: View {
    @Binding var items: [TodoItem]
    
    var body: some View {
        List {
            ForEach(items.indices) { (index) in
                Checkbox(isChecked: self.$items[index].checked) {
                    Text(self.items[index].description)
                }
            }
        }
    }
}
```

Our preview will look like:

```swift
struct TodoItemListView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            TodoItemListView(items: .constant([
                TodoItem(description: "Teste", checked: true),
                TodoItem(description: "Teste 2", checked: false)
            ]))
            TodoItemListView(items: .constant([]))
        }
    }
}
```

That's a list alright, but it's a boring static list. Next up we'll make our list to be more dynamic, allowing us to add items to this list.

You can check the full source code on [Github](https://github.com/edgarkenji/SimpleTodo/tree/feature/checkbox). Keep in mind that since it might be incomplete as I progress through this series.
