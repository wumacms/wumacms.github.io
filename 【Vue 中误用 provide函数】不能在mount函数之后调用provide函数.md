
# ❌ Vue 中误用 provide：不能在 `.mount()` 之后调用 `.provide()`

在 Vue 3 中，**`provide` 函数不能在挂载后的组件实例上调用**。下面这段代码是错误的：

```js
const app = createApp(App).mount('#app')

app.provide('message', 'Hello, World!')
```

这是 **不正确的写法**，因为：

- `mount()` 返回的是 **组件实例**（`ComponentPublicInstance`），
- 而不是 **应用实例**（`App` 类型），
- 因此不能在其上调用 `.provide()` 方法。

---

## ✅ 正确写法

你应该在 `.mount()` **之前**调用 `.provide()`：

```js
const app = createApp(App)
app.provide('message', 'Hello, World!')
app.mount('#app')
```

---

## 📌 总结对比

### 错误写法与原因

| 错误写法                      | 原因                                                         |
|------------------------------|--------------------------------------------------------------|
| `createApp().mount().provide()` | `.mount()` 返回的是组件实例，不能调用 `.provide()` 方法         |

### 正确写法与描述

| 正确写法                      | 描述                                           |
|------------------------------|------------------------------------------------|
| `createApp().provide().mount()` | `.provide()` 应该在 `.mount()` 前调用，作用于整个应用实例 |

---

## 💡 组件内部使用 `provide` 的方法

如果你是在 **组件内部** 使用 `provide`，可以采用组合式 API 的写法：

```html
<script setup>
import { provide } from 'vue';
provide('message', 'Hello, World!');
</script>
```
