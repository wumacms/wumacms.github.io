# SwiftUI中`onDisappear()`触发时机总结



在 SwiftUI 中，`onDisappear()` 是一个视图生命周期修饰符，会在视图从视图层次结构中移除时触发。以下是常见的触发场景以及完整代码示例，展示 `onDisappear` 的常见使用场景，涵盖导航、模态视图、条件渲染和列表滚动等典型情况。每个示例都附带详细注释和实际应用逻辑。

---

### 1. **导航离开当前视图**

   - **`NavigationStack` / `NavigationView` 的弹出**：
     - 当用户点击返回按钮或编程调用 `navigationStack.pop()` 时，当前视图会消失，触发 `onDisappear`。
   - **跳转到新视图**：
     - 通过 `navigationLink` 跳转到新页面时，原视图的 `onDisappear` 也会触发（因为原视图可能被销毁或暂时移出屏幕）。

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            VStack {
                NavigationLink("Go to Detail") {
                    DetailView()
                }
            }
            .padding()
        }
    }
}

struct DetailView: View {
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        VStack {
            Text("Detail View")
            Button("Dismiss") { dismiss() }
        }
        .onAppear { print("DetailView appeared") }
        .onDisappear { print("DetailView disappeared") } // 点击返回或编程 dismiss 时触发
    }
}
```
**触发场景**：  
- 用户点击系统返回按钮或代码调用 `dismiss()`。

---

### 2. **模态视图的关闭**

   - 通过 `sheet`、`fullScreenCover` 或 `popover` 展示的模态视图，在关闭时会触发 `onDisappear`。

```swift
struct ContentView: View {
    @State private var showSheet = false
    
    var body: some View {
        Button("Show Sheet") {
            showSheet.toggle()
        }
        .sheet(isPresented: $showSheet) {
            SheetView()
                .onDisappear { 
                    print("Sheet disappeared") // 下滑关闭或代码置空 showSheet 时触发
                }
        }
    }
}

struct SheetView: View {
    var body: some View {
        Text("This is a Sheet")
    }
}
```

---

### 3. **条件渲染导致的移除**

   - 当视图因条件判断（如 `if`、`switch`）不再渲染时，会触发 `onDisappear`。

```swift
struct ContentView: View {
    @State private var isShowingView = true
    
    var body: some View {
        VStack {
            if isShowingView {
                Text("Conditional View")
                    .onDisappear { 
                        print("Conditional View disappeared") // isShowingView 变 false 时触发
                    }
            }
            Button("Toggle View") {
                isShowingView.toggle()
            }
        }
    }
}
```

---

### 4. **`TabView` 切换标签页**

   - 在 `TabView` 中切换标签页时，非当前页的视图会触发 `onDisappear`（具体行为取决于 SwiftUI 的视图生命周期管理）。

```swift
struct ContentView: View {
    @State private var selectedTab = 0
    
    var body: some View {
        TabView(selection: $selectedTab) {
            Text("Tab 1")
                .tabItem { Label("Tab 1", systemImage: "1.circle") }
                .tag(0)
                .onDisappear { print("Tab 1 disappeared") } // 切换到 Tab 2 时触发
            
            Text("Tab 2")
                .tabItem { Label("Tab 2", systemImage: "2.circle") }
                .tag(1)
        }
    }
}
```

---

### 5. **`List` 或 `ScrollView` 中的视图滚动离开屏幕**

   - 在可滚动容器（如 `List`、`ScrollView`）中，如果 SwiftUI 复用或回收视图（例如 `LazyVStack`），离开屏幕的视图可能触发 `onDisappear`。

```swift
struct ContentView: View {
    let items = (1...100).map { "Item \($0)" }
    
    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
                .onAppear { print("\(item) appeared") }
                .onDisappear { print("\(item) disappeared") } // 滚动出屏幕时触发
        }
    }
}
```

---

### 6. **父视图的销毁**

   - 如果父视图被移除（如被 `if` 条件移除或导航栈重置），其所有子视图会触发 `onDisappear`。

---

#### 注意事项

1. **异步触发**：`onDisappear` 的调用可能会有轻微延迟（例如导航动画完成后）。
2. **与 `onAppear` 的对称性**：通常需要配对使用，管理资源（如订阅、计时器）。
3. **`Lazy` 容器的特殊性**：在 `LazyVStack` 等惰性容器中，视图可能因滚动复用多次触发 `onAppear`/`onDisappear`。

#### **完整示例：结合资源管理（如计时器）**

```swift
struct TimerExampleView: View {
    @State private var isTimerActive = false
    @State private var timer = Timer.publish(every: 1, on: .main, in: .common).autoconnect()
    @State private var counter = 0
    
    var body: some View {
        VStack {
            Text("Counter: \(counter)")
                .onReceive(timer) { _ in
                    if isTimerActive { counter += 1 }
                }
            
            NavigationLink("Go to Next") {
                Text("Next View")
                    .onDisappear { 
                        isTimerActive = false // 导航离开时暂停计时器
                        print("Timer paused")
                    }
                    .onAppear { 
                        isTimerActive = true // 返回时恢复计时器
                        print("Timer resumed")
                    }
            }
        }
        .onDisappear { 
            timer.upstream.connect().cancel() // 视图完全销毁时终止计时器
            print("Timer terminated")
        }
    }
}
```

当 `isActive` 变为 `false` 时，`onDisappear` 会执行并停止计时器。

---

### 关键总结
1. **导航相关**：`NavigationStack`、`NavigationLink`、`dismiss`。  
2. **模态视图**：`sheet`、`fullScreenCover`。  
3. **动态 UI**：`if` 条件、`ForEach` 数据源变化。  
4. **性能优化**：在 `List`/`LazyVStack` 中用 `onDisappear` 释放资源。  
5. **资源管理**：配对使用 `onAppear` 和 `onDisappear` 管理订阅、网络请求或计时器。

通过以上代码，可以清晰理解 `onDisappear` 的实际应用场景和最佳实践。