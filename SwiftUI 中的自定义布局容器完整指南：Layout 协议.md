# SwiftUI 中的自定义布局容器完整指南：Layout 协议

SwiftUI 的 `Layout` 协议是在 WWDC22 中引入的，它允许开发者创建完全自定义的布局容器，提供了比 `ZStack`、`HStack` 和 `VStack` 等内置布局更精细的控制能力。

## 基本概念

`Layout` 协议要求实现两个主要方法：

1. `sizeThatFits(proposal:subviews:cache:)` - 计算容器的理想大小
2. `placeSubviews(in:proposal:subviews:cache:)` - 确定子视图的位置

## 创建自定义布局

### 1. 实现基本布局

```swift
struct MyCustomLayout: Layout {
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) -> CGSize {
        // 计算并返回布局的尺寸
        let sizes = subviews.map { $0.sizeThatFits(proposal) }
        let maxWidth = sizes.map { $0.width }.max() ?? 0
        let totalHeight = sizes.map { $0.height }.reduce(0, +)
        return CGSize(width: maxWidth, height: totalHeight)
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) {
        // 放置子视图
        var y = bounds.minY
        for subview in subviews {
            let size = subview.sizeThatFits(proposal)
            subview.place(
                at: CGPoint(x: bounds.midX, y: y),
                anchor: .top,
                proposal: proposal
            )
            y += size.height
        }
    }
}
```

### 2. 使用自定义布局

```swift
MyCustomLayout {
    Text("第一个视图")
    Text("第二个视图")
    Text("第三个视图")
}
.frame(width: 200, height: 300)
```

## 高级特性

### 1. 布局缓存 (Cache)

为了提高性能，可以实现缓存机制：

```swift
struct CachedLayout: Layout {
    struct CacheData {
        var sizes: [CGSize]
    }
    
    func makeCache(subviews: Subviews) -> CacheData {
        CacheData(sizes: subviews.map { $0.sizeThatFits(.unspecified) })
    }
    
    func updateCache(_ cache: inout CacheData, subviews: Subviews) {
        cache.sizes = subviews.map { $0.sizeThatFits(.unspecified) }
    }
    
    // 其余实现...
}
```

### 2. 响应式布局

可以根据环境值调整布局：

```swift
struct AdaptiveLayout: Layout {
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        // 根据环境或提案调整布局
    }
    
    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        // 动态放置子视图
    }
}
```



## 实际应用示例

### 1. 定义环形布局

```swift
struct CircularLayout: Layout {
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        proposal.replacingUnspecifiedDimensions()
    }
    
    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        let radius = min(bounds.width, bounds.height) / 3.0
        let angle = Angle.degrees(360.0 / Double(subviews.count)).radians
        
        for (index, subview) in subviews.enumerated() {
            let currentAngle = angle * Double(index)
            let x = bounds.midX + radius * cos(currentAngle)
            let y = bounds.midY + radius * sin(currentAngle)
            
            subview.place(
                at: CGPoint(x: x, y: y),
                anchor: .center,
                proposal: .unspecified
            )
        }
    }
}
```

### 2. 使用环形布局

```swift
// 使用之前定义的 CircularLayout
CircularLayout {
    ForEach(0..<6) { index in
        Circle()
            .frame(width: 40, height: 40)
            .foregroundColor(.blue)
            .overlay(Text("\(index + 1)").foregroundColor(.white))
    }
}
.frame(width: 300, height: 300)
.border(Color.gray)
```



### 3. 定义瀑布流布局

```swift
struct WaterfallLayout: Layout {
    var columns: Int
    var spacing: CGFloat
    
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        let width = proposal.width ?? 0
        let columnWidth = (width - CGFloat(columns - 1) * spacing) / CGFloat(columns)
        var columnHeights = Array(repeating: CGFloat(0), count: columns)
        
        for subview in subviews {
            let size = subview.sizeThatFits(
                ProposedViewSize(width: columnWidth, height: nil)
            )
            let shortestColumn = columnHeights.enumerated()
                .min(by: { $0.element < $1.element })!
                .offset
            columnHeights[shortestColumn] += size.height + spacing
        }
        
        let height = columnHeights.max() ?? 0
        return CGSize(width: width, height: height)
    }
    
    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        let width = bounds.width
        let columnWidth = (width - CGFloat(columns - 1) * spacing) / CGFloat(columns)
        var columnHeights = Array(repeating: bounds.minY, count: columns)
        
        for subview in subviews {
            let size = subview.sizeThatFits(
                ProposedViewSize(width: columnWidth, height: nil)
            )
            let shortestColumn = columnHeights.enumerated()
                .min(by: { $0.element < $1.element })!
                .offset
            let x = bounds.minX + CGFloat(shortestColumn) * (columnWidth + spacing)
            let y = columnHeights[shortestColumn]
            
            subview.place(
                at: CGPoint(x: x + columnWidth / 2, y: y + size.height / 2),
                anchor: .center,
                proposal: ProposedViewSize(width: columnWidth, height: size.height)
            )
            
            columnHeights[shortestColumn] += size.height + spacing
        }
    }
}
```



