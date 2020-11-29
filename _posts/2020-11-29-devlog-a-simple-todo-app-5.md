---
layout: post
title: Devlog - A simple to-do app - part 5
tags: devlog swiftui swift ios
---

I'll begin this devlog with a simple to-do app in SwiftUI. The idea is to explain the basic concepts of this language while we learn together. This will be the general roadmap of this series:

* [Create a checkbox](% post_url 2020-11-25-devlog-sample-todo-app %)
* [Add a text to the checkbox](% post_url 2020-11-26-devlog-sample-todo-app-2 %)
* [Use a data model to show a list of items](% post_url 2020-11-27-devlog-sample-todo-app-3 %)
* [Add an item](% post_url 2020-11-28-devlog-sample-todo-app-4 %)
* Allow editing the text of the item
* Delete an item
* Add a tab bar and another view with a summary of the items
* Add persistence to the items

## Allow editing the text of the item

We can add infinite items to our list but it's not useful if we can't edit the text on it, is it? So our goal now is to change the checkbox's text label to a textfield. So I'll rename our `CheckboxText` to `CheckboxTextfield` (also update all references):

```swift
struct CheckboxTextfield : View {
    @State var item: TodoItem
    
    var body: some View {
        Checkbox(isChecked: $item.checked) {
            TextField("Tap to edit your to-do item", text: $item.description)
        }
    }
}
```

Run the app and see that we can change the text on the to-do item. That was fast!

We'll take this opportunity to fix something that might throw some people off. Whenever we toggle the check of an item, the label moves. That's because our checked image occupies a larger space than when there's nothing.

So let's go back to our `Checkbox` and add a spacer with the same size and padding:

```swift
    var checkbox: some View {
        ZStack {
            Image(systemName: "square")
                .resizable()
                .frame(width:20, height:20, alignment: .center)
                .foregroundColor(.gray)
            if checked {
                Image(systemName: "checkmark")
                    .resizable()
                    .frame(width: 20, height: 20, alignment: .trailing)
                    .padding(EdgeInsets(top: 0, leading: 4, bottom: 4, trailing: 0))
                    .foregroundColor(Color.green)
            } else { // Add a spacer to fill up the same space as the checked image
                Spacer()
                    .frame(width: 20, height: 20, alignment: .trailing)
                    .padding(EdgeInsets(top: 0, leading: 4, bottom: 4, trailing: 0))
            }
        }
    }
```

And it's done! Now the text won't move around when toggling the checkbox.

You can check the full source code on [Github](https://github.com/edgarkenji/SimpleTodo/tree/feature/checkbox). Keep in mind that since it might be incomplete as I progress through this series.
