
# SwiftUI 中视图的生命周期详解

SwiftUI 中，视图生命周期不同于 UIKit 的 `UIViewController` 生命周期方法（如 `viewDidLoad` 等）。它以 **声明式设计与状态驱动** 为核心，由系统根据状态自动管理视图的创建、更新与销毁。

---

## 🔁 1. SwiftUI 视图的本质

- SwiftUI 视图是 **值类型结构体 (`struct`)**。
- 状态变化会导致 **视图结构的重建**。
- 系统通过 **diffing 算法** 比较新旧视图并更新 UI。

---

## 🧩 2. 生命周期组成阶段

| 阶段 | 描述 |
|------|------|
| `init()` | 初始化视图数据，状态变化会触发重建。 |
| `body` 计算 | 每次状态改变都会调用以重新生成 UI。 |
| 状态依赖更新 | 使用 `@State`、`@Binding` 等时，状态改变会触发重建。 |
| `onAppear()` | 视图首次或再次显示时触发。 |
| `onDisappear()` | 视图被移出屏幕时触发。 |

---

## 📦 3. 生命周期修饰符

### `onAppear(perform:)`

```swift
Text("Hello")
    .onAppear {
        print("视图出现")
    }
```

- 在视图进入屏幕时调用。

### `onDisappear(perform:)`

```swift
Text("Hello")
    .onDisappear {
        print("视图消失")
    }
```

- 在视图从屏幕移除时调用。

---

## 🧠 4. 示例演示

```swift
struct DemoView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("点击次数：\(count)")
                .onAppear {
                    print("Text 出现")
                }

            Button("点击增加") {
                count += 1
            }
        }
    }
}
```

---

## 💬 5. 生命周期与状态属性的关系

| 属性 | 生命周期特征 |
|------|--------------|
| `@State` | 本地状态，状态改变时视图重建。 |
| `@Binding` | 依赖父视图状态，生命周期由父视图控制。 |
| `@ObservedObject` | 外部状态对象，适合短生命周期使用。 |
| `@StateObject` | 本视图持有的状态对象，适合长生命周期使用。 |
| `@EnvironmentObject` | 来自外部环境注入，生命周期独立于视图。 |

---

## ⚠️ 6. 注意事项

- `init()` 不建议执行副作用。
- `onAppear()` 可能在多种场景重复调用。
- `body` 被频繁调用是设计预期。
- 子视图的 `onAppear` 需单独定义，不会继承。

---

## 🧪 7. 调试生命周期

```swift
struct LoggingView: View {
    init() {
        print("初始化 LoggingView")
    }

    var body: some View {
        print("构建 body")
        return Text("日志测试")
            .onAppear { print("出现") }
            .onDisappear { print("消失") }
    }
}
```

---

## ✅ 总结

| 生命周期事件 | 描述 |
|--------------|------|
| `init()` | 初始化视图结构 |
| `body` | 根据状态重新渲染视图 |
| `onAppear()` | 视图出现时触发 |
| `onDisappear()` | 视图移除时触发 |

SwiftUI 的生命周期以**声明式与响应式状态管理**为核心，掌握生命周期有助于更好地组织副作用逻辑与性能优化。

