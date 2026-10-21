# my-buddy-buddy

个人技术资料库，集中保存 iOS/Swift 实践与研究、跨框架架构笔记、AI 编程工具调研，以及按月份整理的技术和新闻阅读记录。仓库由文档、示例源码和少量辅助工具组成，不是一个统一构建的应用。

## 内容导航

### iOS 与 Swift

- [外接显示与场景状态恢复（UIKit）](ExternalDisplayStateRestoration/README.md)：主场景、外接屏、次级场景切换和状态恢复示例。需要在 Xcode 项目中配置并加入本目录源码。
- [外接显示与场景状态恢复（SwiftUI）](ExternalDisplayStateRestorationSUI/README.md)：SwiftUI 主界面与 UIKit 外接屏桥接示例，同样需要自行创建 Xcode 项目。
- [跨 Framework 触发 UIScene 切换](CrossFrameworkSceneSwitch/README.md)：在 Framework 独立编译、不能共享进程内状态等约束下，由主 App 协调场景创建和内容加载的设计记录。
- [语音键盘与灵动岛代码示例](code/VoiceKeyboardDynamicIsland.swift)：展示键盘扩展、主 App 录音和 Live Activity 之间的协作；这是代码示例，不是可直接构建的完整工程。
- [iOS 技术研究与笔记](docs/)：涵盖 SwiftUI/UIKit、启动和性能、MetricKit、dyld、mergeable libraries、外接显示与 iPhone Duo 等主题。

### Flutter 迁移

[Flutter → SwiftUI 原生重写专题](flutter-to-swiftui/README.md)包含迁移方案、历史版本、Screen Spec 和 kickoff 模板、样例工程，以及 Flutter 组件清单扫描工具。当前方案入口为 [00-方案V4-暂定最终版](flutter-to-swiftui/00-方案V4-暂定最终版.md)；扫描器使用 Python 标准库，可按专题 README 中的命令运行并用附带样例验证。

### AI 工具与 Agent

- [AI Agent 设计模式](docs/ai-agent-design-patterns.md)、[自然语言 Agent Harnesses](docs/Natural_Language_Agent_Harnesses_总结.md)、提示词工程及 OpenClaw/Copaw 相关调研位于 `docs/`。
- [Copilot Chat 资料](docs/copilot-chat/)记录 API 配置和端点选择；[OpenCode 资料](docs/opencode/)记录架构和请求头相关信息。
- [iOS 项目生成器 Skill](skills/ios-project-generator.skill.md)及其配套 skill 文件位于 `skills/`；OpenCode 配置 schema 位于 `schema/opencode/`。

### 阅读存档

- [GitHub Trending](github-trending/)：按 `YYYY-MM/` 月目录归档，内容文件通常带有日期。
- [Hacker News](HackerNews/)：按月归档的精选记录，也保留部分旧的日期命名文件。
- [生活与新闻资料](life/)：按月份整理的主题内容。

这些目录是阅读存档；仓库不包含用于保证每日抓取或自动发布的调度配置。文件命名会因主题和时期有所不同，新增内容时沿用对应月份附近的现有格式即可。

## 目录一览

```text
.
├── code/                                Swift 代码示例
├── CrossFrameworkSceneSwitch/           跨 Framework 场景切换设计说明
├── ExternalDisplayStateRestoration/     UIKit 外接屏与状态恢复示例
├── ExternalDisplayStateRestorationSUI/  SwiftUI 外接屏与状态恢复示例
├── flutter-to-swiftui/                  Flutter 迁移方案、模板、样例和扫描工具
├── github-trending/                     GitHub Trending 月度归档
├── HackerNews/                          Hacker News 阅读归档
├── life/                                生活与新闻主题归档
├── docs/                                技术研究、指南和工具脚本
├── schema/opencode/                     OpenCode 配置 schema
├── skills/                              Agent skill 资料
├── LICENSE
└── README.md
```

## 运行与环境

- 仓库没有统一的依赖安装、构建或测试命令。多数内容是 Markdown 文档或示例源码。
- Flutter 扫描器的用法、参数和样例校验方式见 [专题 README](flutter-to-swiftui/README.md)；脚本本身不要求安装第三方 Python 包，也不要求 Dart SDK。
- 外接显示示例的 Xcode 配置步骤见各自目录的 README。运行和验证外接屏行为需要兼容的 Apple 开发环境与设备。
- `docs/strip_mergeable_metadata.py` 和 `docs/strip_mergeable_metadata.sh` 是针对 framework 二进制的辅助脚本，不是普通文档生成工具；使用前请先阅读脚本并确认目标文件及所需工具。

## 许可证

本仓库采用 MIT License，详见 [LICENSE](LICENSE)。
