# SwiftUI View 的生命周期影响 @StateObject 和 @State 属性存储的数据

在开发 SwiftUI 应用时，为了实现“数据唯一来源（source of truth）”并确保数据同步，我们通常：

- 使用 `@State` 与 `@Binding` 来管理结构体（`struct`）类型的数据；
- 使用 `@StateObject` 与 `@ObservedObject` 来管理类（`class`）类型的数据。

其中：

- 用 `@State` 声明的属性是数据的来源，其它通过 `@Binding` 声明的属性会引用它，共享同一份数据；
- 类似地，用 `@StateObject` 声明的属性是数据来源，其它用 `@ObservedObject` 声明的属性会引用它，也共享数据。

那么，`@StateObject` 和 `@ObservedObject` 之间到底有什么区别？它们保存的数据又是在什么时候会被清除呢？接下来让我们揭开这个谜团。

## 定义被观察的数据类型 DiceObject

```swift
class DiceObject: ObservableObject {
    @Published var value = 1

    func roll() {
        value = Int.random(in: 1...6)
    }
}
```

## 当视图重新生成时，@ObservedObject 保存的数据不会保留之前的状态

下面是用于显示骰子的 DiceView，点击 “Play” 按钮会随机出点数：

```swift
struct DiceView: View {
    @ObservedObject var diceObject = DiceObject()

    var body: some View {
        VStack {
            Image(systemName: "die.face.\(diceObject.value).fill")
                .resizable()
                .scaledToFit()
            Button("Play") {
                diceObject.roll()
            }
        }
    }
}
```

然后在 GameView 中使用 DiceView，点击 “Random Background Color” 按钮会改变背景颜色：

```swift
struct GameView: View {
    @State private var color = Color.white

    var body: some View {
        ZStack {
            color
                .ignoresSafeArea()
            VStack {
                DiceView()
                Button("Random Background Color") {
                    color = Color(
                        red: .random(in: 0...1),
                        green: .random(in: 0...1),
                        blue: .random(in: 0...1)
                    )
                }
            }
            .font(.title)
            .padding()
        }
    }
}

struct GameView_Previews: PreviewProvider {
    static var previews: some View {
        GameView()
    }
}
```

运行 App 后，先点击“Play”随机出一个骰子点数：

