# Awesome AI Android Reverse

> Android 逆向、移动安全分析与 AI 辅助逆向工具/文章收集。
>
> 最后更新：2026-07-11

本仓库只做公开资料索引，面向授权安全测试、恶意样本分析、漏洞研究、CTF 和学习用途。请勿用于未授权逆向、绕过商业授权、侵犯隐私或其他违法场景。

## 目录

- [AI / MCP 逆向工具](#ai--mcp-逆向工具)
- [AI Skills、Prompts 与 Agent 工作流](#ai-skillsprompts-与-agent-工作流)
- [LLM 解混淆、漏洞发现与算法还原](#llm-解混淆漏洞发现与算法还原)
- [基础 Android 逆向工具链](#基础-android-逆向工具链)
- [动态分析、Hook 与抓包辅助](#动态分析hook-与抓包辅助)
- [Flutter / Native / 二进制分析](#flutter--native--二进制分析)
- [文章、论文与教程](#文章论文与教程)
- [推荐工作流](#推荐工作流)
- [收录标准](#收录标准)

## AI / MCP 逆向工具

- [jadx-ai-mcp](https://github.com/zinja-coder/jadx-ai-mcp) - JADX GUI 插件，把当前类、Manifest、资源、xref、调试信息等上下文暴露给支持 MCP 的 LLM 客户端。
- [jadx-mcp-server](https://github.com/zinja-coder/jadx-mcp-server) - 独立 Python MCP Server，连接 JADX-AI-MCP 插件，让模型读取反编译代码、搜索类/方法/字段、辅助审计 APK。
- [apktool-mcp-server](https://github.com/zinja-coder/apktool-mcp-server) - 基于 Apktool 的 MCP Server，支持解包、Manifest/resource/smali 读取与修改、重打包等 APK 修改流程。
- [GhidraMCP](https://github.com/LaurieWired/GhidraMCP) - Ghidra MCP Server，把反编译、函数、符号与程序结构上下文提供给 LLM。
- [ida-pro-mcp](https://github.com/mrexodia/ida-pro-mcp) - IDA Pro MCP Server，支持 headless / GUI 流程，面向 IDA 内的变量重命名、类型调整、注释、报告生成等 vibe reversing 场景。
- [IDA-NO-MCP](https://github.com/P4nda0s/IDA-NO-MCP) - 针对 IDA Pro 的轻量 AI 交互方案，目标是降低传统 MCP 交互的复杂度与延迟。
- [frida-mcp](https://github.com/zhizhuodemao/frida-mcp) - Frida 动态分析 MCP Server，支持应用启动/附加、前台应用识别、应用列表、脚本注入和自定义 frida-server 配置。
- [ZIN-MCP-Client](https://github.com/zinja-coder/zin-mcp-client) - 面向本地 LLM / Ollama 的命令行 MCP Client，可连接 JADX、Apktool 等逆向 MCP Server。

## AI Skills、Prompts 与 Agent 工作流

- [android-reverse-engineering-skill](https://github.com/SimoneAvogadro/android-reverse-engineering-skill) - Claude Code Android 逆向 Skill，覆盖 APK/XAPK/JAR/AAR 反编译、API 提取、Kotlin/R8 名称恢复、Ktor/Apollo/Koin/HMAC 识别等流程。
- [ai-reverse-toolkit](https://github.com/zhizhuodemao/ai-reverse-toolkit) - 面向逆向分析的 skills、rules 和 prompts 集合，包含加密入口定位、补环境、AST 解混淆等任务模板。
- [mcp-reversing-dataset](https://github.com/mrexodia/mcp-reversing-dataset) - IDA Pro MCP 示例数据与演示素材，可用于验证 LLM 逆向流程和提示词。
- [Code2MCP / MCP-Github-Agent](https://github.com/DEFENSE-SEU/MCP-Github-Agent) - 将 GitHub 代码仓库自动转换为 MCP 服务的多 Agent 框架，可参考其 MCP 化思路扩展逆向工具。

## LLM 解混淆、漏洞发现与算法还原

- [Androidmeda](https://github.com/In3tinct/Androidmeda) - LLM 驱动的 Android 代码解混淆与漏洞扫描工具，支持 OpenAI、Gemini、Anthropic API 和 Ollama 本地模型。
- [AlgoKiller](https://github.com/lidongyooo/AlgoKiller) - 面向 ARM64 执行 trace 的算法还原 harness，驱动 LLM 从 trace 中追踪数据流、恢复加密/签名/编码逻辑，并输出 Python 复现代码或结构化报告。
- [SecretLoc](https://arxiv.org/abs/2510.18601) - LLM 检测 Android App 硬编码 secrets 的研究，强调上下文和结构线索对未知密钥类型的发现能力。
- [Evaluating LLMs for Obfuscation Detection and Classification in Android Apps](https://arxiv.org/abs/2606.14233) - 2026 年 Android 混淆检测论文，评估 LLM 在语义层面识别混淆代码的能力。
- [Can LLMs Deobfuscate Binary Code?](https://arxiv.org/abs/2604.08083) - BinDeObfBench，系统评测 LLM 对多阶段二进制混淆的反混淆能力。
- [Constraint-Guided Multi-Agent Decompilation](https://arxiv.org/abs/2604.23940) - 多 Agent 约束引导反编译框架，使用语法、编译和行为约束迭代修复反编译输出。

## 基础 Android 逆向工具链

- [jadx](https://github.com/skylot/jadx) - Dex/APK/AAB/XAPK 到 Java 的反编译器，带 GUI、资源解码、搜索、xref、deobfuscator 和 smali debugger。
- [Apktool](https://github.com/iBotPeaches/Apktool) - Android APK 资源解码、smali 修改、重打包和调试的核心工具。
- [Frida](https://github.com/frida/frida) - 动态插桩框架，Android Java 层和 Native 层 Hook 的事实标准之一。
- [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) - 自动化移动应用安全分析平台，支持 Android/iOS 静态分析、动态分析、恶意样本分析、隐私分析和 CI/CD 集成。
- [Ghidra](https://github.com/NationalSecurityAgency/ghidra) - 开源二进制逆向平台，适合 Android Native so、JNI、算法还原与漏洞分析。
- [IDA Pro](https://hex-rays.com/ida-pro/) - 商业二进制分析和反编译平台，Android Native、固件、复杂算法逆向常用。
- [JEB Decompiler](https://www.pnfsoftware.com/) - 商业 Android/Dalvik/Native 反编译器，JEB 5.x 已加入 VIBRE AI assistant 和 MCP Server 相关能力。
- [Rizin](https://github.com/rizinorg/rizin) / [radare2](https://github.com/radareorg/radare2) - 开源二进制分析框架，适合脚本化、CLI 批处理和 Native 分析。

## 动态分析、Hook 与抓包辅助

- [objection](https://github.com/sensepost/objection) - 基于 Frida 的移动运行时探索工具，支持 Android/iOS，常用于 SSL pinning 绕过、文件系统查看、内存 patch、对象探索等。
- [android-unpinner](https://github.com/mitmproxy/android-unpinner) - mitmproxy 出品的 APK 证书绑定移除工具，不要求 root，使用 Frida Gadget/JDWP 注入思路，支持 XAPK。
- [apk-mitm](https://github.com/niklashigi/apk-mitm) - 自动修改 APK 以便 HTTPS 抓包，处理 Network Security Configuration、常见证书绑定逻辑、重打包和签名。
- [HTTP Toolkit Frida Android Unpinning](https://github.com/httptoolkit/frida-android-unpinning) - 常用 Frida SSL pinning 绕过脚本集合。
- [r0capture](https://github.com/r0ysue/r0capture) - Android 应用层抓包辅助脚本，常用于快速观察明文请求、响应和加密前后数据。
- [PriviSense](https://arxiv.org/abs/2601.22414) - Frida-based Android 多传感器/系统信号 spoofing 框架研究，可用于上下文敏感 App 行为复现实验。

## Flutter / Native / 二进制分析

- [reFlutter](https://github.com/Impact-I/reFlutter) - Flutter 逆向框架，支持 Android/iOS Flutter App 的流量代理、dump.dart 提取和部分 pinning 绕过。
- [Blutter](https://github.com/worawit/blutter) - Flutter Dart AOT 反汇编/符号辅助工具，适合 Flutter Native 层分析。
- [Doldrums](https://github.com/rscloura/Doldrums) - Flutter snapshot 分析与 Dart 结构恢复工具。
- [SoFixer](https://github.com/F8LEFT/SoFixer) - Android ELF/so dump 修复工具，适合脱壳后 so 修复。
- [unidbg](https://github.com/zhkl0228/unidbg) - Android Native 库模拟执行框架，可用于 JNI 算法调用、签名函数复现和协议分析。
- [Qiling](https://github.com/qilingframework/qiling) - 多架构二进制模拟框架，可辅助 Android Native 算法环境搭建。

## 文章、论文与教程

### 2026 / 2025 研究论文

- [Evaluating LLMs for Obfuscation Detection and Classification in Android Apps](https://arxiv.org/abs/2606.14233) - 2026-06，LLM 判断 Android 混淆与混淆类型的实证研究。
- [Can LLMs Deobfuscate Binary Code?](https://arxiv.org/abs/2604.08083) - 2026-04，BinDeObfBench 二进制反混淆基准。
- [Constraint-Guided Multi-Agent Decompilation for Executable Binary Recovery](https://arxiv.org/abs/2604.23940) - 2026-04，多 Agent 修复反编译代码至可编译/可执行。
- [PriviSense: A Frida-Based Framework for Multi-Sensor Spoofing on Android](https://arxiv.org/abs/2601.22414) - 2026-01，基于 Frida 的 Android 传感器和系统值动态 spoofing。
- [Evaluating Large Language Models in detecting Secrets in Android Apps](https://arxiv.org/abs/2510.18601) - 2025-10，SecretLoc，LLM 辅助 Android 硬编码密钥发现。
- [Breaking Android with AI: A Deep Dive into LLM-Powered Exploitation](https://arxiv.org/abs/2509.07933) - 2025-09，Android 渗透测试自动化与 LLM exploit 辅助研究。
- [Deconstructing Obfuscation](https://arxiv.org/abs/2505.19887) - 2025-05，LLM 反汇编代码解混淆能力评测框架。

### 实战文章与工具评测

- [Benchmarking Android APK Deobfuscation Using LLMs](https://fuzzinglabs.com/llm-assisted-android-deobfuscation-benchmark/) - FuzzingLabs 对 Androidmeda、Claude、LLaMA、DeepSeek、StarCoder2 等模型在 APK 解混淆/漏洞发现上的实测。
- [Deobfuscating Android malware](https://www.mobile-hacker.com/) - Androidmeda README 引用的 Android 恶意样本解混淆文章入口。
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
- [Apktool Docs](https://apktool.org/docs/) - Apktool 安装、解包、构建与调试文档。

## 推荐工作流

### 1. 静态初筛

1. 使用 `jadx` / `jadx-gui` 反编译 APK、XAPK、AAB。
2. 用 `MobSF` 做 Manifest、权限、组件导出、硬编码 URL/secret、第三方 SDK 和基础风险扫描。
3. 对混淆较重的目标使用 `android-reverse-engineering-skill`、`Androidmeda` 或 `jadx-ai-mcp` 辅助恢复语义、定位入口和梳理调用链。

### 2. 动态验证

1. 使用 `Frida` / `objection` 附加目标进程，确认 Java 层和 Native 层关键函数。
2. 抓包优先尝试 `apk-mitm` 或 `android-unpinner`，复杂目标使用 Frida unpinning 脚本动态绕过。
3. 对登录、签名、风控、加密参数，结合 Frida trace、日志和 `AlgoKiller` 做数据流追踪。

### 3. Native / Flutter 目标

1. JNI/so 优先用 `Ghidra`、`IDA Pro`、`Rizin` 建立函数视图。
2. 可独立调用的 so 使用 `unidbg` 或 `Qiling` 复现环境。
3. Flutter App 使用 `reFlutter`、`Blutter`、`Doldrums` 分析 snapshot、流量代理与 Dart 逻辑。

### 4. AI 辅助建议

1. 先让模型总结结构，不要直接要求“还原全部逻辑”。
2. 对模型输出的函数名、漏洞结论、密钥类型必须回到代码、trace 或运行结果验证。
3. MCP Server 建议默认只绑定 `127.0.0.1`，不要把 JADX/Frida/IDA/Ghidra 的分析接口暴露到不可信网络。

## 收录标准

- 优先收录仍在维护、能直接服务 Android 逆向/移动安全/恶意样本分析的工具。
- AI 工具优先收录能接入真实反编译器、动态插桩、trace、MCP 或可复现实验流程的项目。
- 文章和论文优先收录 2025 年以后、含实测数据、工具链细节或可复现方法的资料。
- 不收录明显面向未授权攻击、批量滥用、盗版破解变现的项目。

## 贡献

欢迎提交 PR 补充工具、文章、论文、用法和失效链接。建议格式：

```markdown
- [项目名](https://example.com) - 一句话说明它解决什么问题，最好补充适用场景。
```
