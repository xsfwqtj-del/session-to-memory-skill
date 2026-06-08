# 示例输出

以下展示一条完整记忆文件的内容。转化引擎的步骤 ❹-❻ 应产出此格式。

---

```markdown
---
分类: 问题
项目: 示例项目
时间: 2026-06-07 14:38
主题: topic-deaddc90-001
前置主题: []
关键词: [Harness, hook, settings.json, SessionStart, PreToolUse, 配置审计]
关联:
  - [[问题/02_依赖地图过时]]
  - [[决策/01_修复优先级]]
---

# Harness 层 hook 全部缺失

## 原文
> "[逐字引用，来源: session.jsonl:45-48]"

## 起因
用户要求检查项目现有配置，确认是否有遗漏。触发全链路审计。

## 过程
1. 全项目扫描，覆盖目录树、hook 配置、技能、规则
2. 逐条对比架构文档与实际配置
3. 发现 settings.local.json 仅含权限白名单，无任何 hooks 块

## 结果
架构文档规划的 4 个 hook（SessionStart / PreToolUse / PostToolUse / Stop）全部未配置。

## 因果
触发了后续诊断：规则正文审查、配置链路分析、修复计划制定。

## 操作环境
- 会话: deaddc90
- 来源文件: session.jsonl
- 时间: 14:38 - 15:00
- 技能: [无]
- 工具: [Read(12), Write(2), WebSearch(3)]
- 外部命令: [无]
- 新发现: [Harness 层 hook 需同时在 settings.json 和系统架构中配置]
- 前置主题: []
```