### 4. 使用瀑布流布局

```swift
// 使用之前定义的 WaterfallLayout
let colors: [Color] = [.red, .green, .blue, .orange, .purple, .pink, .yellow]

WaterfallLayout(columns: 3, spacing: 8) {
    ForEach(0..<15) { index in
        let height = CGFloat.random(in: 50...200)
        RoundedRectangle(cornerRadius: 8)
            .fill(colors[index % colors.count])
            .frame(height: height)
            .overlay(Text("\(index + 1)").foregroundColor(.white))
    }
}
.padding()
```



## 动态调整布局

### 1. 根据条件切换布局

```swift
@State private var useVerticalLayout = false

var body: some View {
    VStack {
        Toggle("垂直布局", isOn: $useVerticalLayout)
        
        if useVerticalLayout {
            MyCustomLayout {
                contentViews()
            }
        } else {
            CircularLayout {
                contentViews()
            }
            .frame(width: 300, height: 300)
        }
    }
    .padding()
}

@ViewBuilder
func contentViews() -> some View {
    ForEach(1...5, id: \.self) { index in
        Text("项目 \(index)")
            .padding()
            .background(Color.blue.opacity(0.2))
            .cornerRadius(8)
    }
}
```

### 2. 响应环境变化

```swift
struct MyView: View {
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    
    var body: some View {
        let columns = horizontalSizeClass == .compact ? 2 : 4
        
        WaterfallLayout(columns: columns, spacing: 8) {
            ForEach(0..<20) { index in
                RoundedRectangle(cornerRadius: 8)
                    .fill(Color.random())
                    .frame(height: CGFloat.random(in: 50...200))
            }
        }
        .animation(.default, value: horizontalSizeClass)
        .padding()
    }
}
```

## 布局修饰符

你可以为自定义布局创建修饰符以便更优雅地使用：

```swift
extension View {
    func inCustomLayout() -> some View {
        MyCustomLayout {
            self
        }
    }
}

// 使用方式
Text("单独的项目").inCustomLayout()
```

## 组合多个布局

```swift
VStack {
    CircularLayout {
        ForEach(1...4, id: \.self) { index in
            Circle()
                .frame(width: 40)
                .overlay(Text("\(index)"))
        }
    }
    .frame(height: 200)
    
    MyCustomLayout {
        ForEach(5...8, id: \.self) { index in
            Capsule()
                .frame(width: 80, height: 30)
                .overlay(Text("\(index)"))
        }
    }
}
```

## 调试自定义布局

```swift
MyCustomLayout {
    ForEach(1...5, id: \.self) { index in
        Text("项目 \(index)")
            .padding()
            .background(Color.blue.opacity(0.2))
            .cornerRadius(8)
    }
}
.debugLayout() // 添加这个修饰符查看布局信息

// 定义调试修饰符
extension View {
    func debugLayout() -> some View {
        self.overlay(
            GeometryReader { proxy in
                Text("尺寸: \(Int(proxy.size.width))×\(Int(proxy.size.height))")
                    .font(.caption)
                    .foregroundColor(.white)
                    .padding(2)
                    .background(Color.black.opacity(0.7))
                    .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .bottomTrailing)
            }
        )
    }
}
```

## 注意事项

1. **尺寸提案**：自定义布局应该正确处理父视图提出的尺寸建议
2. **性能考虑**：对于大量子视图，确保布局计算高效
3. **动画支持**：布局变化默认有动画效果，可以通过 `.animation(nil)` 禁用
4. **动态内容**：确保布局能正确处理动态添加或删除的子视图

通过以上方法，你可以灵活地使用自定义布局容器，创建各种独特的界面排列方式。



## 最佳实践

1. **性能考虑**：在 `sizeThatFits` 和 `placeSubviews` 中进行高效计算
2. **响应提案**：正确处理父视图提出的尺寸建议
3. **灵活布局**：考虑子视图的不同尺寸需求
4. **使用缓存**：对于复杂计算，实现缓存机制提高性能
5. **测试不同设备**：确保布局在各种屏幕尺寸上都能正常工作

通过 `Layout` 协议，你可以创建几乎任何你能想象到的布局结构，从简单的自定义堆栈到复杂的响应式网格和环形布局。
