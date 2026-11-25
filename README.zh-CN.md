<p align="center">
  <img src="https://img.shields.io/badge/License-MIT-d79921.svg?style=flat-square" alt="License: MIT" />
  <img src="https://img.shields.io/badge/测试用例-144%20条%20(90%20基线%20%2B%2054%20公共)-98971a.svg?style=flat-square" alt="测试用例: 144 条" />
  <img src="https://img.shields.io/badge/适配目标-iPhone%20Duo%20%2F%20iOS%2027.2-458588.svg?style=flat-square" alt="适配目标: iPhone Duo / iOS 27.2" />
  <img src="https://img.shields.io/badge/工具链-Xcode%2027.2%20%2B%20serve--sim-b16286.svg?style=flat-square" alt="工具链: Xcode 27.2 + serve-sim" />
</p>

<h1 align="center">iPhone Duo Test</h1>

<p align="center">
  面向 Apple iPhone Duo 折叠屏适配的公共兼容性测试用例库、姿态测试矩阵与 Xcode 27.2 + Codex 模拟器自动化测试指南。
</p>

<p align="center">
  <a href="./README.md">English</a> ·
  <a href="https://xsoway.github.io/iphone-duo-test/">项目主页</a> ·
  <a href="https://github.com/xsoway/iphone-duo-test/discussions">Discussions 讨论区</a>
</p>

<p align="center">
  144 条结构化用例 · 合盖 / 展开 / 半开 / 分屏全覆盖 · 模拟器镜像串流与取证闭环
</p>

<hr>

## 目录

