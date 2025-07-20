# 使用SwiftUI构建动态图片网格视图

在移动应用开发中，图片网格是展示内容的一种常见方式，特别是在相册、社交媒体或电商类应用中。本文将介绍如何使用SwiftUI构建一个灵活、动态的图片网格视图，用户可以自由调整网格的列数。

## 项目概述

我们创建一个包含以下功能的图片网格视图：
- 显示正方形图片网格
- 支持1-6列动态调整
- 平滑的列数切换动画
- 自适应不同屏幕尺寸

## 实现步骤

### 1. 基础结构搭建

首先，我们创建基本的视图结构：

```swift
struct ContentView: View {
    var body: some View {
        ImageGridView()
    }
}
```

### 2. 图片网格容器视图

`ImageGridView`作为容器视图，负责管理图片数据：

```swift
struct ImageGridView: View {
    let sampleImages: [String] = (1...11).map({"\($0)"})
    
    var body: some View {
        NavigationView {
            SquareImageGrid(items: sampleImages)
                .navigationTitle("图片网格")
        }
    }
}
```

这里我们使用1到11的数字作为示例图片名称，实际应用中可替换为真实的图片URL或资源名称。

### 3. 核心网格实现

`SquareImageGrid`是实现动态网格的核心组件：

```swift
struct SquareImageGrid: View {
    let items: [String]  // 图片URL或资源名称数组
    @State private var columns: Int = 3  // 默认3列
    
    var body: some View {
        GeometryReader { geometry in
            VStack {
                // 列数选择器
                Picker("列数", selection: $columns.animation(.easeInOut)) {
                    ForEach(1..<7) { num in
                        Text("\(num)列").tag(num)
                    }
                }
                .pickerStyle(SegmentedPickerStyle())
                .padding(.vertical)
                
                // 自适应网格
                ScrollView {
                    LazyVGrid(
                        columns: Array(repeating: GridItem(.flexible(), spacing: 2), count: columns),
                        spacing: 2
                    ) {
                        ForEach(items, id: \.self) { item in
                            Color.clear
                                .aspectRatio(1, contentMode: .fit)
                                .overlay(
                                    Image(item)
                                        .resizable()
                                        .scaledToFill()
                                )
                                .clipped()
                                .background(Color.gray.opacity(0.2))
                        }
                    }
                }
            }
        }
    }
}
```

### 4. 关键实现细节

#### 动态列数调整

通过`@State`属性`columns`存储当前列数，并使用`Picker`让用户选择1-6列：

```swift
@State private var columns: Int = 3

Picker("列数", selection: $columns.animation(.easeInOut)) {
    ForEach(1..<7) { num in
        Text("\(num)列").tag(num)
    }
}
```

`.animation(.easeInOut)`修饰符使列数变化时有平滑的过渡效果。

#### 正方形图片处理

使用以下技巧确保每个网格项为正方形：

```swift
Color.clear
    .aspectRatio(1, contentMode: .fit)
    .overlay(
        Image(item)
            .resizable()
            .scaledToFill()
    )
    .clipped()
```

1. `Color.clear`创建透明背景
2. `.aspectRatio(1, contentMode: .fit)`确保宽高比为1:1
3. `overlay`添加图片并填充整个空间
4. `.clipped()`裁剪超出部分

#### 网格布局

使用`LazyVGrid`创建垂直网格：

```swift
LazyVGrid(
    columns: Array(repeating: GridItem(.flexible(), spacing: 2), count: columns),
    spacing: 2
)
```

`GridItem(.flexible())`使每列等宽，`spacing`参数控制行列间距。

### 5. 预览设置

```swift
#Preview {
    ContentView()
}
```

## 扩展与优化

1. **图片加载优化**：实际应用中应使用异步图片加载库（如Kingfisher或Nuke）处理网络图片
2. **占位图**：加载过程中显示占位图
3. **图片点击**：添加`onTapGesture`实现点击放大等功能
4. **性能优化**：对于大量图片，考虑实现分页加载

## 总结

本文介绍了如何使用SwiftUI构建一个动态可调的图片网格视图。通过结合`LazyVGrid`、`GeometryReader`和状态管理，我们创建了一个灵活、响应式的布局组件。这种实现方式简洁高效，充分利用了SwiftUI的声明式语法和自动布局能力。

该组件可以轻松集成到各种应用中，只需替换图片数据源即可快速实现美观的图片展示功能。