![骰子初始值](https://miro.medium.com/v2/resize:fit:992/1*63md__t_ld26ZJVHr0x1UQ.png)

再点击“Random Background Color”，背景颜色变了，但骰子点数却重置为 1，这是为什么呢？

![骰子值重置](https://miro.medium.com/v2/resize:fit:1008/1*FzN6PRgIEnthkqrWDd5LIw.png)

原因如下：

- GameView 的 `color` 状态改变；
- 触发 GameView 的 body 重新计算；
- 导致 DiceView 被重新构建；
- DiceView 中的 `diceObject` 也重新初始化，回到默认值 `1`。

这说明，当 View 因更新而重新创建时，`@ObservedObject` 并不会保留旧的对象或数据。

## @State 和 @StateObject 属性存储的数据会持续存在，直到视图生命周期结束

如果我们希望视图在重新构建时仍保留之前的数据，就需要使用 `@State` 或 `@StateObject`。它们的数据会存储在 SwiftUI 管理的外部空间中，不会随着视图 body 的重建而消失。只要视图的生命周期没有结束，数据就会持续存在。

我们将 DiceView 中的 `diceObject` 改为 `@StateObject`：

```swift
struct DiceView: View {
    @StateObject var diceObject = DiceObject()

    var body: some View {
        VStack {
            Image(systemName: "die.face.\(diceObject.value).fill")
                .resizable()
                .scaledToFit()
            Button("Play") {
                diceObject.roll()
            }
        }
    }
}
```

再次运行 App，点击 “Play” 改变骰子点数，再点击 “Random Background Color”，你会发现骰子点数保持不变。这说明：即便 DiceView 被重新生成，`@StateObject` 中的数据依然存在，并未重置。

![StateObject 持续存在](https://miro.medium.com/v2/resize:fit:1012/1*T82K2XIR6tK9L_ZWK2Kr0g.png)

根据 Apple 在 WWDC 2021 演讲《Demystify SwiftUI》的说明，视图的生命周期取决于其 identity（标识）。只要 identity 没有改变，SwiftUI 会认为是“同一个视图”，不会清除它。

![WWDC 图示](https://miro.medium.com/v2/resize:fit:1400/1*pXc9lxYe4jHkW7BNjlAvZA.png)

> [Demystify SwiftUI — WWDC 2021 视频地址](https://developer.apple.com/videos/play/wwdc2021/10022/?source=post_page-----ffd4982fcece)

在刚才的例子中，GameView 更新后 DiceView 被重建，但 identity 没变，因此还是同一个 DiceView，里面的 `@StateObject` 没有被清除。

## 视图在哪些情况下生命周期会结束？

有两种情形会让 SwiftUI 视图“死亡”（结束生命周期），从而清除 `@State` 和 `@StateObject`：

### 1. 从界面上被移除
### 2. identity（标识）发生改变

下面我们通过具体例子来说明这两种情况。

...（后续略，可继续添加）



## 当视图从界面上移除时，其生命周期结束，@StateObject 数据将被清除

我们使用一个 `showDice` 布尔值控制 DiceView 是否显示：

```swift
struct GameView: View {
    @State private var color = Color.white
    @State private var showDice = true

    var body: some View {
        ZStack {
            color
                .ignoresSafeArea()
            VStack {
                if showDice {
                    DiceView()
                }
                Button("Random Background Color") {
                    color = Color(
                        red: .random(in: 0...1),
                        green: .random(in: 0...1),
                        blue: .random(in: 0...1)
                    )
                }
                Button("\(showDice ? "hide" : "show") dice") {
                    showDice.toggle()
                }
            }
            .font(.title)
            .padding()
        }
    }
}
```

操作流程如下：

1. 点击“Play”，然后点击“Random Background Color”：

![操作前](https://miro.medium.com/v2/resize:fit:1032/1*Y1uSZObvzzc95KA2lyRFXw.png)

2. 点击“hide dice”，此时 `showDice = false`，DiceView 被从界面移除，生命周期结束，`diceObject` 数据被清除：

![移除后](https://miro.medium.com/v2/resize:fit:1016/1*quDpmPsjbAFuAJ-UUOYinA.png)

3. 再次点击“show dice”，`showDice = true`，DiceView 被重新创建，`diceObject` 是全新对象，点数恢复初始值 1：

![恢复后](https://miro.medium.com/v2/resize:fit:988/1*qQxTziiJKTTj6s6h5_J_nw.png)

## 当视图的 identity 改变时，其生命周期结束，数据也会被清除

如果通过 `.id(...)` 强制改变 View 的 identity，SwiftUI 会认为它是全新的 View，从而销毁旧的视图与状态：

```swift
struct GameView: View {
    @State private var color = Color.white
    @State private var diceID = UUID()

    var body: some View {
        ZStack {
            color
                .ignoresSafeArea()
            VStack {
                DiceView()
                    .id(diceID)
                Button("Random Background Color") {
                    color = Color(
                        red: .random(in: 0...1),
                        green: .random(in: 0...1),
                        blue: .random(in: 0...1)
                    )
                }
                Button("change dice ID") {
                    diceID = UUID()
                }
            }
            .font(.title)
            .padding()
        }
    }
}
```

步骤：

1. 点击“Play”，然后点击“Random Background Color”：

![改变前](https://miro.medium.com/v2/resize:fit:980/1*5WRmsrnmv1Z1qI0F9d334w.png)

2. 点击“change dice ID”，DiceView 的 `.id` 改变，视图重新生成，`diceObject` 数据被重置为初始值 1：

![改变后](https://miro.medium.com/v2/resize:fit:992/1*I0pGCACM5fWvEapSnOV_rw.png)

## @State 的示例说明

前面的示例我们主要使用了 `@StateObject`。其实 `@State` 也有类似的生命周期行为——它同样依赖视图是否被移除或 identity 是否变化来判断是否保留数据。

你可以用同样的方式验证 `@State` 在不同生命周期下的行为。

---

**总结：**

- 使用 `@ObservedObject` 时，数据不会在视图重建中保留；
- 使用 `@StateObject` 可保持数据，只要视图 identity 不变；
- 视图被移除或 `.id(...)` 改变会终止其生命周期；
- 同样的生命周期机制也适用于 `@State`。

掌握这些细节，有助于你在 SwiftUI 中更好地管理状态数据，避免不必要的状态丢失与初始化。

