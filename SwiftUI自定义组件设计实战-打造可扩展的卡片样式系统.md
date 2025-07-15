# SwiftUI 自定义组件设计实战-打造可扩展的卡片样式系统

在 SwiftUI 中，良好的组件设计不仅要关注功能实现，更应关注其可扩展性、灵活性以及 API 的易用性。
本文将从零构建一个支持多样式切换的卡片组件系统，逐步讲解协议抽象、类型擦除、环境变量注入、视图修饰符等核心概念的实际应用。

## ✨ 项目目标

我们希望实现这样一个组件系统：

- 使用 `CardView(title:description:)` 构建基础卡片；
- 使用 `.cardStyle(...)` 修改样式；
- 同时支持使用结构体样式或简洁调用方式；
- 卡片样式与结构解耦，可灵活扩展。

## 🧱 Step 1：定义样式协议 CardStyle

```swift
protocol CardStyle {
    associatedtype Body: View
    func makeBody(configuration: CardStyleConfiguration) -> Body
}
```

这段代码定义了卡片样式协议 `CardStyle`。
其中：

- `associatedtype Body: View` 允许每种样式定义自己的视图类型，提供最大的灵活性；
- `makeBody(configuration:)` 方法接收配置对象，返回样式渲染的视图；

通过这个协议，我们可以将卡片的外观设计从主组件中分离出来，实现样式与结构解耦。这种方式与 SwiftUI 原生的 `ButtonStyle` 和 `ToggleStyle` 接口如出一辙，是一种非常主流的样式协议设计方法。

## 📦 Step 2：卡片样式的上下文配置 CardStyleConfiguration

```swift
struct CardStyleConfiguration {
    let title: String
    let description: String
}
```

样式配置对象用于传递视图数据，如标题与说明文字。这一设计将组件数据与样式实现解耦，允许样式自由读取所需数据而不依赖具体组件。`CardStyleConfiguration` 就是承载这些输入的配置容器，类似 SwiftUI 中的 `ButtonStyleConfiguration`。这样，样式就可以只关注如何展示这些内容，而不用关心组件本身的数据逻辑。这种设计也为未来的扩展打下基础，例如你可以在这里加入 `icon`, `action`, `colorScheme` 等字段。

## 🌀 Step 3：引入类型擦除 AnyCardStyle

```swift
struct AnyCardStyle: CardStyle {
    private let _makeBody: (CardStyleConfiguration) -> AnyView

    init<S: CardStyle>(_ style: S) {
        _makeBody = { AnyView(style.makeBody(configuration: $0)) }
    }

    func makeBody(configuration: CardStyleConfiguration) -> some View {
        _makeBody(configuration)
    }
}
```

为什么需要 `AnyCardStyle`？

- Swift 协议中使用 `associatedtype` 后就不能直接作为属性存储；
- 但我们希望将任意样式存储到 `@Environment` 变量中；

通过类型擦除（type erasure），我们用 `AnyCardStyle` 包裹任意实现了 `CardStyle` 协议的结构体，将其转换为统一返回 `AnyView` 的包装器，使它具备良好的传递能力。

- `_makeBody` 是一个闭包，内部执行具体样式逻辑；
- 返回值强制转换为 `AnyView`，以统一输出类型；
- 这为组件使用 `.environment(\.cardStyle, ...)` 提供可能。

这是 SwiftUI 中高度通用的设计技巧，例如 `AnyView`, `AnyTransition`, `AnyShape` 都基于类似原理。

## 🎨 Step 4：实现具体样式

### 4.1 默认样式

```swift
struct DefaultCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        VStack {
            Text(configuration.title).font(.title)
            Text(configuration.description).font(.subheadline)
        }
        .padding()
        .background(Color.gray.opacity(0.1))
        .cornerRadius(12)
    }
}
```

默认样式采用简单的背景色与圆角修饰，是一种中性的呈现方式。

- 使用 `VStack` 实现上下排列；
- 设置字体样式以区分层级；
- 统一添加 padding 与背景；
- 使用 cornerRadius 实现视觉统一。

你可以把这个样式当作系统 fallback 样式。

### 4.2 成功样式

```swift
struct SuccessCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        VStack {
            Text(configuration.title).font(.headline)
            Text(configuration.description).font(.subheadline)
        }
        .padding()
        .background(Color.green.opacity(0.1))
        .overlay(RoundedRectangle(cornerRadius: 12).stroke(Color.green, lineWidth: 2))
    }
}
```

该样式更具视觉反馈性，适用于“操作成功”这类场景。

- 使用绿色背景提示成功状态；
- 添加 `overlay` 边框强化样式层次；
- 字体风格偏向轻量、系统默认。

## 🧩 Step 5：实现支持样式注入的组件 CardView

```swift
struct CardView: View {
    let title: String
    let description: String
    @Environment(\.cardStyle) private var style

    var body: some View {
        style.makeBody(configuration: .init(title: title, description: description))
    }
}
```

这是核心组件部分：
- `@Environment(\.cardStyle)` 读取样式上下文；
- 将传入的数据封装成 `CardStyleConfiguration`，再传给样式；
- `CardView` 自身并不关心样式如何渲染，符合“职责单一”原则。

这种设计使 `CardView` 可复用性大大增强，也便于测试与扩展。

## 🔗 Step 6：添加样式修饰符 .cardStyle

```swift
extension View {
    func cardStyle<S: CardStyle>(_ style: S) -> some View {
        environment(\.cardStyle, AnyCardStyle(style))
    }
}
```

这个扩展让我们可以像设置字体、颜色一样设置卡片样式：

