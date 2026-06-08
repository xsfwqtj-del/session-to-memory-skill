# 数据格式规范

## 步骤 ❷ 统一中间格式

无论源 JSONL 来自 CC/Codex/Hermes/WorkBuddy，去噪后统一为：

```json
{
  "exchangeId": 3,
  "role": "user",
  "content": "检查项目现有配置，看看有没有遗漏的地方",
  "timestamp": "2026-06-07T14:38:00.000Z",
  "cwd": "d:\\我的项目"
}
```

## 步骤 ❷ operationsSummary

脚本在去噪同时统计工具调用，供步骤 ❹ 写入"操作环境"段：

```json
{
  "sessionId": "deaddc90",
  "skillsInvoked": ["mio-sbagent-r-web-plan", "skill-creator"],
  "mcpUsed": [],
  "externalCommands": [
    {"command": "npx skills find \"session\"", "purpose": "搜索经验转化引擎"},
    {"command": "powershell New-Item -ItemType Junction ...", "purpose": "建 Junction"}
  ],
  "toolCallStats": {"Read": 38, "Bash": 42, "Write": 16, "WebSearch": 6},
  "filesCreated": ["知识/DeepSeek前缀缓存机制.md", "常看/操作备忘.md"],
  "filesModified": [".claude/skills/Mio-SBAgnet—session-to-memory/SKILL.md"],
  "writeContents": [
    {
      "path": "知识/DeepSeek前缀缓存机制.md",
      "timestamp": "2026-06-07T14:38:00.000Z",
      "charLength": 3200,
      "snippet": "# DeepSeek 前缀缓存...",
      "snapshotFile": "_write-snapshots/知识/DeepSeek前缀缓存机制.md"
    }
  ]
}
```

## 步骤 ❸ 主题列表

```json
[
  {
    "topicId": "topic-deaddc90-001",
    "summary": "全链路审计发现Harness缺失",
    "uniqueKeywords": ["Harness", "hook", "settings.json", "执行层"],
    "categoryHint": ["问题", "决策"],
    "exchangeRange": {"first": 1, "last": 28},
    "precedingTopicIds": [],
    "primaryWriteOps": [],
    "sharedWriteOps": []
  }
]
```

## 步骤 ❹ 生成 .md

每个主题一步完成聚拢→提取→分类→打标→生成 .md。完整模板见 `references/02-file-format.md`。末尾必含"操作环境"段。

**操作环境段字段来源**：
- `会话`/`来源文件`/`时间`/`技能`/`工具`/`外部命令` → 步骤 ❷ operationsSummary（脚本统计产出）
- `新发现` → 步骤 ❹ LLM 判断产出（本主题中发现的 CC 特性/外部知识/bug，LLM 识别后填入）
- `前置主题` → 步骤 ❸ 主题列表

## 步骤 ❺ 去重融合

全部主题的 .md 生成完后统一执行。不产生中间暂存文件——LLM 在内存中比对即可。

## 步骤 ❻ _aggregate 汇聚

从所有新 .md 的"操作环境"段汇聚，更新 `_aggregate/stats-YYYY-MM-DD.json`：

```json
{
  "generatedAt": "2026-06-07 16:00",
  "sessionId": "deaddc90",
  "toolUsage": {"Read": 38, "Bash": 42, "Write": 16, "WebSearch": 6},
  "skillsUsed": ["mio-sbagent-r-web-plan", "skill-creator"],
  "externalCommands": ["npx skills find(3)", "npx skills add(1)", "powershell New-Item(1)"],
  "hotPathsRead": ["项目/.claude/settings.local.json"],
  "hotPathsWritten": ["项目/.claude/skills/*"],
  "newDiscoveries": ["CC changelog路径", "skill安装后需重启"],
  "errorsEncountered": ["cmd.exe /c在WSL中失败", "find-skills丢失"],
  "topicsExtracted": 7,
  "memoriesWritten": 17,
  "causalChain": [
    "topic-001 → topic-002 → topic-003",
    "topic-001 → topic-004 → topic-005 → topic-006",
    "topic-004 → topic-007"
  ]
}
```
