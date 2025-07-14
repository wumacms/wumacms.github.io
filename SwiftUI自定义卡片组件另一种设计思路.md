# SwiftUI自定义卡片组件另一种设计思路



## 1. 定义卡片样式协议和具体样式

```swift
// 卡片配置结构体
struct CardConfiguration {
    let title: String
    let description: String
}

// 卡片样式协议
protocol CardStyle {
    associatedtype Body: View
    func makeBody(configuration: CardConfiguration) -> Body
}

// 现代卡片样式实现
struct ModernCardStyle: CardStyle {
    func makeBody(configuration: CardConfiguration) -> some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(configuration.title)
                .font(.headline)
                .foregroundColor(.primary)
            
            Text(configuration.description)
                .font(.subheadline)
                .foregroundColor(.secondary)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(10)
        .shadow(color: Color.black.opacity(0.1), radius: 5, x: 0, y: 2)
    }
}

// 默认卡片样式
struct DefaultCardStyle: CardStyle {
    func makeBody(configuration: CardConfiguration) -> some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(configuration.title)
                .font(.title3)
                .bold()
            
            Divider()
            
            Text(configuration.description)
                .font(.body)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(8)
    }
}
```

## 2. 定义 CardView 和样式修饰符

```swift
struct CardView: View {
    private let configuration: CardConfiguration
    private var style: AnyCardStyle = AnyCardStyle(DefaultCardStyle())
    
    init(title: String, description: String) {
        self.configuration = CardConfiguration(title: title, description: description)
    }
    
    var body: some View {
        style.makeBody(configuration: configuration)
    }
}

// 类型擦除包装器
struct AnyCardStyle: CardStyle {
    private let _makeBody: (CardConfiguration) -> AnyView
    
    init<S: CardStyle>(_ style: S) {
        self._makeBody = { configuration in
            AnyView(style.makeBody(configuration: configuration))
        }
    }
    
    func makeBody(configuration: CardConfiguration) -> some View {
        _makeBody(configuration)
    }
}

// 添加 cardStyle 修饰符
extension CardView {
    func cardStyle<S: CardStyle>(_ style: S) -> some View {
        var view = self
        view.style = AnyCardStyle(style)
        return view
    }
}
```

## 3. 使用示例

现在你可以按照你想要的方式使用卡片组件了：

```swift
// 使用默认样式
CardView(title: "天气预报", description: "明天晴朗，气温25℃")

// 使用现代样式
CardView(title: "科技新闻", description: "最新的人工智能突破")
    .cardStyle(ModernCardStyle())

// 可以轻松添加其他样式
struct RoundedCardStyle: CardStyle {
    func makeBody(configuration: CardConfiguration) -> some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(configuration.title)
                .font(.title2)
                .bold()
            
            Text(configuration.description)
                .font(.body)
        }
        .padding()
        .background(Color.blue)
        .foregroundColor(.white)
        .cornerRadius(15)
    }
}

// 使用自定义样式
CardView(title: "特别通知", description: "系统将于今晚进行维护")
    .cardStyle(RoundedCardStyle())
```

## 优势说明

这种实现方式有以下优点：

1. **更直观的初始化**：直接通过 `CardView(title:description:)` 初始化，更符合 SwiftUI 的惯用语法
2. **默认样式**：内置了默认样式，不需要强制指定样式
3. **灵活扩展**：可以轻松添加新的卡片样式
4. **类型安全**：利用 Swift 的泛型和协议保证了类型安全

这种设计模式类似于 SwiftUI 中内置的 `Button` 和 `ButtonStyle` 的关系，提供了清晰的使用接口和灵活的定制能力。
