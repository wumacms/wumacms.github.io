# 📱 优雅构建 SwiftUI 数字键盘：从原始实现到组件化优化的思考

## ✨ 引言

在 SwiftUI 项目中，代码的第一版本往往为了「先跑通」，不可避免地牺牲了部分结构清晰性与复用性。本文以一个数字键盘（`KeyPadView`）为例，展示如何从一个功能完备但结构繁杂的实现，逐步演进到一个**高内聚、低耦合、可维护性强**的组件化结构，并剖析每一步背后的设计思路与工程实践价值。

## 🧱 原始实现分析：功能完整但结构臃肿

原始版本的 `KeyPadView` 完成了数字与操作符按钮的展示及点击逻辑，但存在以下问题：

```swift
import SwiftUI

struct KeyPadView: View {
    @EnvironmentObject var calcVm: CalcViewModel
    
    // 按钮二维数组
    let buttons: [[CalcButton]] = [
        [.clear, .left, .right, .delete],
        [.one, .two, .three, .add],
        [.four, .five, .six, .subtract],
        [.seven, .eight, .nine, .multiply],
        [.clearScreen, .zero, .decimal, .divide],
    ]
    
    var body: some View {
        VStack(spacing: 3) {
            ForEach(buttons, id: \.self) { row in
                HStack(spacing: 3) {
                    // MARK: 键盘按钮
                    ForEach(row, id: \.self) { item in
                        Button {
                            calcVm.input(button: item)
                        } label: {
                            Group {
                                if item == .delete {
                                    Image(systemName: "delete.left")
                                } else if item == .clearScreen {
                                    //Image(systemName: "gearshape")
                                    //Image(systemName: "paintpalette")
                                    Text(LocalizedStringKey(item.rawValue))
                                } else if item == .add {
                                    // +号
                                    Image(systemName: "plus")
                                } else if item == .subtract {
                                    // -号
                                    Image(systemName: "minus")
                                } else if item == .multiply {
                                    // ×号
                                    Image(systemName: "multiply")
                                } else if item == .divide {
                                    // ÷号
                                    Image(systemName: "divide")
                                } else if item == .decimal {
                                    // 小数点加粗
                                    Text(item.rawValue).bold()
                                } else {
                                    Text(item.rawValue)
                                }
                            }
                            .frame(maxWidth: .infinity, maxHeight: .infinity)
                            .frame(minWidth: 35, minHeight: 35)
                            .foregroundColor(calcVm.theme.text)
                            .font(.system(size: Font.font.getFontSize()))
                            .background(calcVm.theme.keyPadBgLight)
                            .cornerRadius(10)
                            .overlay {
                                RoundedRectangle(cornerRadius: 10)
                                    .stroke(calcVm.theme.inputBorder, lineWidth: 1)
                            }
                        }
                    }
                }
            }
        }
    }
}

struct KeyPadView_Previews: PreviewProvider {
    static var previews: some View {
        KeyPadView()
            .environmentObject(CalcViewModel())
    }
}
```

### 🔍 问题一：UI 与业务逻辑交叉在主视图中

视图本应专注于「展示」，但这里却承载了复杂的判断逻辑，如按钮样式差异、符号选择、文字粗细等，使 `body` 结构难以阅读和扩展。

### 🔍 问题二：可复用性差

UI 细节（如 `.frame`, `.foregroundColor`, `.overlay`）在按钮循环中重复出现，**一旦设计变更需要逐一修改**。

### 🔍 问题三：语义表达弱

比如 `item.rawValue` 既用于展示，又被本地化处理，但没有明确告诉我们什么时候是符号、什么时候是文本。这种写法不利于新成员阅读与维护。

## ✨ 目标：组件化与可维护性并存

优化目标不仅是「重构」，而是**以更清晰的表达、更强的结构和更易拓展的方式组织代码**。优化方向如下：

| 维度 | 优化目标 |
|------|----------|
| 结构组织 | 分离主视图与按钮渲染逻辑 |
| 可读性 | 使用专属扩展封装判断逻辑 |
| 复用性 | 抽出可复用按钮组件 |
| 本地化支持 | 明确哪些文字需本地化处理 |
| 风格一致性 | 按钮样式集中定义，便于全局修改 |