- [是什么](#是什么)
- [为什么需要](#为什么需要)
- [核心概念](#核心概念)
- [仓库结构](#仓库结构)
- [快速开始](#快速开始)
- [测试矩阵与用例覆盖](#测试矩阵与用例覆盖)
- [取证与排障](#取证与排障)
- [设计原则](#设计原则)
- [常见问题](#常见问题)
- [路线图](#路线图)
- [贡献指南](#贡献指南)
- [License](#license)
- [维护者](#维护者)

---

## 是什么

**iPhone Duo Test** 是一个面向 iOS 原生及混合容器（Safari / `WKWebView`）应用的 **iPhone Duo 折叠屏适配测试知识库与公共用例集**。

仓库包含两类核心可落地资产：

1. **144 条公共兼容性测试用例**（Markdown + 5 工作表 Excel 工作簿）：包含 90 条通用化基线用例（`M1`–`M7`）与 54 条 Duo 专项新增公共用例（`PUB-001`–`PUB-054`）。
2. **模拟器自动化与取证实践指南**：完整记录如何在 **Xcode 27.2 / iOS 27.2 Runtime** 下校验 `.app` 构建产物、通过 `serve-sim@0.1.49` 将模拟器画面串流至本机浏览器、执行合盖/半开/展开（`0°` / `130°` / `180°`）与旋转指令，并按显示面截图及抓取 `SpringBoard` 日志。

---

## 为什么需要

普通直板 iPhone 上，列表进入详情再返回大多发生在同一块不变的物理屏幕上；而在 iPhone Duo 上，用户可能在**键盘未收起、弹窗打开、甚至文件上传或异步请求尚未结束时**折叠、半开或切入系统分屏。页面拿到的可用区域变了，但手上的任务和对象状态必须继续保持正确。

| 设备差异 | 普通 iPhone 常见基线 | iPhone Duo 多出来的变量 | 测试时重点观察 |
| --- | --- | --- | --- |
| 物理显示面 | 同一块主屏内变化 | 合盖外屏（`A`）与展开内屏（`B`）切换 | 切换前后对象 ID、路由栈、未提交输入、滚动锚点与主动作 |
| 屏幕比例与安全区 | 刘海、相机、底部手势区 | 外屏短而宽，系统控件可位于侧边，左右安全区不对称 | 分别测试左右方向，确认按钮可见且点击命中目标 |
| 内屏布局 | 随当前窗口宽度适配 | 同一页面可能从单栈变为列表与详情双栏，或保持单列 | 默认选中项、单栏返回关系、左右栏数据同步 |
| 半开折叠区 | 无横向/纵向铰链保留区 | 半开（`C`）出现折叠保留区与分段可用区域 | 文字、必填字段、关闭动作与触摸热区是否避开保留区 |
| 系统多任务 | 单 App 手机窗口 | 内屏可与另一 App 并排进入系统 Split View（`D`） | 左右换边、当前焦点、键盘归属与退出分屏后恢复 |
| 相机与输入 | 同一显示环境内弹出选择器 | 切屏涉及不同前置相机位置 | 取景、回调、权限弹窗及遮罩在切屏后的表现 |

---

## 核心概念

测试记录中必须严格区分以下四个概念，才能准确定位究竟是哪一层布局规则出了偏差：

- **外屏与内屏（`A` / `B`）**：两块独立的物理显示面。
- **应用内双栏**：同一个 App 在较宽窗口下同时展示列表与详情。
- **系统 Split View（`D`）**：两个不同 App 在内屏并排运行，各自拿到缩窄后的窗口。
- **半开与折叠保留区（`C`）**：设备处于半开角度（如 `130°`）时产生的窗口分段与不可交互铰链保留区。

---

## 仓库结构

```text
iphone-duo-test/
├── README.md                                  # 英文版说明文档
├── README.zh-CN.md                            # 中文版说明文档（本文件）
├── index.html                                 # 黑金风格交互式项目主页（支持中英切换）
├── LICENSE                                    # MIT 开源协议（Copyright 2026 Alan Hsu）
├── RELEASE_CHECKLIST.md                       # 开源发布逐项检查记录与敏感信息扫描证据
├── .gitignore                                 # 忽略系统文件、日志、取证产物与凭据
├── .gitattributes                             # 统一 LF 行尾并标记 .xlsx 为二进制文件
├── docs/
│   ├── iphone-duo-public-test-cases.md        # 144 条公共测试用例（90 基线 + 54 PUB-001..054）
│   └── xcodex-simulator-automation-guide.md   # Xcode 27.2 + Codex + serve-sim 自动化测试指南
└── data/
    └── iphone-duo-public-test-cases.xlsx      # 含 5 个工作表的 Excel 用例执行与追踪工作簿
```

| 文件 | 职责说明 |
| --- | --- |
| [docs/iphone-duo-public-test-cases.md](./docs/iphone-duo-public-test-cases.md) | 完整定义执行矩阵（`A`、`B`、`C`、`D`、`A↔B`）、判定标准、54 条 `PUB-*` 新增用例及 90 条通用基线用例。 |
| [data/iphone-duo-public-test-cases.xlsx](./data/iphone-duo-public-test-cases.xlsx) | 结构化 Excel 工作簿，含 `用例总览`、`姿态矩阵`、`通用基线`、`参考资料`、`新增公共用例` 共 5 个工作表。 |
| [docs/xcodex-simulator-automation-guide.md](./docs/xcodex-simulator-automation-guide.md) | 从 `.app` 平台校验、Device Hub 启动、Codex `ios-simulator-browser` 接入到 `serve-sim` 姿态切换与系统取证的全流程说明。 |
| [index.html](./index.html) | 单文件静态项目主页，内置实时时钟、核心测试流水线动画、可展开目录树与交互式命令试用终端。 |

---

## 快速开始

### 环境要求

- **macOS** + **Xcode 27.2**（或 27.1+），并在 **Xcode → Settings → Components** 中安装 **iOS 27.2 Simulator Runtime**。
- **Node.js / npx**（用于运行 `serve-sim@0.1.49`）。
- 可选：**Codex** 及插件市场的 `build-ios-apps@openai-curated` 插件（`ios-simulator-browser` 技能）。

### 1. 核对工具链与模拟器

```bash
xcodebuild -version
xcrun simctl list runtimes
xcrun simctl list devices available
```

### 2. 校验模拟器 `.app` 构建并安装启动

先确认可执行文件目标平台为 `IOSSIMULATOR`（而非 macOS `arm64`），再使用对应 iPhone Duo 模拟器 UDID 启动、安装并拉起应用：

```bash
APP="/absolute/path/to/TestApp.app"
BUNDLE=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' "$APP/Info.plist")
EXE=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleExecutable' "$APP/Info.plist")
xcrun vtool -show-build "$APP/$EXE"

SIM="<iPhone Duo UDID>"
xcrun simctl bootstatus "$SIM" -b
xcrun simctl install "$SIM" "$APP"
xcrun simctl launch "$SIM" "$BUNDLE"
```

### 3. 启动浏览器镜像并驱动折叠 / 旋转切换

通过 `serve-sim@0.1.49` 在本机端口建立镜像，并在业务交互过程中注入姿态角度与方向变化：

```bash
# 启动浏览器镜像（保持终端前台运行）
npx --yes serve-sim@0.1.49 --host 127.0.0.1 --panes tools --fit "$SIM"

# 驱动合盖、半开、展开与旋转
npx --yes serve-sim@0.1.49 fold 0 -d "$SIM"
npx --yes serve-sim@0.1.49 fold 130 -d "$SIM"
npx --yes serve-sim@0.1.49 fold 180 -d "$SIM"
npx --yes serve-sim@0.1.49 rotate landscape_left -d "$SIM"
npx --yes serve-sim@0.1.49 rotate portrait -d "$SIM"

# 测试结束后清理当前 UDID 的镜像进程
npx --yes serve-sim@0.1.49 --kill "$SIM"
```

---

## 测试矩阵与用例覆盖

### 执行矩阵

| 标记 | 环境或姿态 | 最低覆盖要求 |
| --- | --- | --- |
| `A` | 合盖外屏（单屏） | 冷启动、窄宽安全区、单栈导航、打开键盘 |
| `B` | 全展开内屏 | 单列自适应、应用内列表与详情双栏、图集或表格 |
| `C` | 半开内屏 | 书本或桌面姿态，检查折叠保留区与控件热区 |
| `D` | 系统 Split View | 与另一 App 并排、交换左右位置与焦点切换 |
| `A↔B` | 外屏与内屏双向切换 | 稳态与交互中均测（输入、滚动、弹窗、请求进行中） |
| 对照 | 普通 iPhone 与 Duo 真机 | 同构建同数据复测疑似兼容问题；真机验收推送、相机与性能 |

### 用例构成（共 144 条）

- **54 条新增公共用例（`PUB-001`–`PUB-054`）**：
  - 单屏与姿态（`PUB-001`–`PUB-007`）
  - 双栏与系统分屏（`PUB-008`–`PUB-016`）
  - 控件与手势（`PUB-017`–`PUB-025`）
  - H5 与混合容器（`PUB-026`–`PUB-034`）
  - 上传、下载与文件（`PUB-035`–`PUB-043`）
  - App 回归与发布验收（`PUB-044`–`PUB-054`）
- **90 条通用基线用例（`M1`–`M7`）**：
  - 姿态与连续性、原生展示、控件与手势、H5 与混合页、登录与会话、App 功能回归、消息推送、安装与升级、性能与稳定性。

---

## 取证与排障

当浏览器镜像出现黑帧或应用意外退回主屏时，通过“Device Hub + 活动显示面系统截图 + SpringBoard 日志”同步取证，避免把镜像断连误判为渲染缺陷：

```bash
mkdir -p evidence
# 按实际活动显示面 ID 截图（示例中的 3 请替换为本机真实 display ID）
xcrun simctl io "$SIM" screenshot --display=3 evidence/inner.png

# 检查 serve-sim 镜像健康状态
npx --yes serve-sim@0.1.49 doctor -d "$SIM"

# 过滤 SpringBoard 中目标 Bundle ID 的进程退出或生命周期日志
xcrun simctl spawn "$SIM" log show --last 10m --style compact \
  --predicate 'process == "SpringBoard" AND eventMessage CONTAINS[c] "com.example.testapp"'

# 校验切屏传输文件完整性与模拟推送
shasum -a 256 evidence/source.pdf evidence/downloaded.pdf
xcrun simctl push "$SIM" "$BUNDLE" evidence/test-notification.apns
```

---

## 设计原则

1. **先对照、后折叠**：先在普通 iPhone 上用同构建、同数据跑通主流程，再测 Duo 姿态，快速剥离通用业务 Bug 与折叠屏专项 Bug。
2. **重“操作中切换”**：页面静止时折叠只能测出静态重排；真正的问题往往暴露在键盘开启、弹窗遮罩或 `A/B` 异步请求并发时。
3. **高风险页面双断言**：既验证控件可见且可点击（UI 可用性），也验证对象 ID 未漂移、提交不重复（任务状态一致性）。
4. **证据隔离归档**：截图、进程日志与服务端记录统一绑定用例编号（`PUB-xxx`），严禁跨轮次拼凑结论。

---

## 常见问题

- **Q：模拟器能完全替代 iPhone Duo 真机测试吗？**
  - **A**：不能。模拟器适合做布局、导航栈、H5 视口与交互回归；真实 APNs 送达、应用商店/渠道安装升级、物理相机硬件、发热、电耗及发布性能必须在 Duo 真机环境下验收。
- **Q：为什么 Safari 27.1 的姿态与视口分段 API 需要开关？**
  - **A**：根据 WebKit 官方对 Safari 27.1 的说明，姿态与视口分段 API 属于需要开启功能开关的预览特性。业务回归应先测默认配置下的降级表现，再在明确开启开关的环境中验证实验能力。
- **Q：合盖后浏览器画面黑了怎么归因？**
  - **A**：同一时刻分别查看 Device Hub 与执行 `xcrun simctl io "$SIM" screenshot --display=<id>`。若两边画面均正常，则归因为 `serve-sim` 镜像流问题；若两边均异常，再继续排查 App 或系统状态。

---

## 路线图

- [x] 完成 144 条结构化公共测试用例（90 条基线 + 54 条 Duo 专项 `PUB-001`–`PUB-054`）
- [x] 提供 5 工作表 Excel 执行追踪工作簿（`data/iphone-duo-public-test-cases.xlsx`）
- [x] 整理 Xcode 27.2 + `serve-sim@0.1.49` + Codex 模拟器自动化测试指南
- [x] 补齐中英双语文档（`README.md` / `README.zh-CN.md`）与交互式项目主页（`index.html`）
- [ ] 补充可复用的取证打包辅助脚本（自动采集截图、日志与 SHA-256 校验）
- [ ] 提供示例 `.apns` 推送测试 payload 与最小化 `WKWebView` H5 测试页

---

## 贡献指南

欢迎补充真实项目踩坑记录、自动化脚本或优化现有用例！

- 有疑问或想讨论新的折叠屏测试场景，欢迎前往 [GitHub Discussions](https://github.com/xsoway/iphone-duo-test/discussions) 发起讨论。
- 提交 PR 时请同步维护 `README.md` 与 `README.zh-CN.md` 的内容一致性，并确保不引入任何本机绝对路径或敏感凭据。

---

## License

本项目采用 [MIT License](./LICENSE) 开源协议。

---

## 维护者

- **Alan Hsu** ([@xsoway](https://github.com/xsoway))
