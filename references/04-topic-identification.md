# 主题识别与关键词聚拢

## 目的

一个会话含多个话题。不按时间切分——而是 LLM 遍览全对话，识别所有独立主题，然后用独有关键词回搜聚拢同一主题的分散片段。

**为什么不裁切**: 同一主题可能在对话中隔了 20 轮再次提及。按时间硬切会切断信息。主题识别 + 关键词回搜保证完整性。

## 识别信号

LLM 遍历去噪后的清洁对话流，检测以下主题边界提示（仅作为参考，不强制切分）：

| 信号 | 示例 | 作用 |
|------|------|------|
| 用户显式切换 | "不说这个了""回到XX""换话题" | 强主题边界 |
| 收尾信号 | "搞定""汇总""完成""好了""就这样" | 当前主题结束 |
| 任务指令转向 | 讨论架构 → 突然"查一下XX" | 大概率新主题 |
| 回马枪 | 20 轮后又提到之前的话题 | 同一主题的延续，不是新主题 |

## 步骤

### 1. 遍览全对话，列出主题

```
输入: 清洁对话流（全部 exchange）
输出: [
  {
    topicId: "topic-{sessionId前8位}-NNN",
    summary: "一句话——这个主题在讲什么",
    uniqueKeywords: ["词1", "词2", "词3"],    // 3-5个，该主题独有的关键词
    categoryHint: ["决策", "问题"],           // 预判分类方向
    exchangeRange: {first: 3, last: 145},     // 该主题出现的区间
    precedingTopicId: "topic-xxx-NNN"          // 前置主题ID（因果链）
  }
]
```

### 2. 关键词回搜聚拢

对每个主题，用 `uniqueKeywords` 回搜全对话流。找到所有匹配的 exchange——即使它们被其他话题隔断。聚拢为完整上下文。

```
主题 "Harness审计" 关键词: [Harness, hook, settings.json]
  → 回搜全对话 → 找到 exchange #3-24, #67-72, #89-95
  → 聚拢为完整事件上下文
```

### 3. 去重合并

同会话内主题重叠（关键词重叠 ≥ 50%）→ 合并为一个主题，扩大 uniqueKeywords。

## 示例

```
会话 deaddc90 (2026-06-07 14:38-16:00)

topic-deaddc90-001  审计发现Harness缺失
  → 关键词: [Harness, hook, settings.json, 执行层]
  → 后续触发: topic-002, topic-003, topic-004
  → 前置: null

topic-deaddc90-002  递进展开设计分析
  → 关键词: [递进展开, 法索引, 地图先行, DeepSeek缓存]
  → 前置: topic-001

topic-deaddc90-003  DeepSeek前缀缓存机制研究
  → 关键词: [DeepSeek, 前缀缓存, KV Cache, token优化]
  → 前置: topic-002

topic-deaddc90-004  中转站断裂诊断 + 舍弃决策
  → 关键词: [中转站, JSONL回炉, skill-check, 碎片遗漏]
  → 前置: topic-001

topic-deaddc90-005  Junction D-----→CCHuihua迁移
  → 关键词: [Junction, CCHuihua, 中文路径, 会话存档]
  → 前置: topic-004

topic-deaddc90-006  find-skills丢失+搜索经验转化引擎
  → 关键词: [find-skills, npx skills, clerk, extract-transcripts]
  → 前置: topic-005

topic-deaddc90-007  记忆分类体系讨论
  → 关键词: [11类, 4项目, 关键词网, 裁切, 操作备忘]
  → 前置: topic-004, topic-006
```

## 注意事项

- 主题不是事件边界——主题之间可以有交叉、嵌套、延续
- `uniqueKeywords` 决定聚拢质量。太宽泛（如"修复"）会拉进无关 exchange；太窄（如"settings.local.json第3行"）会漏掉相关上下文
- 同一个 exchange 可以属于多个主题（如用户一句话同时提到 Harness 和 DeepSeek）
