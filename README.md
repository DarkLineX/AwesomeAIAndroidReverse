# Awesome AI Android Reverse

> Android 逆向、移动安全分析与 AI 辅助逆向工具/文章收集。
>
> 最后更新：2026-11-11

本仓库只做公开资料索引，面向授权安全测试、恶意样本分析、漏洞研究、CTF 和学习用途。请勿用于未授权逆向、绕过商业授权、侵犯隐私或其他违法场景。


## 目录

- [前置：用哪家模型做 AI 辅助逆向](#前置用哪家模型做-ai-辅助逆向)
- [破甲（jailbreak）提示词研究](#破甲jailbreak提示词研究知道边界别当生产力)
- [AI / MCP 逆向工具](#ai--mcp-逆向工具)
- [AI Skills、Prompts 与 Agent 工作流](#ai-skillsprompts-与-agent-工作流)
- [LLM 解混淆、漏洞发现与算法还原](#llm-解混淆漏洞发现与算法还原)
- [基础 Android 逆向工具链](#基础-android-逆向工具链)
- [动态分析、Hook 与抓包辅助](#动态分析hook-与抓包辅助)
- [Flutter / Native / 二进制分析](#flutter--native--二进制分析)
- [文章、论文与教程](#文章论文与教程)
- [推荐工作流](#推荐工作流)
- [收录标准](#收录标准)

## 前置：用哪家模型做 AI 辅助逆向

同一份 APK，换一个模型，结论准确度和稳定性差别很大。下面按任务给判断框架，不抄排行榜；排名和价格变化极快，数字统一截至 2026-11-11。

| 任务 | 首选 | 备选与理由 |
|---|---|---|
| 长任务 agentic 编码：写 Frida 脚本、改 smali、多文件重构 | Claude Opus 5.5 | GPT-6.1 Sol 约 85% 成功率、约 1/8 成本；Kimi K3.1 Pro 差 2 分、约 1/3 成本 |
| 整包上下文：几十万行 smali、全量 jar、大型 so 反编译结果一次读入 | Gemini 4 Argon、GPT-6 Astra（约 1M） | Kimi K3.1 Pro（1,048,576，开源权重）、Claude Fable 5.2（约 1M，$8/M 输入偏贵） |
| 工具调用 / MCP 链路 | Claude Opus 5.5（终端类基准领先） | 现有 jadx-ai-mcp、GhidraMCP、ida-pro-mcp 的文档与默认配置基本以 Claude Code / Cline / Codex 为首测客户端 |
| 敏感样本、样本不能出网 | DeepSeek V4.1 Flash（MIT）、Qwen3.8-Max（开放权重） | GLM-5.3（$1.40/M 输入）、Kimi K3.1 Pro；本地权重是唯一能保证样本不出内网的方案 |
| 高频批量：批量解混淆、批量命名、批量出报告 | DeepSeek V4.1 Flash（$0.15/$0.60，缓存 $0.003） | Gemini 3.8 Flash（$0.75/$3.75）、Claude Haiku 5.5（$0.10/$0.50，注意长 prompt 的计价阈值） |
| 防守型安全任务：样本分析、写检测规则 | 厂商信任访问计划 | Gemini 4 Carbon 经 Fairwind 通道、GPT-6 Sol trusted access；通用模型对多数 cyber 任务直接拒答 |

四条实操原则：

1. **按失败代价选，不按榜单选。** 错了要重跑的批量任务用便宜模型，错一次就毁结论的用最强的。定版前拿自己的 30–50 条真实任务（自己的 APK、自己的 trace）做对照，公开榜单只用来缩小候选范围。
2. **1M 上下文不等于该塞 1M。** 整包灌进去通常又贵又容易丢关键位置；先用 jadx / apktool 分模块定位，再把可疑的类与调用链交给模型。
3. **客户端兼容性优先于模型强度。** MCP Server 与 Skill 生态第一适配的是 Claude Code / Codex / Cline；换模型先验证工具调用、长任务中断、结构化输出是否还稳，再谈推理能力。
4. **APK 是证据，不是普通文本。** 未脱敏的客户包、私有 so、内网地址不要直传云端 API；涉外或涉密样本走本地权重。

口径来源：[Best LLMs of 2026](https://iadecider.com/articles/best-llm-2026)、[Best LLM for AI Agents（2026-11）](https://o-mega.ai/articles/best-llm-for-ai-agents-november-2026-ranking)。同一个模型换一套 harness 分数能差几十点，榜单只能用来排除，不能当结论。

## 破甲（jailbreak）提示词研究：知道边界，别当生产力

通用模型对「分析这个样本的 C2 通信」「绕过这个 App 的证书校验」这类请求会高频拒答，这是选型绕不开的现实约束。合规解法是厂商的信任访问 / 红队计划；另一条是所谓「破甲」——用身份设定、授权语气和措辞把模型的拒绝倾向「说服」掉。它不改权重、不碰二进制、不绕鉴权，是提示词层的行为偏移，不是技术攻破。了解这套东西有助于判断模型行为，但把它当生产依赖会反噬。

- [gpt-instruct](https://github.com/MDX-Tom/gpt-instruct) - 面向 Codex 的破甲提示词与测试包（MIT，11.3k★，2026-10 仍在提交）。做法是用官方 `model_instructions_file` 把一份 Markdown 装成全局指令；真正值得看的是工程化：三条产品线、A/B/C 三层发布门禁、SHA256 校验、原子写入与字段级回滚、JailbreakBench 适配层。成绩也没被神话——`gpt-6-astra-v1` 在 B 阶段 66 cases 只过 52、74 turns 过 60、工件门禁 15/16，未达 66/66 硬门槛。作者自己在 README 写明「存在账号风险，建议在日抛账号上使用」。本仓库只记录机制，不转载其提示词内容。
- [gpt-instruct 源码审计](https://github.com/Quriosity-agent/articles/blob/main/2026-10-09/2026-10-09-gpt-instruct-codex-instruction-eval-audit.md) - 2026-10-09 的独立审计，目前对这类项目最硬的一份拆解：确认它是「全局 agent policy patch」，不是新模型也不是 Skill；实测部署器 27/27、评分合约 49/49 通过，但 `tests/`、`reports/`、`docs/architecture/` 在公开快照里都是 0 个跟踪文件，所以**部署可复现、能力分数不可独立复算**；同时解释了客户端指令为什么关不掉服务端策略与账户权限。

三条成本自己算清：

- **账号**：厂商按行为判风控，封号限权是直接损失，作者自己都建议日抛账号。
- **可靠性**：提示词对模型版本极敏感，模型一更新即失效，自建测试集都过不了八成。
- **判断力**：一个「有求必应」的 Agent 不再对错误请求说不，错误会落到你的代码库和样本结论上。

## AI / MCP 逆向工具

- [jadx-ai-mcp](https://github.com/zinja-coder/jadx-ai-mcp) - JADX GUI 插件，把当前类、Manifest、资源、xref、调试信息等上下文暴露给支持 MCP 的 LLM 客户端。
- [jadx-mcp-server](https://github.com/zinja-coder/jadx-mcp-server) - 独立 Python MCP Server，连接 JADX-AI-MCP 插件，让模型读取反编译代码、搜索类/方法/字段、辅助审计 APK。
- [apktool-mcp-server](https://github.com/zinja-coder/apktool-mcp-server) - 基于 Apktool 的 MCP Server，支持解包、Manifest/resource/smali 读取与修改、重打包等 APK 修改流程。
- [GhidraMCP](https://github.com/LaurieWired/GhidraMCP) - Ghidra MCP Server，把反编译、函数、符号与程序结构上下文提供给 LLM。
- [ida-pro-mcp](https://github.com/mrexodia/ida-pro-mcp) - IDA Pro MCP Server，支持 headless / GUI 流程，面向 IDA 内的变量重命名、类型调整、注释、报告生成等 vibe reversing 场景。
- [REA (Reverse Engineer Anything)](https://github.com/morluto/rea) - 2026-10 起高速迭代的开源 CLI/MCP（`npx rea-agents setup`），把 Hopper / Ghidra / IDA / JADX / JEB / Frida 等引擎统一暴露给编码 Agent，返回带来源、置信度与未知项的 Evidence bundle，分析全程在本地；4.1 起原生支持 headless JADX 静态 APK 分析、Manifest 与 ADB 设备交互，并覆盖固件、Electron、.NET 等目标，深度 Native 分析仍需自备相应引擎与许可。附带 [reverse-engineer-anything Skill](https://github.com/morluto/rea/tree/main/.agents/skills/reverse-engineer-anything)。
- [IDA-NO-MCP](https://github.com/P4nda0s/IDA-NO-MCP) - 针对 IDA Pro 的轻量 AI 交互方案，目标是降低传统 MCP 交互的复杂度与延迟。
- [frida-mcp](https://github.com/zhizhuodemao/frida-mcp) - Frida 动态分析 MCP Server，支持应用启动/附加、前台应用识别、应用列表、脚本注入和自定义 frida-server 配置。
- [ZIN-MCP-Client](https://github.com/zinja-coder/zin-mcp-client) - 面向本地 LLM / Ollama 的命令行 MCP Client，可连接 JADX、Apktool 等逆向 MCP Server。
- [jshookmcp](https://github.com/vmoranv/jshookmcp) - 「搜索优先」的 Agent 逆向工作区，735 个工具按 36 个域按需加载（默认仅约 3K tokens 工具元数据），覆盖 CDP 浏览器 Hook、JS 解混淆、WASM 反汇编、Frida / Ghidra / IDA 桥接以及 APK 静态分诊（apktool / jadx / unidbg）；AGPL-3.0、Node.js 22+，2026-11 仍在活跃提交。仓库含反检测与验证码相关模块，仅限授权测试使用。
- [ghidra-mcp](https://github.com/bethington/ghidra-mcp) - 把 Ghidra 的反编译、符号、交叉引用、内存与 patch 能力拆成 200 多个 MCP 工具，同时提供 GUI 插件和 headless 服务端，按需懒加载工具清单以控制上下文；Apache-2.0，2026-11 仍在更新。
- [open-reverselab](https://github.com/LING71671/open-reverselab) - 开源的 AI 逆向 Agent 平台，一套 MCP 同时接 Ghidra、Frida、x64dbg 和 Rizin，把静态反编译与动态调试串成可自动执行的调查流程。
- [ReArk](https://github.com/lkimuk/ReArk) - 面向 HarmonyOS NEXT 与 Android 的 AI 辅助逆向工具，支持 HAP/APP/ABC 与 APK 的结构解析、反编译和还原；做鸿蒙样本或双端对比时补上现有工具链的空缺。
- [binary-ninja-headless-mcp](https://github.com/mrphrazer/binary-ninja-headless-mcp) - Binary Ninja 的 headless MCP 服务，向 Agent 暴露约 180 个工具做反汇编、反编译与交叉引用分析，适合把 Binary Ninja 接进批量流水线。

## AI Skills、Prompts 与 Agent 工作流

- [android-reverse-engineering-skill](https://github.com/SimoneAvogadro/android-reverse-engineering-skill) - Claude Code Android 逆向 Skill，覆盖 APK/XAPK/JAR/AAR 反编译、API 提取、Kotlin/R8 名称恢复、Ktor/Apollo/Koin/HMAC 识别等流程。
- [ai-reverse-toolkit](https://github.com/zhizhuodemao/ai-reverse-toolkit) - 面向逆向分析的 skills、rules 和 prompts 集合，包含加密入口定位、补环境、AST 解混淆等任务模板。
- [mcp-reversing-dataset](https://github.com/mrexodia/mcp-reversing-dataset) - IDA Pro MCP 示例数据与演示素材，可用于验证 LLM 逆向流程和提示词。
- [Code2MCP / MCP-Github-Agent](https://github.com/DEFENSE-SEU/MCP-Github-Agent) - 将 GitHub 代码仓库自动转换为 MCP 服务的多 Agent 框架，可参考其 MCP 化思路扩展逆向工具。
- [android-reverse-engineering-skill (Codex)](https://github.com/CreditTone/android-reverse-engineering-skill) - 专为 Codex 适配的 Android 逆向 skill，可在会话里反编译 APK/XAPK/JAR/AAR，并结合 jadx、Fernflower/Vineflower 梳理 Manifest、入口与调用链。
- [MobileRE-Skill](https://github.com/index-login/MobileRE-Skill) - AI Agent 驱动的移动端逆向技能集，覆盖 Frida hook、一键脱壳、内存 DEX dump 与 Ghidra MCP 符号/结构恢复。

## LLM 解混淆、漏洞发现与算法还原

- [Androidmeda](https://github.com/In3tinct/Androidmeda) - LLM 驱动的 Android 代码解混淆与漏洞扫描工具，支持 OpenAI、Gemini、Anthropic API 和 Ollama 本地模型。
- [AlgoKiller](https://github.com/lidongyooo/AlgoKiller) - 面向 ARM64 执行 trace 的算法还原 harness，驱动 LLM 从 trace 中追踪数据流、恢复加密/签名/编码逻辑，并输出 Python 复现代码或结构化报告。
- [SecretLoc](https://arxiv.org/abs/2510.18601) - LLM 检测 Android App 硬编码 secrets 的研究，强调上下文和结构线索对未知密钥类型的发现能力。
- [Evaluating LLMs for Obfuscation Detection and Classification in Android Apps](https://arxiv.org/abs/2606.14233) - 2026 年 Android 混淆检测论文，评估 LLM 在语义层面识别混淆代码的能力。
- [Can LLMs Deobfuscate Binary Code?](https://arxiv.org/abs/2604.08083) - BinDeObfBench，系统评测 LLM 对多阶段二进制混淆的反混淆能力。
- [Constraint-Guided Multi-Agent Decompilation](https://arxiv.org/abs/2604.23940) - 多 Agent 约束引导反编译框架，使用语法、编译和行为约束迭代修复反编译输出。
- [Towards LLM-Enhanced Android Taint Analysis](https://arxiv.org/abs/2608.24269) - 2026-08，LLM 经 JADX-MCP / Ghidra-MCP 迭代读取反编译代码并推断 source-to-sink 污点流，DroidBench 上 Gemini-3 Flash F1 0.96，FlowDroid 为 0.55。

## 基础 Android 逆向工具链

- [jadx](https://github.com/skylot/jadx) - Dex/APK/AAB/XAPK 到 Java 的反编译器，带 GUI、资源解码、搜索、xref、deobfuscator 和 smali debugger。
- [Apktool](https://github.com/iBotPeaches/Apktool) - Android APK 资源解码、smali 修改、重打包和调试的核心工具。
- [Frida](https://github.com/frida/frida) - 动态插桩框架，Android Java 层和 Native 层 Hook 的事实标准之一。
- [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) - 自动化移动应用安全分析平台，支持 Android/iOS 静态分析、动态分析、恶意样本分析、隐私分析和 CI/CD 集成。
- [Ghidra](https://github.com/NationalSecurityAgency/ghidra) - 开源二进制逆向平台，适合 Android Native so、JNI、算法还原与漏洞分析。
- [IDA Pro](https://hex-rays.com/ida-pro/) - 商业二进制分析和反编译平台，Android Native、固件、复杂算法逆向常用。
- [JEB Decompiler](https://www.pnfsoftware.com/) - 商业 Android/Dalvik/Native 反编译器，JEB 5.x 已加入 VIBRE AI assistant 和 MCP Server 相关能力。
- [Rizin](https://github.com/rizinorg/rizin) / [radare2](https://github.com/radareorg/radare2) - 开源二进制分析框架，适合脚本化、CLI 批处理和 Native 分析。
- [Bytecode Viewer](https://github.com/Konloch/bytecode-viewer) - Java 8+ 与 Android APK 的逆向套件，集反编译、字节码编辑、调试于一体，可在多种反编译器之间切换对比。
- [Androguard](https://github.com/androguard/androguard) - Python 的 APK/DEX 分析库与命令行工具，可解析 Manifest、权限、签名证书、字符串与调用图，适合写批量扫描脚本。
- [Simplify](https://github.com/CalebFenton/simplify) - Android 虚拟机与 smali 反混淆器，把混淆后的 Dalvik 代码还原成可读形式，常与 jadx 配合处理加固/混淆样本。
- [APK Studio](https://github.com/vaibhavpandeyvpz/apkstudio) - Qt6 跨平台 APK 逆向 IDE，把反编译、编辑、重打包与签名整合在同一个界面里。
- [APKLab](https://github.com/APKLab/APKLab) - VS Code 的 Android 逆向工作台，将 apktool、jadx、adb 串进编辑器，改 smali 与资源后可直接重打包安装。
- [Android_boot_image_editor](https://github.com/cfig/Android_boot_image_editor) - 解析与重打包 boot.img / vbmeta.img / payload.bin，已支持 Android 16；改机、脱壳环境准备和无 system 分区设备常用。
- [Reversense (Dexcalibur2)](https://github.com/reversenseorg/dexcalibur) - Dexcalibur 的重写版，Android 二进制智能分析与 Frida 脚本辅助生成，面向 DEX 层的方法定位与 hook 编排。
- [android_unpacker](https://github.com/CheckPointSW/android_unpacker) - Check Point 开源的通用 APK 脱壳器，针对整体加密与运行时自解密类壳。
- [RePairip](https://github.com/ispointer/RePairip) - 针对 Google PairIP 加固的逆向工具，dump VMRunner 快照并尝试完整重打包，处理加固样本时的一个补充选项。

## 动态分析、Hook 与抓包辅助

- [objection](https://github.com/sensepost/objection) - 基于 Frida 的移动运行时探索工具，支持 Android/iOS，常用于 SSL pinning 绕过、文件系统查看、内存 patch、对象探索等。
- [android-unpinner](https://github.com/mitmproxy/android-unpinner) - mitmproxy 出品的 APK 证书绑定移除工具，不要求 root，使用 Frida Gadget/JDWP 注入思路，支持 XAPK。
- [apk-mitm](https://github.com/niklashigi/apk-mitm) - 自动修改 APK 以便 HTTPS 抓包，处理 Network Security Configuration、常见证书绑定逻辑、重打包和签名。
- [HTTP Toolkit Frida Interception & Unpinning](https://github.com/httptoolkit/frida-interception-and-unpinning) - Frida 脚本在运行时重写移动应用网络栈，直接对全部 HTTPS 流量做 MitM；原 frida-android-unpinning 仓库已迁移至此，AGPL-3.0。
- [r0capture](https://github.com/r0ysue/r0capture) - Android 应用层抓包辅助脚本，常用于快速观察明文请求、响应和加密前后数据。
- [PriviSense](https://arxiv.org/abs/2601.22414) - Frida-based Android 多传感器/系统信号 spoofing 框架研究，可用于上下文敏感 App 行为复现实验。
- [RMS - Runtime Mobile Security](https://github.com/m0bilesecurity/RMS-Runtime-Mobile-Security) - Web 界面的移动运行时操控平台，把 Frida 的常用操作封装成按钮，适合快速验证 hook 点与内存修改。
- [Medusa](https://github.com/Ch0pin/medusa) - 模块化的移动安全分析框架，内置大量 Frida 模块，覆盖 SSL pinning、root/调试检测、加密 API 与常见风控点。
- [frida-scripts](https://github.com/0xdea/frida-scripts) - 个人维护的 Frida instrumentation 脚本集合，覆盖 Android/iOS 常见的类枚举、方法追踪与绕过场景。
- [magisk-frida](https://github.com/ViRb3/magisk-frida) - Magisk 模块形式在开机时拉起 frida-server，版本自动跟随上游，免去手动 push 与权限处理。
- [frinet](https://github.com/synacktiv/frinet) - Synacktiv 的 Frida tracer，支持 Android/iOS/Linux/Windows，跟踪函数调用、参数与返回值，适合先摸清调用链再下断点。
- [friTap](https://github.com/fkie-cad/friTap) - 基于 Frida 的加密流量分析工具，在 TLS 库层拦截明文，简化协议解密与密钥提取。
- [GumTrace](https://github.com/lidongyooo/GumTrace) - 基于 Frida Gum (Stalker) 的 ARM64 指令级追踪工具，支持 Android 与 iOS，适合配合 AlgoKiller 做算法 trace。

## Flutter / Native / 二进制分析

- [reFlutter](https://github.com/Impact-I/reFlutter) - Flutter 逆向框架，支持 Android/iOS Flutter App 的流量代理、dump.dart 提取和部分 pinning 绕过。
- [Blutter](https://github.com/worawit/blutter) - Flutter Dart AOT 反汇编/符号辅助工具，适合 Flutter Native 层分析。
- [Doldrums](https://github.com/rscloura/Doldrums) - Flutter snapshot 分析与 Dart 结构恢复工具。
- [SoFixer](https://github.com/F8LEFT/SoFixer) - Android ELF/so dump 修复工具，适合脱壳后 so 修复。
- [unidbg](https://github.com/zhkl0228/unidbg) - Android Native 库模拟执行框架，可用于 JNI 算法调用、签名函数复现和协议分析。
- [Qiling](https://github.com/qilingframework/qiling) - 多架构二进制模拟框架，可辅助 Android Native 算法环境搭建。
- [kill_flutter](https://github.com/f3rb123/kill_flutter) - Flutter 的 SSL pinning 绕过工具，声称不依赖 pattern 库、适配任意 Flutter 版本，Android 与 iOS 通用。
- [ghidriff](https://github.com/clearbluejar/ghidriff) - 基于 Ghidra 的命令行二进制 diff 引擎，做版本间函数匹配与差异分析，适合跟踪 so 更新的补丁点。
- [D810-ng](https://github.com/w00tzenheimer/d810-ng) - IDA Pro 的微码反混淆插件（D810 的继任版），针对 OLLVM 一类的控制流平坦化与指令替换做自动化清理。

## 文章、论文与教程

### 2026 / 2025 研究论文

- [Rethinking Neural Decompilation: Candidate Selection as the Hidden Bottleneck](https://arxiv.org/abs/2611.02788) - 2026-11，把 LLM 反编译在混淆二进制上「best-of-N」与单次输出的差距归因到候选生成与候选选择环节，指出继续堆候选数量收益有限。
- [VulDec: Vulnerability Detection in Binaries via Neural Decompilation](https://arxiv.org/abs/2610.33716) - 2026-10，构建汇编与源码对齐、带精确漏洞标注的大规模数据集，并用神经反编译输出做二进制漏洞检测。
- [Evidence-Grounded LLM-Assisted Android Mobile Forensic Reporting](https://arxiv.org/abs/2610.26946) - 2026-10，把 LLM 生成的 Android 取证报告拆成 612 条原子声明逐条核验，量化「表述流畅不等于证据可靠」。
- [SoK: Are LLMs Reliable at Source Code Recovery?](https://arxiv.org/abs/2610.11556) - 2026-10，对源码恢复方法做 taxonomy 与统一实证评测，回答 LLM 在恶意样本分析、漏洞评估中能否可靠还原源码。
- [When LLM Decompilers Recompile More and Preserve Less](https://arxiv.org/abs/2609.05370) - 2026-09，实证 LLM 反编译器的可重编译率与语义保持度不同步，只用「能编译、能跑」评测会高估效果。
- [MobileCybench: Evaluating Agent Vulnerability Discovery via Executable Probes](https://arxiv.org/abs/2609.23980) - 2026-09，13 个 Android App、495 个可执行 probe 的 Agent 漏洞发现基准；仅给混淆 APK 时最佳 Agent 在恶意 App 场景触发 53.8% 的应用，并发现 23 个此前未报告的漏洞。
- [When Agents See Differently: Exposing UI Desynchronization Threats in Mobile Agents](https://arxiv.org/abs/2609.16732) - 2026-09（v2 于 2026-11 修订），重打包 APK 可利用人类可见界面与 Agent 截图/无障碍表征的差异诱导误操作，546 个任务上静态与动态误导率分别为 77.4% / 67.2%。
- [Moirae: A Multimodal Agent Collaborative Framework for Dynamic Android Malware Detection](https://arxiv.org/abs/2608.27994) - 2026-08，多模态 Agent 协同做 Android 恶意样本动态检测，用多源证据缓解混淆与概念漂移。
- [CHISEL-ing Back Source Code with AI-enabled Iterative Recovery](https://arxiv.org/abs/2608.27981) - 2026-08，不依赖官方测试套件的 LLM 迭代式源码恢复，在 Ghidra 输出上逐轮修复并用行为验证替代测试用例。
- [Decoupling is a Necessity: Transformation-Agnostic Decompiled Code Recovery (ReSource)](https://arxiv.org/abs/2608.27889) - 2026-08，把编译优化与混淆造成的偏差拆成词法、语法、语义三层分别修复，8 万余对反编译-源码函数上 Top-5 源码检索准确率 83%。
- [Towards LLM-Enhanced Android Taint Analysis](https://arxiv.org/abs/2608.24269) - 2026-08，Agent 驱动 JADX-MCP / Ghidra-MCP 做 Android 污点流推理，DroidBench F1 0.96 vs FlowDroid 0.55。
- [When Binaries Talk Back: Representation-Confusion Attacks on LLM-Assisted Reverse Engineering](https://arxiv.org/abs/2607.12507) - 2026-07，针对 AI 逆向管线的对抗研究：样本能让数据冒充指令、让单源观察冒充独立证据，提示 MCP 工具链需要来源与权限边界。
- [REFORGE: Benchmarking LLMs' Reverse Engineering Capabilities in Decompiled Binary Function Naming](https://arxiv.org/abs/2607.07738) - 2026-07，函数命名基准的构建方法，重点解决函数级真值「有多少真的可评估」这一常被忽略的前提。
- [Multi-View Decompilation for LLM-Based Malware Classification](https://arxiv.org/abs/2606.20436) - 2026-06，融合多个反编译器的输出视图做恶意代码分类，说明单一反编译视图不足以支撑判断。
- [LLM Agent-Assisted Reverse Engineering with Quantitative Readability Metrics](https://arxiv.org/abs/2606.06838) - 2026-06，用可读性量化指标引导 Agent 改写反编译 C 代码，记录从 Ghidra MCP 驱动的三阶段演进与失败模式。
- [Evaluating LLMs for Obfuscation Detection and Classification in Android Apps](https://arxiv.org/abs/2606.14233) - 2026-06，LLM 判断 Android 混淆与混淆类型的实证研究。
- [Can LLMs Deobfuscate Binary Code?](https://arxiv.org/abs/2604.08083) - 2026-04，BinDeObfBench 二进制反混淆基准。
- [Constraint-Guided Multi-Agent Decompilation for Executable Binary Recovery](https://arxiv.org/abs/2604.23940) - 2026-04，多 Agent 修复反编译代码至可编译/可执行。
- [PriviSense: A Frida-Based Framework for Multi-Sensor Spoofing on Android](https://arxiv.org/abs/2601.22414) - 2026-01，基于 Frida 的 Android 传感器和系统值动态 spoofing。
- [Evaluating Large Language Models in detecting Secrets in Android Apps](https://arxiv.org/abs/2510.18601) - 2025-10，SecretLoc，LLM 辅助 Android 硬编码密钥发现。
- [Breaking Android with AI: A Deep Dive into LLM-Powered Exploitation](https://arxiv.org/abs/2509.07933) - 2025-09，Android 渗透测试自动化与 LLM exploit 辅助研究。
- [Deconstructing Obfuscation](https://arxiv.org/abs/2505.19887) - 2025-05，LLM 反汇编代码解混淆能力评测框架。

### 实战文章与工具评测

- [AI APK Reverse Engineering: What an Agent Automates, Where Frida and Manual Work Stay](https://www.xhack.io/blog/ai-apk-reverse-engineering) - 2026-11，梳理 jadx-ai-mcp / jadx-mcp-server 的实际链路与默认端口，列出哪些环节可交给 Agent、哪些必须人工复核，并附截至 2026-11 的工具版本表。
- [AI 加持 Android 逆向：脱壳、Hook、SO 分析与漏洞挖掘实战](http://mp.weixin.qq.com/s?__biz=MzIxNDcwOTcwOQ==&mid=2247504379&idx=1&sn=a7baf017d678f271c38d9ba0d21c4453) - 2026，把逆向拆成信息收集、静态分析、脱壳、动态分析、Native 还原、漏洞挖掘六阶段，逐段说明 AI 能加速什么、哪些结论必须人工验证。
- [Frida 学习笔记（二十六）：DEX 脱壳实战](http://mp.weixin.qq.com/s?__biz=Mzk0MjQxNTI5MA==&mid=2247486588&idx=1&sn=4e63d2cd7cd9991f459180135d1cc6ca) - 2026，讲清一代/二代/三代壳与 Dex2C 的差别、主流加固厂商的 SO 特征表，并对 frida-dexdump、frida_dump、FART 做源码级对比（含 2026-07 的维护状态）。
- [Benchmarking Android APK Deobfuscation Using LLMs](https://fuzzinglabs.com/llm-assisted-android-deobfuscation-benchmark/) - FuzzingLabs 对 Androidmeda、Claude、LLaMA、DeepSeek、StarCoder2 等模型在 APK 解混淆/漏洞发现上的实测。
- [Deobfuscating Android Apps with Androidmeda: A Smarter Way to Read Obfuscated Code](https://www.mobile-hacker.com/2025/07/22/deobfuscating-android-apps-with-androidmeda-a-smarter-way-to-read-obfuscated-code/) - 2025-07，Androidmeda 实测：Crocodilus 恶意样本解混淆、变量名与控制流恢复、vuln_report 输出，并明确提醒人工复核误报。
- [某大厂加密的 AI 全流程分析和逆向](https://www.52pojie.cn/thread-2099021-1-1.html) - 国内 AI 辅助逆向实战笔记。
- [过 1000 Star！跟 AI 说句话就能逆向 Android 应用？](https://www.h3blog.com/article/815/) - Android AI 逆向工具介绍文章。
- [用 ChatGPT 辅助编写 smali 代码逆向修改 DEX 文件](https://blog.csdn.net/mongodb5scout/article/details/154944018) - ChatGPT + smali 修改实践。
- [AI 驱动漏洞挖掘：利用智能体发现 57 个安卓 APP 未知漏洞](https://www.secrss.com/articles/82848) - AI Agent 辅助 Android App 漏洞挖掘案例。

### 官方与方法论资料

- [OWASP MASVS](https://mas.owasp.org/MASVS/) - 移动应用安全验证标准。
- [OWASP MASTG](https://mas.owasp.org/MASTG/) - 移动应用安全测试指南，适合和 JADX、Frida、MobSF 工作流配合。
- [Android Developers - Network Security Configuration](https://developer.android.com/privacy-and-security/security-config) - 抓包、证书信任和网络安全配置分析基础。
- [Frida Docs](https://frida.re/docs/home/) - Frida 官方文档。
- [JADX Wiki](https://github.com/skylot/jadx/wiki) - JADX 使用、插件、调试和常见问题。
- [Apktool Docs](https://apktool.org/docs/install) - Apktool 3.x 安装、解包、构建与调试文档入口。
- [OWASP MASTG 仓库](https://github.com/OWASP/mastg) - MASTG 指南的源文件与测试用例仓库，含各平台测试项的原始描述和脚本，比网页版更方便检索与引用。

## 推荐工作流

### 1. 静态初筛

1. 使用 `jadx` / `jadx-gui` 反编译 APK、XAPK、AAB。
2. 用 `MobSF` 做 Manifest、权限、组件导出、硬编码 URL/secret、第三方 SDK 和基础风险扫描。
3. 对混淆较重的目标使用 `android-reverse-engineering-skill`、`Androidmeda` 或 `jadx-ai-mcp` 辅助恢复语义、定位入口和梳理调用链。
4. 需要多反编译器交叉验证时用 `Bytecode Viewer` 或 `APKLab`；脱壳后的 smali 可先过一遍 `Simplify` 再做人工阅读。

### 2. 动态验证

1. 使用 `Frida` / `objection` 附加目标进程，确认 Java 层和 Native 层关键函数。
2. 抓包优先尝试 `apk-mitm` 或 `android-unpinner`，复杂目标使用 Frida unpinning 脚本动态绕过；需要看 TLS 明文时用 `friTap` 在库层截获。
3. 对登录、签名、风控、加密参数，结合 Frida trace、日志和 `AlgoKiller` 做数据流追踪。

### 3. Native / Flutter 目标

1. JNI/so 优先用 `Ghidra`、`IDA Pro`、`Rizin` 建立函数视图。
2. 可独立调用的 so 使用 `unidbg` 或 `Qiling` 复现环境。
3. Flutter App 使用 `reFlutter`、`Blutter`、`Doldrums` 分析 snapshot、流量代理与 Dart 逻辑；只卡在 pinning 时先试 `kill_flutter`。

### 4. AI 辅助建议

1. 先让模型总结结构，不要直接要求“还原全部逻辑”。
2. 对模型输出的函数名、漏洞结论、密钥类型必须回到代码、trace 或运行结果验证。
3. MCP Server 建议默认只绑定 `127.0.0.1`，不要把 JADX/Frida/IDA/Ghidra 的分析接口暴露到不可信网络。
4. 拒答频繁时先走厂商的信任访问通道，或换到可本地部署的开源权重（见上文「前置」一节），不要把破甲提示词当生产链路的一部分。

## 收录标准

- 优先收录仍在维护、能直接服务 Android 逆向/移动安全/恶意样本分析的工具。
- AI 工具优先收录能接入真实反编译器、动态插桩、trace、MCP 或可复现实验流程的项目。
- 文章和论文优先收录 2025 年以后、含实测数据、工具链细节或可复现方法的资料。
- 不收录明显面向未授权攻击、批量滥用、盗版破解变现的项目。
- 破甲 / 越狱类只收录可核查的机制拆解与审计资料，不收录提示词本体、代理转发与分发镜像。

## 贡献

欢迎提交 PR 补充工具、文章、论文、用法和失效链接。建议格式：

```markdown
- [项目名](https://example.com) - 一句话说明它解决什么问题，最好补充适用场景。
```
