
# SwiftUI 中的 `.mask` 修饰符：原理、实战与进阶封装

在 SwiftUI 中，我们可以用 `.mask(_:)` 修饰符对任意视图进行遮罩处理，实现图形裁剪、渐变消隐、动态动画等高级视觉效果。本文将从 **原理讲解、实战示例到可复用封装** 全面剖析 `.mask` 的用法，助你深入掌握这一强大工具。

---

## 🧠 什么是 `.mask`

`.mask` 是 SwiftUI 提供的一个视图修饰符：

```swift
func mask<M>(_ mask: M) -> some View where M : View
```

它的作用是：

> **根据“遮罩视图”的透明度通道（Alpha 通道）来决定当前视图哪些部分可见，哪些部分被隐藏。**

- 遮罩部分 **透明（alpha = 0）** → 原图被隐藏  
- 遮罩部分 **不透明（alpha = 1）** → 原图被显示  
- 遮罩部分 **半透明** → 原图部分显示（叠加透明度）

这与图像处理中常见的 Alpha Mask 原理一致。

---

## 📐 原理图解

```
遮罩图： [透明 ←—— 渐变 ——→ 不透明]
               ↓ alpha 通道

应用图：  [被遮挡 ←—— 渐显 ——→ 显示完全]
```

`.mask` 不关心遮罩图的颜色（黑白红蓝都行），**只关心透明度**！

---

## 💡 常见误区

许多初学者会这样写：

```swift
.mask(
    LinearGradient(
        gradient: Gradient(colors: [.black, .white]),
        startPoint: .leading,
        endPoint: .trailing
    )
)
```

⚠️ **错！虽然是黑白渐变，但透明度都是 1（不透明），所以不会遮掉任何东西。**

正确写法应使用透明度变化：

```swift
Gradient(colors: [.clear, .black])
```

或者：

```swift
Color.black.opacity(0.0) → Color.black.opacity(1.0)
```

---

## 🛠 实战示例

### 1️⃣ 文字遮罩图像（图片填充文字）

```swift
Image("background")
    .resizable()
    .scaledToFill()
    .frame(width: 300, height: 100)
    .mask(
        Text("SwiftUI")
            .font(.system(size: 60, weight: .bold))
    )
```

📌 **效果**：文字轮廓内显示图片，类似 Photoshop 的“剪贴蒙版”。

---

### 2️⃣ 渐变遮罩图片（图像左边隐藏、右边显示）

```swift
Image("background")
    .resizable()
    .scaledToFill()
    .frame(width: 300, height: 200)
    .mask(
        LinearGradient(
            gradient: Gradient(colors: [.clear, .black]),
            startPoint: .leading,
            endPoint: .trailing
        )
        .frame(width: 300, height: 200)
    )
```

📌 **效果**：图像左侧逐渐隐去，右侧完全可见。

---

### 3️⃣ 动态遮罩动画（圆形扩展显示）

```swift
struct AnimatedMaskView: View {
    @State private var scale: CGFloat = 0.5

    var body: some View {
        Image("sky")
            .resizable()
            .scaledToFill()
            .frame(width: 300, height: 300)
            .mask(
                Circle()
                    .scale(scale)
                    .frame(width: 100, height: 100)
                    .animation(.easeInOut(duration: 2).repeatForever(), value: scale)
            )
            .onAppear { scale = 2.0 }
    }
}
```

📌 **效果**：圆形遮罩放大，图片逐渐显现，适合用于页面过渡动画。

---

## 🧱 高可复用封装：`.gradientMask(...)` 修饰符

为了方便开发，我们可以封装一个通用的渐变遮罩修饰符。

### 1️⃣ 定义方向枚举

```swift
enum MaskDirection {
    case leadingToTrailing, trailingToLeading, topToBottom, bottomToTop

    var gradientPoints: (start: UnitPoint, end: UnitPoint) {
        switch self {
        case .leadingToTrailing: return (.leading, .trailing)
        case .trailingToLeading: return (.trailing, .leading)
        case .topToBottom: return (.top, .bottom)
        case .bottomToTop: return (.bottom, .top)
        }
    }
}
```

---

