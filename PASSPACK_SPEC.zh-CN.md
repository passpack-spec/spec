# PassPack 规范 — 开放式语言学习卡片格式

**版本**: 1.0  
**状态**: 草案  
**日期**: 2026-02-19  
**完整英文版**: [PASSPACK_SPEC.md](./PASSPACK_SPEC.md)

---

## 1. PassPack 是什么？

PassPack 是一个**开放的语言学习卡片标准**。它定义了卡片、媒体、AI 分析、学习进度的存储和交换格式。

### 为什么需要 PassPack？

语言学习领域没有富媒体卡片的开放标准：
- **Anki** — 最大的开源卡片平台，但 .apkg 格式**没有官方规范**（全靠社区逆向工程）
- **Quizlet** — 完全封闭
- **Mochi** — 私有格式
- **CSV** — 纯文本，两列，无结构

同时 AI 正在大规模生成结构化的语言解析——但每个 App 的存储格式都不一样。

**PassPack 解决的问题：让卡片像文本文件一样自由——任何人创建、任何工具打开、永远属于学习者自己。**

### 设计原则

| 原则 | 说明 |
|------|------|
| **内容优先** | 卡片承载真实内容和真实媒体 |
| **可移植** | 不锁定厂商，任何 App 都能实现 |
| **分层** | 基础卡片极简；分析、媒体、进度是可选层 |
| **可扩展** | 新分析类型、新字段不会破坏已有读取器 |
| **数据归用户** | 学习进度和笔记属于用户，不属于格式 |

---

## 2. 卡片结构

一张卡片是一个 JSON 对象。**只有三个字段是必填的**，其余全部可选。

### 必填

| 字段 | 说明 | 举例 |
|------|------|------|
| `uuid` | 全球唯一 ID | `a1b2c3d4-e5f6-...` |
| `schemaVersion` | 格式版本 | `passpack-v1` |
| `text` | 原文内容（一个词、一句话、一段对话都行） | `I'm gonna grab a bite.` |

### 可选

| 字段 | 说明 | 举例 |
|------|------|------|
| `cardType` | 卡片类型 | `sentence` / `vocabulary` / `cloze` / `free` |
| `sourceLang` | 学的是什么语言 | `en` |
| `targetLang` | 用什么语言学 | `zh-CN` |
| `source` | 来源（自由文本） | `The Middle S01E03` |
| `media` | 关联媒体 | 见 §3 |
| `analysis` | AI 分析层 | 见 §4 |
| `tags` | 标签 | `["daily_life", "greeting"]` |
| `deck` | 所属甲板，`/` 分层 | `PathEnglish-900/Unit 1` |
| `notes` | 用户笔记 | |
| `origin` | 谁做的 | `official` / `community` / `import` / `manual` |
| `difficulty` | CEFR 难度 | `A1` ~ `C2` |
| `progress` | 学习进度（迁移用） | 见 §5 |
| `createdAt` | 创建时间 | |
| `updatedAt` | 更新时间 | |

### 最简卡片

```json
{
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "schemaVersion": "passpack-v1",
  "text": "I'm gonna grab a bite."
}
```

这就是一张合法的 PassPack 卡片。

---

## 3. 媒体

| 字段 | 说明 |
|------|------|
| `visual` | 视频 (.mp4) 或图片 (.jpg) 的文件路径 |
| `audio` | 独立音频文件 (.m4a) 的路径 |

全部可选。纯背单词的卡片可以没有任何媒体。

如果只有视频没有独立音频，App 可以直接从 mp4 里提取音频轨道使用。

---

## 4. AI 分析层（核心创新）

卡片是**事实**（这句话 + 这段视频），分析是**解读**（AI 或人对这句话的结构化解析）。

### 为什么分开？

- 同一张卡片可以有多份分析（中文版、日文版、初级版、高级版）
- 分析可以单独更新（AI 升级了，重新生成，卡片本身不动）
- App 按需加载（免费版只显示基础，付费版加载高级）
- 第三方可以贡献分析

### 结构

`analysis` 是一个数组，每个元素：

| 字段 | 必填 | 说明 |
|------|------|------|
| `type` | ✅ | 分析类型 |
| `version` | ✅ | 版本 |
| `targetLang` | ❌ | 这份分析的目标语言 |
| `generatedBy` | ❌ | `ai` / `human` / `ai+human` |
| `data` | ✅ | 具体内容（由 type 决定） |

