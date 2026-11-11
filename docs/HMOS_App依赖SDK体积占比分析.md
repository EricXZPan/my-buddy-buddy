# HMOS App 包依赖 SDK 体积占比分析（归纳总结）

> 调研时间：2026-11-12 ｜ 信息源：华为官方文档、OpenHarmony/arkcompiler 源码、OHPM、社区实测案例
> 定位：一页纸归纳——包结构 → 依赖体积怎么算 → 怎么归因 → 怎么压

---

## 1. 结论速览（TL;DR）

1. **HAP/APP 本质是 ZIP**，依赖 SDK 体积 = so 文件（直接归因）+ resources 归属文件 + `modules.abc` 内 record/方法字节码（需反汇编归因）
2. **HAR 是静态拷贝**：被每个引用方各复制一份，最坏膨胀公式 `(HAP数 + HSP数) × ΣHAR体积`；OHPM 托管的库全是 HAR，这是依赖体积失控的第一根源
3. **HSP 动态单实例**：恒一份；DevEco 6.0.1+ 支持 `deduplicateHar` 自动去重
4. **ABC 逐依赖归因**：`ark_disasm --verbose` 输出每 record 的 offset+size，record 名含库名/版本（`@normalized:` 前缀），按前缀聚合 + 方法 offset 区间累加即得依赖排名
5. **审计级精确值**仍靠 A/B 差量构建（Release 模式）；abc 归因的价值是**一次构建出全依赖排名**
6. **永远用 Release 包分析**：Debug 不压缩资源、含 sourceMaps.map，量出来全是噪音

---

## 2. 包结构与典型体积占比

### 2.1 三层视图

| 视图 | 内容 | 关键点 |
|---|---|---|
| 开发态 | entry/feature/library + `oh-package.json5` | 依赖在此是 `.har`（oh_modules 目录） |
| 编译态 | HAP/HSP（**HAR 被编译进引用方，包内无 .har 文件**） | 代码合并进 `modules.abc` |
| 发布态 | `.app`（App Pack）= Σ HAP + Σ HSP + pack.info | 上架单元；分发安装仍按 HAP/HSP |

### 2.2 Release HAP 内部结构与典型占比

```
entry-default-signed.hap (ZIP)
├── ets/modules.abc     ← 代码字节码（含 HAR 编译产物）     5-15%（纯 ArkTS 工程更高）
├── libs/<abi>/*.so     ← 原生库（默认零压缩！）           10-50%
├── resources/          ← 资源                           30-60%（通常最大头）
├── resources.index / module.json / pack.info / pkgContextInfo.json
```

| 组成 | 典型占比 | 优化空间 |
|---|---|---|
| 图片资源 | 30%-50% | ⭐⭐⭐⭐⭐ WebP/SVG/去重 |
| 第三方库 | 15%-30% | ⭐⭐⭐⭐ HAR→HSP、版本统一 |
| Native 库(.so) | 10%-25% | ⭐⭐⭐ ABI 裁剪 + compressNativeLibs |
| 代码(.abc) | 5%-15% | ⭐⭐ 混淆 + TreeShaking |
| 其他资源 | 5%-10% | ⭐⭐⭐ ignoreResourcePattern |

### 2.3 共享包对体积的影响

| 类型 | 链接方式 | N 个模块引用时 |
|---|---|---|
| **HAR** | 静态，编译进每个引用方 | `N × HAR体积`（重复膨胀） |
| **HSP** | 动态，运行时单实例 | **1 份** |
| feature HAP | 按需分发 | 不占首装包 |

> 元服务硬约束：单包（含依赖 HSP）≤ 2MB、总包 ≤ 10MB——依赖体积计算是刚需。

---

## 3. 依赖 SDK 体积计算方法（五层，粗→精）

| # | 方法 | 精度 | 命令/入口 |
|---|---|---|---|
| 1 | oh_modules 直查 | ❌ 粗估 | `ohpm ls` + `du -sh oh_modules/*`（≠实际入包，TreeShaking 会裁） |
| 2 | 解压 HAP 文件级归因 | ⭕ 中 | HAP 即 ZIP；so/资源按文件名归因；`pkgContextInfo.json` 看依赖清单 |
| 3 | **app-check-tool**（官方） | ⭕ 高 | `java -jar app_check_tool.jar --input xxx.app --out-path ./report --stat-duplicate true`；输出重复文件 md5+位置+浪费字节 |
| 4 | DevEco **Build > Analyze App Size** | ⭕ 高 | 树状占比 + 压缩前后对比 + 依赖关系图 |
| 5 | **A/B 差量构建** | ✅ 100% | 注释依赖前后 Release 包体积差；唯一精确归因 ArkTS SDK 的办法 |

