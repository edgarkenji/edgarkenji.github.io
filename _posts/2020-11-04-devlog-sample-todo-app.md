---
layout: post
title: Devlog - Um app de tarefas simples I
tags: devlog swiftui swift ios
---

Vou começar este devlog com um simples app de lista de tarefas (to-do) feita em SwiftUI. A intenção é explicar os conceitos básicos de SwiftUI para quem quer aprender junto comigo.

Segue a idéia geral desta série:

* Criar um checkbox
* Adicionar texto ao checkbox
* Criar um data model e apresentar uma lista de itens
* Adicionar um item
* Permitir modificar o texto do item
* Apagar o item
* Adicionar uma tab bar e uma lista somente-leitura dos itens
* Persistir os itens localmente

# 1 - Criar um checkbox

Mas Kenji, não existe um checkbox pronto em SwiftUI? Bem, existe o tal "Toggle", que é aquela chavinha que fica verde quando ligada, mas não representa tão bem o conceito de uma lista de tarefas. Mas vou utilizar o mesmo tipo de construção para criar uma view própria para o checkbox:

```swift
Checkbox(isChecked: Binding<Bool>, label: @ViewBuilder ()->View)
```

Há varios jeitos de fazer uma caixa com dois estados (checado e não-checado), mas o mais fácil é usar duas imagens da biblioteca SF Symbols da própria Apple uma em cima da outra. Por baixo, a imagem "square", e por cima, quando a caixa está checada, a imagem "check" é mostrada, senão permanece vazia. Para finalizar, essa view é mostrada como a apresentação de um button.


```swift
@State var checked: Bool // mantém o estado de checado ou não
var checkbox: some View {
    ZStack { // vamos empilhar cada view a seguir uma em cima da outra
        Image(systemName: "square")
            .resizable()
            .frame(width:20, height:20, alignment: .center)
            .foregroundColor(.gray)
        if checked { // se checado mostramos o check
            Image(systemName: "checkmark")
                .resizable()
                .frame(width: 20, height: 20, alignment: .trailing)
                .padding(EdgeInsets(top: 0, leading: 4, bottom: 4, trailing: 0))
                .foregroundColor(Color.green)
        }
    }
}
```

Agora precisamos tratar a caixa como um botão.

```swift
var body: some View {
    HStack {
        Button(action: {
            checked = !checked
        }, label: {
            checkbox
        })
        label
    }
}
```

O código final: 

```swift
struct CheckboxView : View {
    @State var checked: Bool
    var body: some View {
        Button(action: {
            self.checked = !self.checked
        }) {
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
                } else {
                    Spacer()
                        .frame(width: 20, height: 20, alignment: .trailing)
                        
                        .padding(EdgeInsets(top: 0, leading: 4, bottom: 4, trailing: 0))
                }
            }
        }
        .foregroundColor(Color(UIColor.systemBackground))
    }
}

```