### 首批官方分析类型

#### `logicBlocks` — 逻辑意群解析

```json
{
  "type": "logicBlocks",
  "version": "1.0",
  "targetLang": "zh-CN",
  "generatedBy": "ai+human",
  "data": {
    "blocks": [
      { "phrase": "I'm gonna", "meaning": "我将要（口语）" },
      { "phrase": "grab a bite", "meaning": "吃点东西（非正式）" }
    ],
    "vibeTranslation": "我去吃点东西。"
  }
}
```

#### `usageGuide` — 使用指南

formality、bestFor、avoidWith、grammar、commonMistakes、culturalNote 等字段，App 按需渲染。

#### `definition` — 词义（单词卡用）

pronunciation、partOfSpeech、definitions 等字段。

**社区可以注册新的分析类型**，用 `x-` 前缀避免冲突。App 遇到不认识的类型直接跳过。

---

## 5. 学习进度

进度是**可选的**，用于 App 间迁移。不是卡片内容的一部分。

### 为什么不直接存算法状态？

因为各家算法完全不同：Anki 用 SM-2，Mochi 用 FSRS，SuperMemo 用 SM-18。PassPack 只存**算法无关的事实**。

### 三层设计（全部可选）

#### 第一层：掌握等级

`new` → `learning` → `familiar` → `known` → `mastered`

人类可读的快照。新 App 至少知道"这些卡你已经学过了"。

#### 第二层：记忆保持率

```json
{ "probability": 0.85, "estimatedAt": "2026-02-19T10:00:00Z" }
```

此刻你有多大概率记得这张卡？0 到 1 之间。任何 SRS 算法都能用这个数作为起点。

#### 第三层：复习日志

```json
[
  { "date": "2026-01-15", "rating": 3 },
  { "date": "2026-01-22", "rating": 4 }
]
```

评分 1-4：1=完全忘了，2=想了很久，3=想了一下，4=秒回忆。这是 Anki/FSRS 的事实标准。

新 App 拿到完整日志，可以用自己的算法重跑一遍，几乎完美还原。

---

## 6. 卡片包（.passpack）

ZIP 压缩包：

```
unit_01.passpack
├── manifest.json
└── media/
    ├── a1b2c3d4.mp4
    └── ...
```

### manifest.json

| 必填字段 | 说明 |
|----------|------|
| `schemaVersion` | `passpack-v1` |
| `cardCount` | 卡片数量 |
| `cards` | 卡片数组 |

| 可选字段 | 说明 |
|----------|------|
| `title` | 包名称 |
| `description` | 描述 |
| `author` | 作者 |
| `license` | 许可证（推荐 CC BY-NC-SA 4.0） |
| `sourceLang` | 源语言（卡片级可覆盖） |
| `targetLang` | 目标语言 |
| `generator` | 生成工具 |
| `generatedAt` | 生成时间 |

---

## 7. 导入规则

| 情况 | App 必须怎么做 |
|------|---------------|
| UUID 没见过 | 新增（含 notes 一起导入） |
| UUID 已存在 | 更新内容，**不许动用户的进度和笔记** |
| 已有卡片的 notes 冲突 | 保留本地 notes；导入的 notes 另存，让用户手动合并 |

这条是**强制的**。用户的学习进度和笔记是神圣不可侵犯的。

---

## 8. 扩展规则

| 规则 | 说明 |
|------|------|
| 自定义字段 | `x_` 开头，其他 App 忽略 |
| 自定义分析类型 | `x-` 开头 |
| 不认识的字段 | 忽略，不报错（像浏览器对待未知 HTML 标签一样） |
| 不认识的版本号 | 报错，提示用户升级 |

**宽容读取，严格写入。**

---

## 9. 许可证

| 层 | 许可证 | 范围 |
|----|--------|------|
| **规范文档** | CC BY-SA 4.0 | 任何人都能实现 |
| **卡片内容** | 作者自选（推荐 CC BY-NC-SA 4.0） | 在 manifest.license 里设 |

卡片内容推荐使用 CC BY-NC-SA 4.0（允许个人使用和分享，禁止商业转售）。

---

*技术细节（JSON Schema、MIME type、命名约定、完整示例等）请参考[英文完整版](./PASSPACK_SPEC.md)。*

*本规范由 PassPack 项目维护。*  
*GitHub: https://github.com/passpack-spec*
