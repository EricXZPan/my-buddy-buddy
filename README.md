# my-buddy-buddy

个人技术资料库，记录 iOS/Swift 实践与研究、Flutter 到 SwiftUI 的迁移方案、AI 编程工具调研，以及按月份整理的技术阅读存档。仓库主要由文档、示例源码和辅助脚本组成，不是一个统一构建的应用。

## 内容导航

### iOS 与 Swift

- [外接显示与场景状态恢复（UIKit）](sample_code/ExternalDisplayStateRestoration/README.md)：主场景、外接屏、次级场景切换和状态恢复 Demo。
- [外接显示与场景状态恢复（SwiftUI）](sample_code/ExternalDisplayStateRestorationSUI/README.md)：SwiftUI 主界面与 UIKit 外接屏桥接 Demo。
- [跨 Framework 触发 UIScene 切换](sample_code/CrossFrameworkSceneSwitch/README.md)：在 Framework 独立编译、不能共享进程内状态等约束下，由主 App 协调场景创建和内容加载的设计说明。
- [语音键盘与灵动岛示例](sample_code/voice_keyboard/VoiceKeyboardDynamicIsland.swift)：键盘扩展、主 App 录音和 Live Activity 协作示例；不是可直接构建的完整工程。
- [iOS 技术研究与笔记](docs/)：SwiftUI/UIKit、启动和性能、MetricKit、dyld、mergeable libraries、iOS 27 与 iPhone Duo 等主题。

### Flutter 迁移

[Flutter → SwiftUI 原生重写专题](flutter-to-swiftui/README.md)包含迁移方案、历史版本、Screen Spec 与 kickoff 模板、样例工程和 Flutter 组件清单扫描工具。当前方案入口是[方案 V4](flutter-to-swiftui/00-方案V4-暂定最终版.md)；若要先用最小成本验证可行性，见 [POC 制作方案](flutter-to-swiftui/POC.md)。扫描器使用 Python 标准库，可按专题 README 的命令运行并用附带样例验证。

### AI 工具与 Agent

- [AI Agent 设计模式](docs/ai-agent-design-patterns.md)、[自然语言 Agent Harnesses](docs/Natural_Language_Agent_Harnesses_总结.md)和[提示词工程指南](docs/prompt-engineering-guide.md)。
- OpenClaw/Copaw 调研：[Areal 分析](docs/areal_analysis.md)、[Areal 与 OpenClaw 集成](docs/areal_openclaw_integration.md)、[Copaw 分析](docs/copaw_analysis.md)、[OpenClaw 安装指南](docs/openclaw-install-guide.md)。
- [Copilot Chat 资料](docs/copilot-chat/)记录 API 配置和端点选择；[OpenCode 资料](docs/opencode/)记录架构和请求头相关信息。
- [iOS 项目生成器 Skill](skills/ios-project-generator.skill.md)及配套文件位于 `skills/`；OpenCode 配置 schema 位于 `schema/opencode/`。

### 阅读存档

- [GitHub Trending](github-trending/)：按 `YYYY-MM/` 月目录归档，文件通常包含具体日期。
- [Hacker News](HackerNews/)：包含按月存档、按日命名的记录，以及部分早期根目录记录。
- [生活与新闻资料](life/)：按月份整理的主题内容。

这些目录是阅读存档；仓库不包含用于保证每日抓取或自动发布的调度配置。文件命名会因主题和时期有所不同，新增内容时可沿用对应月份附近的格式。

## 目录结构

```text
.
├── docs/                    技术研究、指南、调研和辅助脚本
│   ├── copilot-chat/        Copilot Chat API 笔记
│   └── opencode/            OpenCode 架构与请求头笔记
├── flutter-to-swiftui/       迁移方案、历史文档、模板、样例和扫描工具
├── github-trending/          GitHub Trending 月度存档
├── HackerNews/               Hacker News 月度与历史存档
├── life/                     生活与新闻主题存档
├── sample_code/              iOS/Swift 示例源码与专题说明
│   ├── CrossFrameworkSceneSwitch/
│   ├── ExternalDisplayStateRestoration/
│   ├── ExternalDisplayStateRestorationSUI/
│   └── voice_keyboard/
├── schema/opencode/           OpenCode 配置 schema
├── skills/                   Agent skill 文件
├── LICENSE
└── README.md
```

## 使用说明

- 仓库没有统一的依赖安装、构建或测试命令；多数内容是 Markdown 文档或示例源码。
- Flutter 组件扫描器的用法、参数和样例校验方式见[专题 README](flutter-to-swiftui/README.md)。脚本不需要第三方 Python 包或 Dart SDK。
- 外接显示示例的 Xcode 配置步骤见各自目录的 README。运行并验证外接屏行为需要兼容的 Apple 开发环境与设备。
- `docs/strip_mergeable_metadata.py` 和 `docs/strip_mergeable_metadata.sh` 用于处理 framework 二进制的辅助脚本。使用前请先阅读脚本，并确认目标文件及依赖工具。

## 许可证

本仓库采用 MIT License，详见 [LICENSE](LICENSE)。