### 2️⃣ 扩展 View 添加 `.gradientMask(...)`

```swift
extension View {
    func gradientMask(
        direction: MaskDirection = .leadingToTrailing,
        startOpacity: Double = 0.0,
        endOpacity: Double = 1.0
    ) -> some View {
        self.mask(
            LinearGradient(
                gradient: Gradient(colors: [
                    Color.black.opacity(startOpacity),
                    Color.black.opacity(endOpacity)
                ]),
                startPoint: direction.gradientPoints.start,
                endPoint: direction.gradientPoints.end
            )
        )
    }
}
```

---

### 3️⃣ 使用示例

```swift
Image("background")
    .resizable()
    .scaledToFill()
    .frame(width: 300, height: 200)
    .gradientMask(direction: .leadingToTrailing)
```

你还可以通过配置方向和透明度控制遮罩行为：

```swift
.view
    .gradientMask(direction: .topToBottom)

.view
    .gradientMask(direction: .trailingToLeading, startOpacity: 1.0, endOpacity: 0.0)
```

---

## ✨ 进阶玩法建议

- ✅ 支持多段 `Gradient(stops:)`
- ✅ 使用 `RadialGradient` / `AngularGradient`
- ✅ 添加 `.animatedGradientMask(...)` 动画修饰符
- ✅ 结合 `.canvas` 实现粒子遮罩
- ✅ 实现打字机文字遮罩、闪光遮罩、火焰动画等特效

---

## 📌 小结

| 功能             | 用法                           |
|------------------|--------------------------------|
| 剪裁文字形状     | `.mask(Text(...))`             |
| 渐变淡入淡出     | `.mask(LinearGradient(...))`   |
| 圆形遮罩裁剪     | `.mask(Circle())`              |
| 动画渐显         | 遮罩视图 + 动画绑定状态        |
| 通用渐变封装     | `.gradientMask(...)` 修饰符    |

---

## 📦 Demo 项目建议

建议将本文代码封装为一个 SwiftUI 模板或 Playground，加入 `TabView` 展示多种遮罩效果，便于教学与复用。

---

## 💬 结语

`.mask` 是 SwiftUI 中极具表现力的视觉工具，配合动画、渐变和形状能实现丰富的视觉交互。在实际项目中，无论是制作滚动图淡出效果、图文合成、动画过渡、组件裁剪，它都能大显身手。

学会 `.mask`，你就能解锁 SwiftUI 中更多 UI 魔法 ✨

---

## 🆚 SwiftUI 中的蒙版 vs Photoshop 蒙版

很多有设计经验的开发者会问：SwiftUI 的 `.mask` 和 Photoshop 的蒙版机制一样吗？它们在原理上是类似的，但也有一些关键区别：

| 比较项                 | SwiftUI `.mask` 修饰符                      | Photoshop 图层蒙版                     |
|------------------------|---------------------------------------------|----------------------------------------|
| 作用对象               | SwiftUI 中任意 View                        | Photoshop 中的图层                     |
| 基于的遮罩信息         | 遮罩视图的透明度（Alpha）通道             | 遮罩图层的灰度值（明度）               |
| 白色区域的表现         | 完全显示（alpha = 1）                      | 完全显示                               |
| 黑色区域的表现         | 完全隐藏（alpha = 0）                      | 完全隐藏                               |
| 灰色/半透明区域        | 部分显示（alpha = 0~1）                    | 部分透明，显示强度按灰度控制           |
| 遮罩图是否可见         | 不可见，只影响被遮罩视图的显示            | 遮罩图本身不可见，仅影响图层显示       |
| 动画支持               | 可组合 SwiftUI 动画与遮罩                 | 无动画机制，仅静态遮罩                 |

📌 总结：

- **相同点**：两者都使用遮罩图的「透明程度」决定目标显示区域。
- **不同点**：SwiftUI 用 Alpha 通道 + 动画能力，更具交互性；Photoshop 用灰度图静态处理，适合图像编辑。

这意味着你可以将设计师习惯的“黑白遮罩”逻辑迁移到 SwiftUI，但还可以进一步发挥 SwiftUI 的动画和响应式能力，做出动态 UI 效果。
