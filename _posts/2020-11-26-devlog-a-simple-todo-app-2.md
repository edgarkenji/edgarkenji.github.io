---
layout: post
title: Devlog - A simple to-do app - part 2
tags: devlog swiftui swift ios
---

I'll begin this devlog with a simple to-do app in SwiftUI. The idea is to explain the basic concepts of this language while we learn together. This will be the general roadmap of this series:

* [Create a checkbox](% post_url 2020-11-25-devlog-sample-todo-app %)
* Add a text to the checkbox
* Use a data model to show a list of items
* Add an item
* Allow editing the text of the item
* Delete an item
* Add a tab bar and another view with a summary of the items
* Add persistence to the items

## Add a text to the checkbox

So now I want a label to go along with the checkbox, kinda like how the switch (it's actually called `Toggle`) works. Let's take a look at the relevant initializer:

```swift
Toggle(isOn: Binding<Bool>, @ViewBuilder label: ()->Label)
```

We can do the same for our Checkbox, but we'll have to change our `@State` to a `@Binding` variable, and adding a `@ViewBuilder` for our label:

```swift
struct Checkbox<Label: View> : View {
    
    @Binding var checked: Bool
    private var label: Label
    
    init(isChecked: Binding<Bool>, @ViewBuilder label: () -> Label) {
        self._checked = isChecked
        self.label = label()
    }
    // ...
}
```

* First thing to note about the above code is that we turned our checkbox into a generic struct by adding `Label`, because SwiftUI doesn't allow using View directly, we have to define a generic type constraint
* Why `@Binding`? Because we'll let the view containing the checkbox to update the value of our checkbox, instead of letting the checkbox own the state of the property
* We use `@ViewBuilder` to allow the label to be defined to whatever view the parent wants it to be

Now we need to put our label next to our checkbox:

```swift
var body: some View {
    HStack {
            Button(action: {
                checked.toggle()
            }, label: {
                checkbox
            })
            label
        }
    }
```

The final code:

```swift
struct Checkbox<Label: View> : View {
    
    @Binding var checked: Bool
    private var label: Label
    
    init(isChecked: Binding<Bool>, @ViewBuilder label: () -> Label) {
        self._checked = isChecked
        self.label = label()
    }

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
            }
        }
    }
    
    var body: some View {
        HStack {
            Button(action: {
                checked.toggle()
            }, label: {
                checkbox
            })
            label
        }
    }
}
```

Don't forget to update the previews:

```swift
struct Checkbox_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            Checkbox(isChecked: .constant(true), label: { Text("Checked") })
                .preferredColorScheme(.dark)
            Checkbox(isChecked: .constant(false), label: { Text("Not checked") })
        }
    }
}
```

You can check the full source code on [Github](https://github.com/edgarkenji/SimpleTodo/tree/feature/checkbox). Keep in mind that since it might be incomplete as I progress through this series.
