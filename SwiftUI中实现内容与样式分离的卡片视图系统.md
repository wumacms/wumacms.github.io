# SwiftUI 中实现内容与样式分离的卡片视图系统

在 SwiftUI 中，良好的组件设计不仅要求代码结构清晰，更鼓励**内容与样式的彻底分离**。这意味着使用者只需要专注于“写内容”，而不必重复书写样式代码。

本文将带你一步步实现一个灵活、可扩展的卡片视图系统，支持如下优雅的使用方式：

```swift
CardView(title: "科技新闻", description: "最新的人工智能突破")
    .cardStyle(.modern)

CardView(title: "文学赏析", description: "古典诗词的现代解读")
    .cardStyle(.classic)
```

## 🎯 目标

- 内容样式分离
- 支持多个风格 `.modern` / `.classic` / `.default`
- 使用者无需接触样式代码
- 样式定义可扩展、可替换

## 🧱 实现思路

实现思路主要包含以下几部分：

1. 定义 `CardStyle` 协议
2. 使用类型擦除 `AnyCardStyle`
3. 利用 SwiftUI 的 `Environment` 机制传递样式
4. 使用 `enum` 枚举来优雅选择样式
5. 定义 `.cardStyle(.xxx)` 修饰符来应用样式

## ✅ 完整代码

将以下代码保存为 `CardStyleDemo.swift`，即可在项目或 Swift Playground 中运行。

```swift
// ...此处省略，下一段完整代码中包含...
```

### 🔁 请参考后附完整代码。

## 💡 拓展思路

- ✅ 支持自定义样式注入 `.cardStyle(MyCustomStyle())`
- ✅ 使用 `.environment(\.cardStyle)` 全局设置默认风格
- ✅ 配合 `PreferenceKey` 实现响应式动态样式切换
- ✅ 也可用于 ListCell、Button、Modal 等组件样式扩展

## 🧾 总结

本文展示了如何用 SwiftUI 的协议、环境值、泛型和枚举等特性构建一个灵活、声明式、可扩展的**卡片样式系统**。它体现了 SwiftUI 哲学中“组合优先、声明为王”的核心设计思想，是你封装组件和构建设计系统的重要一环。

---

## 📎 附录：完整代码（CardStyleDemo.swift）

```swift
import SwiftUI

protocol CardStyle {
    associatedtype Body: View
    func makeBody(configuration: CardStyleConfiguration) -> Body
}

struct CardStyleConfiguration {
    let title: String
    let description: String
}

struct AnyCardStyle: CardStyle {
    private let _makeBody: (CardStyleConfiguration) -> AnyView

    init<S: CardStyle>(_ style: S) {
        _makeBody = { AnyView(style.makeBody(configuration: $0)) }
    }

    func makeBody(configuration: CardStyleConfiguration) -> some View {
        _makeBody(configuration)
    }
}

private struct CardStyleKey: EnvironmentKey {
    static let defaultValue = AnyCardStyle(DefaultCardStyle())
}

extension EnvironmentValues {
    var cardStyle: AnyCardStyle {
        get { self[CardStyleKey.self] }
        set { self[CardStyleKey.self] = newValue }
    }
}

enum CardStyleType {
    case `default`
    case modern
    case classic

    var style: AnyCardStyle {
        switch self {
        case .default: AnyCardStyle(DefaultCardStyle())
        case .modern: AnyCardStyle(ModernCardStyle())
        case .classic: AnyCardStyle(ClassicCardStyle())
        }
    }
}

extension View {
    func cardStyle(_ styleType: CardStyleType) -> some View {
        self.environment(\.cardStyle, styleType.style)
    }
}

struct CardView: View {
    let title: String
    let description: String

    @Environment(\.cardStyle) private var style

    var body: some View {
        style.makeBody(configuration: .init(title: title, description: description))
    }
}

struct DefaultCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        VStack(alignment: .leading, spacing: 6) {
            Text(configuration.title).font(.headline)
            Text(configuration.description).font(.subheadline)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(8)
        .shadow(radius: 2)
    }
}

struct ModernCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(configuration.title)
                .font(.title)
                .bold()
                .foregroundColor(.white)
            Text(configuration.description)
                .font(.body)
                .foregroundColor(.white.opacity(0.8))
        }
        .padding()
        .background(Color.blue)
        .cornerRadius(12)
        .shadow(radius: 5)
    }
}

struct ClassicCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(configuration.title)
                .font(.headline)
                .foregroundColor(.black)
            Divider()
            Text(configuration.description)
                .font(.footnote)
                .foregroundColor(.gray)
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 8)
                .stroke(Color.gray)
        )
    }
}

struct ContentView: View {
    var body: some View {
        VStack(spacing: 16) {
            CardView(title: "科技新闻", description: "最新的人工智能突破")
                .cardStyle(.modern)

            CardView(title: "文学赏析", description: "古典诗词的现代解读")
                .cardStyle(.classic)

            CardView(title: "默认样式", description: "未指定样式时使用默认卡片样式")
        }
        .padding()
    }
}

#Preview {
    ContentView()
}
```
