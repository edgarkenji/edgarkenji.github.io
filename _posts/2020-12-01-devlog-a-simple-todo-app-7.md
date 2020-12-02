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
* [Delete an item](% post_url 2020-11-30-devlog-sample-todo-app-6 %)
* Add a tab bar and another view with a summary of the items
* Add persistence to the items

## Add a tab bar and another view with a summary of the items

We can now add, update and delete items, but how can we be sure the items are actually changing where we're storing them?

One way to visualize this is to add another view that shows a read-only list of items, like a history of your to-do items. Let's begin with a new view with a list:

```swift
struct SummaryView : View {
    @Binding var list: [TodoItem]
    
    var body: some View {
        NavigationView {
            List {
                ForEach(list, id: \.id) {
                    if $0.checked {
                        Text($0.description)
                            .strikethrough() // apply a strikthrough effect for checked items
                    } else {
                        Text($0.description)
                    }
                }
            }.navigationTitle("Summary")
        }
    }
}
```

We're even adding a strikethrough effect for items that are already checked as done. Check in the preview that it works as expected:

```swift
struct SummaryView_Previews: PreviewProvider {
    static var previews: some View {
        SummaryView(list: .constant([
            TodoItem(id: UUID(), description: "Teste 1", checked: true),
            TodoItem(id: UUID(), description: "Teste 2", checked: false)
        ]))
    }
}
```

Now how can we show this new view alongside our to-do list? Let's use `TabView` to show these into different tabs on our `ContentView`:

```swift
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
```

If you run the app, the tab and summary is working, but try adding an item and changing its text. The summary shows an empty item instead of whatever we type there. What's going on?

Let's rememeber how our to-do list is done. We're using a `Binding<[TodoItem]>` to make a list, and each `TodoItem` is passed to the `CheckboxTextfield`. But this view uses `@State` to store its own state, meaning the text changes don't get updated on the original list.

Can we just swap `@State` to `@Binding`? Well, it's not that easy, as the list passes an individual `TodoItem` instead of a `Binding`. We'd have to find a way to pass the item back to the list to be updated. 

Let's add a callback to delegate the update to our list view:

```swift
struct CheckboxTextfield : View {
    @State var item: TodoItem
    var onUpdate: (TodoItem)->() // our callback for the list to update the item
    
    var body: some View {
        Checkbox(isChecked: $item.checked) {
            TextField("Tap to edit your to-do item", text: $item.description) { (editing) in // update the item both when out of editing
                if !editing {
                    onUpdate(self.item)
                }
            } onCommit: { // and when it's commited
                onUpdate(self.item)
            }
        }
    }
}
```

Our list is now responsible for updating the item using the `UUID` of the item:

```swift
struct TodoItemListView: View {
    
    @Binding var items: [TodoItem]
    
    var body: some View {
        if items.isEmpty {
            TodoItemsEmptyView()
        } else {
            List {
                ForEach(items, id:\.id) { (item) in
                    CheckboxTextfield(item: item) { (item) in
                        let first = items.firstIndex { (lookup) -> Bool in // lookup the item through it's id
                            lookup.id == item.id
                        }
                        guard let index = first else { return }
                        items[index] = item // update the item
                    }
                }
                .onDelete(perform: { indexSet in
                    items.remove(atOffsets: indexSet)
                })
            }
        }
    }
}
```

Both our to-do and summary are now up to date.

You can check the full source code on [Github](https://github.com/edgarkenji/SimpleTodo/tree/feature/checkbox). Keep in mind that since it might be incomplete as I progress through this series.
