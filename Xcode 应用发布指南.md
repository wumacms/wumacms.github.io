
# 📦 Xcode 应用发布指南

本文档将指导你如何在 Xcode 中完成 iOS 应用的发布流程，涵盖从版本配置到 App Store 上传的完整步骤。

---

## ✅ 一、准备工作

### 1. 配置版本号和构建号（Version & Build）
- 打开 Xcode 项目，选中 `.xcodeproj` 或 `.xcworkspace` 文件。
- 在 **"General"** 页签：
  - **Version（版本号）**：用户可见的版本号，如 `1.0.0`。
  - **Build（构建号）**：每次构建需递增，如 `1`、`2`、`3` 等。

### 2. 设置发布配置（Release Build）
- 菜单栏：`Product` → `Scheme` → `Edit Scheme` → 使用 `Release` 配置。

### 3. 证书 & 签名配置
- 登录 Apple Developer 账号（Xcode → Settings → Accounts）。
- 自动签名启用方式：Target → Signing & Capabilities → 勾选 "Automatically manage signing"。

---

## 🚀 二、归档（Archive）

1. 将运行目标设置为 **Any iOS Device (arm64)**。
2. 菜单栏 → `Product` → `Archive`。
3. 构建成功后自动打开 `Organizer` 窗口。

---

## 📦 三、上传 App 到 App Store Connect

### 使用 Xcode 上传

1. 在 `Organizer` 中点击 **Distribute App**。
2. 选择：
   - `App Store Connect`
   - `Upload`
3. 完成上传，等待审核。

---

## 📡 四、App Store Connect 配置

1. 登录 [App Store Connect](https://appstoreconnect.apple.com/)
2. 进入“我的 App” → 选择 App → 点击“新建版本”。
3. 填写信息、添加截图、关键词、版本说明等。
4. 提交审核。

---

## 🧪 命令行辅助（可选）

```bash
# 归档项目
xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -archivePath build/MyApp.xcarchive archive

# 导出 .ipa 包
xcodebuild -exportArchive -archivePath build/MyApp.xcarchive -exportPath build/export -exportOptionsPlist exportOptions.plist
```

---

## 🧰 建议工具

- TestFlight 内测
- Fastlane 自动打包上传
- Git Tag 管理更新日志

---

如需配置企业分发、自定义脚本、CI/CD 自动化发布流程，欢迎继续提问。