## 🔧 优化实现

### ✅ 优化后的 `KeyPadView`

```swift
struct KeyPadView: View {
    @EnvironmentObject var calcVm: CalcViewModel

    private let buttons: [[CalcButton]] = [
        [.clear, .left, .right, .delete],
        [.one, .two, .three, .add],
        [.four, .five, .six, .subtract],
        [.seven, .eight, .nine, .multiply],
        [.clearScreen, .zero, .decimal, .divide],
    ]

    var body: some View {
        VStack(spacing: 3) {
            ForEach(buttons, id: \.self) { row in
                HStack(spacing: 3) {
                    ForEach(row, id: \.self) { item in
                        KeyPadButtonView(button: item)
                            .environmentObject(calcVm)
                    }
                }
            }
        }
    }
}
```


### ✅ 1. 抽离子视图 `KeyPadButtonView`

将每个按钮封装为独立组件，职责单一，专注于展示和交互：

```swift
struct KeyPadButtonView: View {
    let button: CalcButton
    @EnvironmentObject var calcVm: CalcViewModel

    var body: some View {
        Button {
            calcVm.input(button: button)
        } label: {
            Group {
                if let symbol = button.symbolName {
                    Image(systemName: symbol)
                } else {
                    Text(button.textLabel)
                        .fontWeight(button == .decimal ? .bold : .regular)
                }
            }
            .modifier(KeyPadButtonStyle(theme: calcVm.theme))
        }
    }
}
```

#### 🎯 好处：
- **关注点分离**：主视图只关注布局，子组件负责展示。
- **易于测试与修改**：单独修改按钮样式不影响整体结构。

### ✅ 2. 使用扩展封装 `CalcButton` 判断逻辑

通过枚举扩展将 `symbolName` 与 `textLabel` 的逻辑封装：

```swift
extension CalcButton {
    var symbolName: String? {
        switch self {
        case .delete: return "delete.left"
        case .add: return "plus"
        case .subtract: return "minus"
        case .multiply: return "multiply"
        case .divide: return "divide"
        default: return nil
        }
    }

    var textLabel: String {
        switch self {
        case .clearScreen, .clear:
            return NSLocalizedString(rawValue, comment: "")
        default:
            return rawValue
        }
    }
}
```

#### 🎯 好处：
- **消除冗长的 if/else 链**；
- **行为语义化清晰**（调用方只需关心 `.symbolName` 和 `.textLabel`）；
- **提升可测试性**（可以独立对其进行单元测试）；

### ✅ 3. 封装样式逻辑为 `ViewModifier`（可选）

也可以进一步将按钮样式封装为自定义 Modifier 或统一样式结构：

```swift
struct KeyPadButtonStyle: ViewModifier {
    let theme: Theme

    func body(content: Content) -> some View {
        content
            .frame(maxWidth: .infinity, maxHeight: .infinity)
            .frame(minWidth: 35, minHeight: 35)
            .font(.system(size: Font.font.getFontSize()))
            .foregroundColor(theme.text)
            .background(theme.keyPadBgLight)
            .cornerRadius(10)
            .overlay {
                RoundedRectangle(cornerRadius: 10)
                    .stroke(theme.inputBorder, lineWidth: 1)
            }
    }
}
```

## 📈 成本与收益权衡分析

| 方面 | 原始实现 | 优化实现 |
|------|----------|-----------|
| 代码长度 | 相对短 | 稍长（结构化） |
| 可维护性 | 差，易错难查 | 高，结构清晰 |
| 复用性 | 无法复用 | 支持多场景复用 |
| 阅读成本 | 高 | 低（模块分明） |
| 扩展性 | 差（耦合高） | 强（组件可替换） |
| 风格一致性 | 维护困难 | 集中控制 |

## 🧠 设计哲学反思

SwiftUI 应鼓励开发者以模块化视角构建界面。通过本文的重构实践，我们不仅获得了更好的代码质量，也锻炼了设计思维。

## ✅ 结语

优雅的 UI 实现，从不只是视觉，更是结构、行为与可维护性的平衡艺术。如果你正在构建一套 SwiftUI UI 系统，请思考：**它是否具备清晰的责任边界与良好的扩展能力？**

这将是构建可持续代码库的重要一步。