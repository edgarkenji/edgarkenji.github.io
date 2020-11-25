---
layout: post
title: Devlog - A simple to-do app - part 1
tags: devlog swiftui swift ios
---

I'll begin this devlog with a simple to-do app in SwiftUI. The idea is to explain the basic concepts of this language while we learn together. This will be the general roadmap of this series:

* Create a checkbox
* Add a text to the checkbox
* Use a data model to show a list of items
* Add an item
* Allow editing the text of the item
* Delete an item
* Add a tab bar and another view with a summary of the items
* Add persistence to the items

## Create a checkbox

"Wait, there isn't a built-in checkbox?"

Well, if you're versed in the UIKit world you might know about the "switch" component, which works exactly like a checkbox but doesn't really feel like a checkbox. That's why we're building one from scratch.

So how do we get it done? There are several ways to do it, but maybe the easiest one is to just overlap two images, one for the box, and another for the check, as Swift UI 2.0 makes it easy to just conditionally change the view that's used.

We'll be using system images so if you want to check which ones are available, you can use Apple's own SF Symbols app for that.

```swift
@State var checked: Bool // keep the check state
var checkbox: some View {
    ZStack { // stack the views one over the other
        Image(systemName: "square")
            .resizable() // make the image resizable
            .frame(width:20, height:20, alignment: .center) // let's fit into a 20x20 frame
            .foregroundColor(.gray) // and paint it gray
        if checked {
            Image(systemName: "checkmark")
                .resizable()
                .frame(width: 20, height: 20, alignment: .trailing)
                .padding(EdgeInsets(top: 0, leading: 4, bottom: 4, trailing: 0)) // offset the check a bit to make it look like it's coming out of the box
                .foregroundColor(Color.green) // we'll paint this one green
        }
    }
}
```

Now, to treat the component like a button:

```swift
var body: some View {
    Button(action: {
        checked.toggle()
    }, label: {
        checkbox
    })
}
```

The final code:

```swift
struct Checkbox : View {
    @State var checked: Bool
    
    var body: some View {
        Button {
            checked.toggle()
        } label: {
            checkbox
        }
    }
    
    var checkbox: some View {
        ZStack { // stack the views one over the other
            Image(systemName: "square")
                .resizable() // make the image resizable
                .frame(width:20, height:20, alignment: .center) // let's fit into a 20x20 frame
                .foregroundColor(.gray) // and paint it gray
            if checked {
                Image(systemName: "checkmark")
                    .resizable()
                    .frame(width: 20, height: 20, alignment: .trailing)
                    .padding(EdgeInsets(top: 0, leading: 4, bottom: 4, trailing: 0)) // offset the check a bit to make it look like it's coming out of the box
                    .foregroundColor(Color.green) // we'll paint this one green
            }
        }
    }
}

```

Let's add a preview to see how it looks like:

```swift
struct Checkbox_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            Checkbox(checked: true)
                .preferredColorScheme(.dark)
            Checkbox(checked: false)
        }
    }
}
```

You can check the full source code on [Github](https://github.com/edgarkenji/SimpleTodo/tree/feature/checkbox). Keep in mind that since it might be incomplete as I progress through this series.