### 总公式

```
App Pack 体积 = Σ HAP(压缩后) + Σ HSP(压缩后) + pack.info
某 SDK 贡献 ≈ Σso归属文件 + Σresources归属文件 + abc差量(方法5测得)
HAR 最坏贡献 = (HAP数 + HSP数) × HAR原始体积
```

---

## 4. modules.abc 逐依赖归因（进阶）

### 4.1 ABC（Panda File）组成

| 区域 | 内容 | 可归因性 |
|---|---|---|
| Header | 魔数 `PANDA\0\0\0`、Adler32、版本号 | ❌ 全局 |
| 索引区 | class/method/string/literal 索引（16 位引用） | ❌ 全局 |
| **Record 区** | **1 源文件 = 1 record**（类元数据/字段/moduleRecordIdx） | ✅ |
| **Method 区** | 方法定义+字节码；每模块自动生成 `func_main_0` | ✅ |
| LITERALS | 常量数组 + `MODULE_REQUEST_ARRAY`（import 表） | ⚠️ 共享 |
| String 池 | UTF-8 常量 | ⚠️ 共享 |

除依赖代码外，abc 内还有：自研业务代码 records、编译器胶水（`func_main_0`、`_ES*Annotation` 固定 record）、共享常量池与索引开销。

### 4.2 归因操作

```bash
# sdk/default/openharmony/toolchains/ark_disasm.exe
ark_disasm.exe --verbose modules.abc modules.pa
```

```
.record entry/src/main/ets/pages/Index { # offset: 0x00f3, size: 0x0098 (152) }
.function any xxx.func_main_0(...) { # offset: 0x0153, code offset: 0x0245 }
```

1. 按 record 名前缀分组；HAR 依赖为 `@normalized:库名&版本&路径`（需 `useNormalizedOHMUrl: true`）
2. `依赖体积 ≈ Σ record.size + Σ 该 record 方法 (end_offset − code_offset)`
3. 聚合输出"依赖→字节数"排名

**坑**：① ArkGuard 混淆会改 record/方法名（归因用未混淆构建）；② `record.size` 不含方法字节码，必须另行累加；③ String 池/LITERALS 全局共享不可完全归属。

### 4.3 替代工具

- **ABCDE**（github.com/Yricky/abcde，Kotlin）：拖 `.hap` 免解压，树形看 record/方法，自带包体积分析示例，适合平台化集成
- **abc2program**（arkcompiler_runtime_core 官方）：abc→pandasm，可编程统计

---

## 5. 治理手段优先级

| 优先级 | 手段 | 配置位置 |
|---|---|---|
| P0 | so 压缩 `compressNativeLibs: true` | 每模块 module.json5 |
| P0 | ABI 只留 arm64-v8a | build-profile.json5 `abiFilters` |
| P0 | 版本统一：项目级 `overrides` / ohpm 1.5+ `resolve_conflict` | 项目级 oh-package.json5 |
| P1 | HAR→HSP（或伞形 HSP 收敛三方 HAR） | 模块重构 |
| P1 | 混淆+TreeShaking（反射需 `-keep` 白名单） | obfuscation-rules.txt |
| P1 | `ignoreResourcePattern` 剔除无用资源 | build-profile.json5（API 19+） |
| P2 | Feature HAP 按需分发 | 多 HAP 架构 |
| P2 | CI 体积门禁（size-baseline.json，增量>1MB 阻断） | CI 脚本 |

### 避坑清单
1. Debug 包不能用于体积分析（资源未压缩 + sourceMaps.map）
2. `overrides` 只在项目级生效；它是"统一版本"工具，不能隔离多版本
3. 资源覆盖优先级：AppScope > HAP 自身 > HAR（HAR 同名资源被静默顶掉）
4. 清理后按模块二次对比，构建系统可能把体积"补"到别的模块
5. resources.index / src 下非代码文件明文入包，密钥别放资源

---

## 6. 推荐工作流

```
选型期   : ohpm ls + du -sh oh_modules/*        → 排除体积离谱的 SDK
联调期   : DevEco Build Analyzer                 → 早发现 HAR 重复
发版前   : app-check-tool 扫 .app（重复+大文件+类型占比）
精确归因 : ark_disasm --verbose 解析 .pa 聚合 / A/B 差量构建（Release）
长期     : CI 体积门禁 + 季度依赖体检
```

---

*信息源：华为官方文档（应用程序包结构/构建产物说明/扫描工具/优化包体积最佳实践/Disassembler）、OHPM 官方帮助、arkcompiler_runtime_core 源码、ABCDE、社区实测（掘金/CSDN/华为开发者论坛）*
