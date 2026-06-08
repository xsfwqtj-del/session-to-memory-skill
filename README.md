# Session-to-Memory

> 通用会话→记忆转化引擎 · v2.4.0

将 AI agent 会话 JSONL 文件（CC / Codex / Hermes / WorkBuddy）转化为 **11 分类** 结构化、可搜索的 Markdown 记忆文件。产出的 wiki 可直接用 Obsidian 打开，或喂入 [llm_wiki](https://github.com/nashsu/llm_wiki) 等知识库工具。

## 一句话安装

```bash
npx skills add xsfwqtj-del/session-to-memory-skill
```

## 它能做什么

```
JSONL 会话文件 → 6 步管线 → Mio-SBAgnet—memory/
                               ├── 错误/    ├── 知识/
                               ├── 决策/    ├── 配置/
                               ├── 偏好/    ├── 使用/
                               ├── 模式/    ├── 讨论/
                               ├── 排除/    ├── 问题/
                               └── 不变量/
```

- **6 步管线**：预检 → 源适配+去噪 → 识别主题 → 提取+分类+锚定 → 去重融合 → 写入
- **11 分类记忆**：错误 / 决策 / 偏好 / 模式 / 排除 / 不变量 / 讨论 / 问题 / 知识 / 配置 / 使用
- **原文锚点**：高风险记忆（错误/决策/不变量/偏好）保留原始对话的逐字引用
- **级联去重**：LLM 判断 + Jaccard 相似度 + 低热度专有名词约束，防高频通用词误关联
- **分类门禁**：逐类强制过 11 分类信号，跳过的必须附理由
- **source 追溯**：每条 exchange 标注 `{文件名}:{行号}`，关键结论可溯源到原始 JSONL 行

## 设计原则

- **通用**：不绑定任何特定用户或项目
- **分层**：脚本做机械操作（统计/格式化），LLM 做需要判断的步骤（主题识别/分类/去重）
- **零额外依赖**：所有增强在现有 LLM 调用内完成，不加向量库、不加 API 调用轮次
- **增量安全**：去重是增量安全的，重复跑不会产生重复文件

## 研究基础

- **CogCanvas** (2026.01)：原文锚点 vs 转述——93% vs 19% 完全匹配率
- **ProMem** (2025.01)：自问答验证提升记忆完整性至 73.8%
- **CarMem** (COLING 2025)：类别约束提取 0.78-0.95 F1
- **MAGMA** (ACL 2026 Main)：四图解耦验证了多结构互补的必要性

## 支持的会话源

| Agent | 源识别方式 |
|-------|-----------|
| Claude Code (CC) | `~/.claude/projects/` 下的 JSONL |
| Codex CLI | 字段映射表适配 |
| Hermes | 字段映射表适配 |
| WorkBuddy | 字段映射表适配 |

## 用法

在 CC 中输入 `/Mio-SBAgnet—session-to-memory` 触发，或在任何支持 Markdown 指令的 LLM agent 中打开 `SKILL.md`。

输出的记忆目录可直接作为 [llm_wiki](https://github.com/nashsu/llm_wiki) 的 `raw/sources/` 输入。



