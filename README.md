

# SBSB — 烧饼论坛 Android App

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Min SDK](https://img.shields.io/badge/min_sdk-Android_8.0_(API_26)-green.svg)](https://developer.android.com/studio/releases/platforms)
[![Kotlin](https://img.shields.io/badge/language-Kotlin_1.9-orange.svg)](https://kotlinlang.org/)
[![GeckoView](https://img.shields.io/badge/engine-Mozilla_GeckoView_158-red.svg)](https://firefox-source-docs.mozilla.org/mobile/android/geckoview/index.html)

**烧饼论坛 (sb.sb)** 的 Android 原生客户端，采用 **Mozilla GeckoView** 内置独立内核渲染网页，不依赖系统 WebView，彻底解决国产 ROM 锁死旧版 WebView 导致的人机验证组件无法显示问题。

---

### [Releases - SB.SB APP下载](https://github.com/EvanTop/sbsb/releases/tag/SBSB_v2.0.0)

## 项目简介

SBSB 是一个轻量级 Android WebView 壳应用，将 [烧饼论坛](https://sb.sb) 封装为原生 Android 应用。与传统的基于系统 `WebView` 的方案不同，本应用内嵌 **Mozilla GeckoView** 引擎，完全脱离操作系统提供的 Chromium WebView 实现，从而在任何 Android 设备上都能获得一致的现代 Web 标准支持。

### 为什么不用系统 WebView？

| 方案 | 优点 | 缺点 |
|------|------|------|
| **系统 WebView** | APK 体积小（~5 MB），无需额外依赖 | 依赖厂商定制，部分 ROM（如锤子坚果）锁定旧版内核（Chrome 75），不支持现代 JS 语法，Cap 人机验证组件直接 SyntaxError 崩溃 |
| **GeckoView 内置内核** | 独立于系统，始终提供最新 Web 标准支持；绕过所有 ROM 限制；验证组件 100% 可用 | APK 体积较大（~190 MB），需联网拉取 GeckoView AAR |

本项目的核心设计哲学是：**内核一定是网站，APP 只做容器**。不做任何网页内容注入、CSS 覆盖或 JS 篡改，完全依托网站自身手机端样式与功能。

---

## 技术栈

```
┌─────────────────────────────────────────────────────┐
│                   SBSB v2.0.0                       │
├──────────┬──────────┬──────────┬────────────────────┤
│ 语言     │ 框架     │ 内核     │ 构建工具           │
├──────────┼──────────┼──────────┼────────────────────┤
│ Kotlin   │ AndroidX │ GeckoView│ AGP 9.4.1          │
│          │ Core KTX │ 158      │ Gradle 9.6.0       │
│          │ Material │ Mozilla  │ JDK 21             │
└──────────┴──────────┴──────────┴────────────────────┘
```

### 详细依赖清单

| 类别 | 库 / 版本 | 用途 |
|------|-----------|------|
| **UI 框架** | `androidx.appcompat:appcompat:1.7.0` | 兼容层、主题、ActionBar |
| **Material Design** | `com.google.android.material:material:1.12.0` | Material 组件库 |
| **Core Utilities** | `androidx.core:core-ktx:1.15.0` | Kotlin 扩展工具集 |
| **Web 引擎** | `org.mozilla.geckoview:geckoview:158.0.20261021001945` | Mozilla 开源浏览器引擎，替代系统 WebView |
| **构建工具链** | AGP 9.4.1 + Gradle 9.6.0 + Kotlin 1.9.x | Android 应用打包流水线 |

### GeckoView 配置要点

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `javaScriptEnabled` | `true` | 启用 JavaScript 执行 |
| `cookieBehavior` | `ACCEPT_ALL` | 放行第三方 Cookie（Cap 人机验证跨域所需） |
| `userAgentOverride` | 标准 Chrome Android UA | 伪装为标准浏览器指纹，规避 Capjs 对非标准 UA 的拒绝策略 |
| `consoleOutput` | `true` | Console 消息输出到 logcat，用于兜底诊断 |
| `contentBlocking` | ACCEPT_ALL | 不拦截任何网络请求 |

---

## 架构设计

```
┌─────────────────────────────────────────┐
│              MainActivity               │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │         GeckoView                 │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │   sb.sb/login/              │  │  │
│  │  │   ┌───────────────────────┐ │  │  │
│  │  │   │ <cap-widget>          │ │  │  │
│  │  │   │ (Shadow DOM, WASM)    │ │  │  │
│  │  │   └───────────────────────┘ │  │  │
│  │  │                             │  │  │
│  │  │   Username / Password       │  │  │
│  │  │   Login Button              │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### 核心流程

1. **应用启动** → `MainActivity.onCreate()` 初始化 `GeckoRuntime`
2. **创建 Session** → `GeckoSession` 绑定到 `GeckoView` 视图
3. **加载页面** → `geckoSession.loadUri("https://sb.sb/login/")`
4. **UA 伪装** → 覆盖 User-Agent 为标准 Chrome Android 串
5. **Cookie 放行** → ContentBlocking 接受所有 Cookie（含第三方）
6. **渲染完成** → `<cap-widget>` 自定义元素正常注册并显示

### 安全设计

- **无网页注入**：不对 DOM 进行任何修改，不注入 CSS/JS
- **仅日志兜底**：Console 错误输出到 logcat，不弹窗干扰用户体验
- **纯容器定位**：APP 只做加载和展示，功能全部由网站自身提供

---

## 构建说明

### 环境要求

| 组件 | 最低版本 |
|------|----------|
| JDK | 21+ |
| Android Studio | Hedgehog 或更新 |
| SDK Platform | API 37.2 |
| NDK | r27+ |

### 构建步骤

```bash
# 1. 克隆仓库
git clone <your-repo-url>
cd sbsb-app

# 2. 生成签名密钥（首次使用）
keytool -genkey -v -keystore release.keystore \
  -alias sbsb -keyalg RSA -keysize 2048 -validity 10000 \
  -storepass sbsb2026 -keypass sbsb2026 \
  -dname "CN=SB.SB Forum App"

# 3. 构建 Release APK
./gradlew assembleRelease

# 4. 签名 APK
apksigner sign --ks release.keystore --ks-key-alias sbsb \
  app/build/outputs/apk/release/app-release.apk
```

### 产物信息

| 属性 | 值 |
|------|-----|
| 包名 | `com.sbsb.app` |
| 显示名称 | 烧饼论坛 |
| 版本号 | 2.0.0 (versionCode 4) |
| 最小 SDK | 26 (Android 8.0 Oreo) |
| 目标 SDK | 35 (Android 15) |
| ABI 过滤 | arm64-v8a |
| APK 体积 | ~190 MB |
| SHA-256 | `8659aac8a343e4810d6012d97c1d67473a5147591ed3a3c12bafed7208229a93` |

---

## 已知限制

| 限制项 | 原因 | 影响范围 |
|--------|------|----------|
| APK 体积较大 (~190MB) | GeckoView 内置完整浏览器引擎 | 下载时间较长，但功能完整性优先 |
| 仅 arm64-v8a | 为控制体积裁剪了其他 ABI | 32 位设备无法安装 |
| minSdk 26 | GeckoView 158 要求 | Android 8.0 以下设备不可用 |
| 需联网拉取 GeckoView | Maven 远程依赖 | 离线构建需提前缓存 AAR |

---

## 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

---

## 联系方式

- 网站：[sb.sb](https://sb.sb)
- 作者：nsz (Timers)

---

*Built with ❤️ using Mozilla GeckoView & Kotlin on macOS.*
*（内容由AI生成，仅供参考）*