```swift
CardView(title: "成功", description: "已完成任务")
    .cardStyle(SuccessCardStyle())
```

背后的原理是使用 `environment(\.cardStyle, ...)` 将样式注入组件上下文。
通过这个修饰符，我们实现了优雅的 DSL 风格使用方式。

## 🧪 Step 7：演示用法视图 CardStyleDemoView

```swift
struct CardStyleDemoView: View {
    var body: some View {
        VStack(spacing: 16) {
            CardView(title: "默认样式", description: "这是默认卡片")
            CardView(title: "成功样式", description: "操作成功")
                .cardStyle(SuccessCardStyle())
            CardView(title: "警告样式", description: "请注意操作")
                .cardStyle(WarningCardStyle())
        }
        .padding()
    }
}
```

这个视图用于展示三种不同样式的卡片：
- 第一张使用系统默认样式；
- 第二张调用 `.cardStyle(SuccessCardStyle())` 显示成功反馈；
- 第三张可拓展为 `.cardStyle(WarningCardStyle())` 实现更多状态。

通过统一的接口调用方式，开发者可以轻松集成不同视觉语言的卡片 UI。

## 🧠 总结与启发

这套卡片组件设计的关键在于：

- 利用协议抽象样式结构；

- 使用类型擦除解决泛型限制；
- 通过环境变量注入样式上下文；
- 实现语义化修饰符 `.cardStyle(...)`，调用方式现代直观；
- 视图职责单一，CardView 专注结构，样式交由外部配置；
- 扩展性强，只需新增一个样式结构体，即可引入新的样式。

这种设计模式不仅适用于卡片，几乎可以迁移至所有 UI 控件组件中，构建完整的组件系统。

## 📦 附源码（可整理为 .swift 文件）

你可以将下面代码保存为 `CardStyleDemoView.swift`：

```swift

import SwiftUI

// 卡片组件使用示例
struct CardStyleDemoView: View {
    var body: some View {
        VStack {
            CardView(title: "普通消息")
                .cardStyle(.default)
            
            CardView(title: "错误消息")
                .cardStyle(.danger)
            
            CardView(title: "成功消息")
                .cardStyle(.success)
            
            // 实例方式
            CardView(title: "警告消息")
                .cardStyle(WarningCardStyle())
        }
        .padding()
    }
}

// 扩展卡片样式
struct WarningCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        Text(configuration.title)
            .padding()
            .foregroundColor(.white)
            .frame(maxWidth: .infinity, alignment: .leading)
            .background(Color.orange.opacity(0.5))
            .cornerRadius(10)
    }
}

// 预览
#Preview {
    CardStyleDemoView()
}

// 定义卡片样式协议
protocol CardStyle {
    associatedtype Body: View
    func makeBody(configuration: CardStyleConfiguration) -> Body
}

// 定义卡片样式数据模型
struct CardStyleConfiguration {
    let title: String
}

extension View {
    /// 支持 `.cardStyle(.success)`
    func cardStyle(_ styleType: CardStyleType) -> some View {
        self.environment(\.cardStyle, styleType.style)
    }

    /// 支持 `.cardStyle(MyCustomStyle())`
    func cardStyle<S: CardStyle>(_ style: S) -> some View {
        self.environment(\.cardStyle, AnyCardStyle(style))
    }
}

// 类型擦除
struct AnyCardStyle: CardStyle {
    private let _makeBody: (CardStyleConfiguration) -> AnyView
    
    init<S: CardStyle>(_ style: S) {
        _makeBody = { AnyView(style.makeBody(configuration: $0)) }
    }

    func makeBody(configuration: CardStyleConfiguration) -> some View {
        _makeBody(configuration)
    }
}

// 环境支持
private struct CardStyleKey: EnvironmentKey {
    static let defaultValue = AnyCardStyle(DefaultCardStyle())
}

extension EnvironmentValues {
    var cardStyle: AnyCardStyle {
        get { self[CardStyleKey.self] }
        set { self[CardStyleKey.self] = newValue }
    }
}

// 卡片组件样式枚举
enum CardStyleType {
    case `default`
    case danger
    case success

    var style: AnyCardStyle {
        switch self {
        case .default: AnyCardStyle(DefaultCardStyle())
        case .danger: AnyCardStyle(DangerCardStyle())
        case .success: AnyCardStyle(SuccessCardStyle())
        }
    }
}

// 定义卡片组件
struct CardView: View {
    let title: String
    @Environment(\.cardStyle) private var style
    
    var body: some View {
        style.makeBody(configuration: CardStyleConfiguration(title: title))
    }
}

// 默认卡片组件样式实现
struct DefaultCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        Text(configuration.title)
            .padding()
            .foregroundColor(.white)
            .frame(maxWidth: .infinity, alignment: .leading)
            .background(Color.blue.opacity(0.5))
            .cornerRadius(10)
    }
}

// Danger 卡片组件样式实现
struct DangerCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        Text(configuration.title)
            .padding()
            .foregroundColor(.white)
            .frame(maxWidth: .infinity, alignment: .leading)
            .background(Color.red.opacity(0.5))
            .cornerRadius(10)
    }
}

// Success 卡片组件样式实现
struct SuccessCardStyle: CardStyle {
    func makeBody(configuration: CardStyleConfiguration) -> some View {
        Text(configuration.title)
            .padding()
            .foregroundColor(.white)
            .frame(maxWidth: .infinity, alignment: .leading)
            .background(Color.green.opacity(0.5))
            .cornerRadius(10)
    }
}
```
