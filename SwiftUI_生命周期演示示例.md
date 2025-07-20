
# SwiftUI 视图生命周期演示示例

本文提供了一个完整的 SwiftUI 示例代码，用于观察 SwiftUI 视图的 `init()`、`body`、`onAppear()` 和 `onDisappear()` 等生命周期事件。

---

## ✅ 示例代码

```swift
import SwiftUI

struct LifecycleRootView: View {
    var body: some View {
        NavigationView {
            VStack(spacing: 20) {
                NavigationLink("跳转到子视图", destination: LifecycleChildView())

                TabViewExample()

                Spacer()
            }
            .padding()
            .navigationTitle("生命周期演示")
        }
    }
}

struct LifecycleChildView: View {
    @State private var counter = 0

    init() {
        print("🟢 [init] LifecycleChildView")
    }

    var body: some View {
        print("🌀 [body] LifecycleChildView")

        return VStack(spacing: 20) {
            Text("计数器：\(counter)")

            Button("增加计数") {
                counter += 1
            }

            NavigationLink("跳转到第三层", destination: Text("第三层视图"))

        }
        .padding()
        .onAppear {
            print("✅ [onAppear] LifecycleChildView")
        }
        .onDisappear {
            print("❌ [onDisappear] LifecycleChildView")
        }
        .navigationTitle("子视图")
    }
}

struct TabViewExample: View {
    var body: some View {
        TabView {
            TabPageView(index: 1)
                .tabItem {
                    Label("页面一", systemImage: "1.circle")
                }

            TabPageView(index: 2)
                .tabItem {
                    Label("页面二", systemImage: "2.circle")
                }
        }
        .frame(height: 200)
    }
}

struct TabPageView: View {
    let index: Int

    init(index: Int) {
        print("🟢 [init] TabPageView \(index)")
    }

    var body: some View {
        print("🌀 [body] TabPageView \(index)")

        return Text("这是 Tab 页：\(index)")
            .onAppear {
                print("✅ [onAppear] TabPageView \(index)")
            }
            .onDisappear {
                print("❌ [onDisappear] TabPageView \(index)")
            }
    }
}
```

---

## 🧪 使用方式

将上述代码复制到 SwiftUI 项目中，将入口设置为：

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            LifecycleRootView()
        }
    }
}
```

---

## 🔍 生命周期观察点

| 生命周期事件 | 说明 |
|--------------|------|
| `init()` | 每次视图结构创建时调用 |
| `body` | 状态变更时触发，视图会被重新计算 |
| `onAppear()` | 视图进入屏幕时调用 |
| `onDisappear()` | 视图离开屏幕时调用 |

---

## 📌 行为与生命周期影响

| 用户行为 | 生命周期影响 |
|----------|----------------|
| 修改 `@State` | 触发 `body`，不触发 `init/onAppear` |
| 页面导航跳转 | 新视图走完 `init → body → onAppear` 流程 |
| Tab 切换 | 当前页 `onAppear`，上一页 `onDisappear` |
| Navigation 返回 | 离开视图 `onDisappear`，返回页不触发 `onAppear` |

---

通过运行此示例，你可以清晰观察 SwiftUI 中视图的生命周期流转过程。
