---
layout: post
title: Devlog - A simple to-do app - part 6
tags: devlog swiftui swift ios
---

I'll begin this devlog with a simple to-do app in SwiftUI. The idea is to explain the basic concepts of this language while we learn together. This will be the general roadmap of this series:

* [Create a checkbox](% post_url 2020-11-25-devlog-sample-todo-app %)
* [Add a text to the checkbox](% post_url 2020-11-26-devlog-sample-todo-app-2 %)
* [Use a data model to show a list of items](% post_url 2020-11-27-devlog-sample-todo-app-3 %)
* [Add an item](% post_url 2020-11-28-devlog-sample-todo-app-4 %)
* [Allow editing the text of the item](% post_url 2020-11-29-devlog-sample-todo-app-5 %)
* Delete an item
* Add a tab bar and another view with a summary of the items
* Add persistence to the items

## Delete an item

Now that we can add and change the content of the items, let's implement removing them as well. Let's go back to to our `TodoItemListView` where our list is composed:

```swift
struct TodoItemListView: View {
    
    @Binding var items: [TodoItem]
    
    var body: some View {
        List {
            ForEach(items, id:\.id) { (item) in
                CheckboxTextfield(item: item)
            }
            .onDelete(perform: { indexSet in // add this to allow deleting an item by swiping left on the item
                items.remove(atOffsets: indexSet)
            })
        }
    }
}
```

Run the app, add an item and swipe it to the left. The delete button will be revelead under the item. Quite easy, right?

Now, it's getting quite boring to look at an empty list of items whenever we start the app or remove all items. How about taking this opportunity to add a view representing an empty list? Something like this:

```swift
struct TodoItemsEmptyView : View {
    var body: some View {
        VStack {
            Spacer()
            Image(systemName: "tray")
                .resizable()
                .scaledToFit()
                .frame(width: 48, height: 48, alignment: .center)
                .foregroundColor(.gray)
            Text("Nothing to do, no way to go home!")
                .foregroundColor(.gray)
            Spacer()
        }
    }
}

struct TodoItemsEmptyView_Previews: PreviewProvider {
    static var previews: some View {
        TodoItemsEmptyView()
    }
}
```

No let's update our list again:

```swift
struct TodoItemListView: View {
    
    @Binding var items: [TodoItem]
    
    var body: some View {
        if items.isEmpty {
            TodoItemsEmptyView()
        } else {
            List {
                ForEach(items, id:\.id) { (item) in
                    CheckboxTextfield(item: item)
                }
                .onDelete(perform: { indexSet in
                    items.remove(atOffsets: indexSet)
                })
            }
        }
    }
}
```

Now we have an empty list view to nag us to add something.

You can check the full source code on [Github](https://github.com/edgarkenji/SimpleTodo/tree/feature/checkbox). Keep in mind that since it might be incomplete as I progress through this series.
