# PassPack Specification — Open Card Format for Language Learning

**Version**: 1.0  
**Status**: Draft  
**Date**: 2026-02-19  
**License**: CC BY-SA 4.0 (this specification)

---

## 1. Introduction

### 1.1 What is PassPack?

PassPack is an open, portable format for language learning flashcards. It defines how cards, media, AI-generated analysis, and learning progress are structured, packaged, and exchanged between applications.

### 1.2 Why PassPack?

The language learning space has no open standard for rich flashcards. Existing formats are either proprietary (Quizlet), undocumented (Anki .apkg has no official spec), or plain-text only (CSV). Meanwhile, AI is generating structured linguistic analysis at scale — but every app stores it differently.

PassPack solves this by defining:
- A **card format** that any app can read and write
- An **extensible analysis layer** for AI-generated content
- A **packaging format** (.passpack) for distribution
- **Import rules** that protect user data

### 1.3 Design Principles

| Principle | Description |
|-----------|-------------|
| **Content-first** | Cards carry real content and real media, not templates |
| **Portable** | No vendor lock-in; any app can implement this spec |
| **Layered** | Basic card is minimal; analysis, media, progress are optional layers |
| **Extensible** | New analysis types, new fields — without breaking existing readers |
| **User-owns-data** | Learning progress and notes belong to the user, not the format |

### 1.4 Terminology

| Term | Definition |
|------|------------|
| **Card** | A single learning unit: text + optional media + optional analysis |
| **Card Pack** | A `.passpack` file containing cards and media for distribution |
| **Analysis** | A structured interpretation of a card's content (AI or human generated) |
| **Deck** | A hierarchical grouping of cards (e.g. `IELTS/Listening/Part 1`) |

---

## 2. Card Object

A Card is a JSON object. Only three fields are required — everything else is optional.

### 2.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `uuid` | string (UUID v4) | Globally unique identifier. Never changes once assigned. |
| `schemaVersion` | string | Always `"passpack-v1"` for this version of the spec. |
| `text` | string | The core content — a word, sentence, paragraph, or dialogue. |

### 2.2 Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `cardType` | string | `"sentence"` | Card type (see §2.3) |
| `sourceLang` | string (BCP 47) | — | Language being learned (e.g. `"en"`, `"ja"`) |
| `targetLang` | string (BCP 47) | — | Learner's native language (e.g. `"zh-CN"`, `"ko"`) |
| `source` | string | — | Where this content comes from (free text) |
| `media` | object | — | Associated media files (see §3) |
| `analysis` | array | — | AI/human analysis layers (see §4) |
| `tags` | string[] | `[]` | Free-form tags for filtering |
| `deck` | string | — | Deck path, `/` separated for hierarchy |
| `notes` | string | `""` | User's personal notes |
| `origin` | string | — | Who created this card (see §2.4) |
| `difficulty` | string | — | CEFR level (see §2.5) |
| `progress` | object | — | Learning progress for migration (see §5) |
| `createdAt` | string (ISO 8601) | — | Creation timestamp (UTC) |
| `updatedAt` | string (ISO 8601) | — | Last update timestamp (UTC) |

### 2.3 Card Types

| Value | Description |
|-------|-------------|
| `sentence` | Sentence, paragraph, or dialogue (default) |
| `vocabulary` | Single word or phrase |
| `cloze` | Fill-in-the-blank (use `{{answer}}` to mark blanks in `text`) |
| `free` | Freeform — any content the user wants |

The `cardType` field is a hint to the rendering app. A card is valid regardless of its type.

### 2.4 Origin Values

| Value | Description |
|-------|-------------|
| `official` | Curated by a content team |
| `community` | User-contributed |
| `import` | Imported from another format |
| `manual` | Manually created by the user |

### 2.5 Difficulty (CEFR)

The Common European Framework of Reference for Languages:

| Level | Description |
|-------|-------------|
| `A1` | Beginner |
| `A2` | Elementary |
| `B1` | Intermediate |
| `B2` | Upper Intermediate |
| `C1` | Advanced |
| `C2` | Proficiency |

### 2.6 Deck Hierarchy

Use `/` to express hierarchy:

```
"deck": "PathEnglish-900/Unit 1"
"deck": "IELTS/Listening/Part 1"
"deck": "My Vocabulary/Daily"
```

Apps SHOULD support at least 3 levels of nesting.

### 2.7 Minimal Card

```json
{
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "schemaVersion": "passpack-v1",
  "text": "I'm gonna grab a bite."
}
```

This is a valid PassPack card. Everything else is optional.

---

## 3. Media

The `media` object references files bundled in the card pack.

### 3.1 Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `visual` | string | ❌ | Path to video (.mp4) or image (.jpg/.png) |
| `audio` | string | ❌ | Path to standalone audio file (.m4a) |

All paths are relative to the `media/` directory in the card pack.

### 3.2 Media Format Requirements

| Type | Format | Codec | Notes |
|------|--------|-------|-------|
| Video | `.mp4` | H.264, ≤720p, AAC audio | Recommended ≤2.5 Mbps |
| Image | `.jpg` or `.png` | — | — |
| Audio | `.m4a` | AAC | — |

### 3.3 Notes

- A card with only `visual` (mp4) and no `audio` is fine — apps can extract audio from the video track.
- A card with no media at all is fine — text-only cards are first-class citizens.
- Media files are optional. A vocabulary card for "apple" doesn't need a video clip.

### 3.4 Example

```json
"media": {
  "visual": "a1b2c3d4.mp4",
  "audio": "a1b2c3d4.m4a"
}
```

---

## 4. Analysis Layer

The analysis layer is PassPack's core innovation. It separates **content** (the card) from **interpretation** (what AI or humans say about it).

### 4.1 Why Separate?

- Same card, multiple analyses: Chinese explanation + Japanese explanation
- Analyses can be updated independently (better AI model → regenerate)
- Apps load what they need: free tier shows basic, paid tier shows advanced
- Third parties can contribute analyses without touching the card

### 4.2 Structure

The `analysis` field is an array. Each element:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | ✅ | Analysis type identifier |
| `version` | string | ✅ | Version of this analysis type |
| `targetLang` | string (BCP 47) | ❌ | Target language of this analysis |
| `generatedBy` | string | ❌ | `"ai"`, `"human"`, or `"ai+human"` |
| `data` | object | ✅ | The analysis content (structure defined by `type`) |

### 4.3 Example

```json
"analysis": [
  {
    "type": "logicBlocks",
    "version": "1.0",
    "targetLang": "zh-CN",
    "generatedBy": "ai+human",
    "data": {
      "blocks": [
        { "phrase": "I'm gonna", "meaning": "我将要（口语，I am going to 的缩写）" },
        { "phrase": "grab a bite", "meaning": "吃点东西（非正式）" }
      ],
      "vibeTranslation": "我去吃点东西。"
    }
  }
]
```

### 4.4 Analysis Type Registry

Each analysis type has its own sub-specification defining the `data` structure. The following are the initial official types:

| Type | Purpose | Sub-spec |
|------|---------|----------|
| `logicBlocks` | Phrase-level breakdown + natural translation | See Appendix A |
| `usageGuide` | Formality, context, grammar, common mistakes | See Appendix A |
| `definition` | Word definition, pronunciation, part of speech | See Appendix A |

Community-defined types are welcome. To avoid collisions, use a namespace prefix:

```
"type": "x-myapp-pronunciation-score"
```

### 4.5 Handling Unknown Types

Apps MUST ignore analysis types they don't recognize. An unknown type is not an error — it's simply a layer the app doesn't support yet.

---

## 5. Learning Progress

Progress data is **optional** and intended for migration between apps. It is NOT part of the card content — it represents one user's learning history with that card.

### 5.1 Design Philosophy

Different apps use different algorithms (SM-2, FSRS, SM-18, proprietary). PassPack stores **algorithm-agnostic facts**, not algorithm-specific state.

### 5.2 Three Layers (all optional)

#### Layer 1: Level (human-readable snapshot)

```json
"level": "known"
```

| Value | Meaning |
|-------|---------|
| `new` | Never studied |
| `learning` | Currently learning, not stable |
| `familiar` | Recognizes it, sometimes forgets |
| `known` | Reliably recalls |
| `mastered` | Deeply embedded, long intervals |

#### Layer 2: Retention (algorithm-usable)

```json
"retention": {
  "probability": 0.85,
  "estimatedAt": "2026-02-19T10:00:00Z"
}
```

The probability (0 to 1) that the user can recall this card at the given timestamp. Any SRS algorithm can use this as a starting point.

#### Layer 3: Review Log (complete history)

```json
"reviewLog": [
  { "date": "2026-01-15", "rating": 3 },
  { "date": "2026-01-22", "rating": 4 },
  { "date": "2026-02-10", "rating": 2 }
]
```

Rating uses the **universal 1-4 scale**:

| Rating | Meaning | Anki equivalent |
|--------|---------|-----------------|
| 1 | Forgot completely | Again |
| 2 | Recalled with difficulty | Hard |
| 3 | Recalled with some effort | Good |
| 4 | Instant recall | Easy |

#### Full Example

```json
"progress": {
  "level": "known",
  "retention": {
    "probability": 0.85,
    "estimatedAt": "2026-02-19T10:00:00Z"
  },
  "reviewLog": [
    { "date": "2026-01-15", "rating": 3 },
    { "date": "2026-01-22", "rating": 4 },
    { "date": "2026-02-10", "rating": 2 }
  ]
}
```

### 5.3 How Apps Use This

| Available data | What the app can do |
|----------------|---------------------|
| Only `level` | Map to internal state (approximate) |
| `level` + `retention` | Use retention + elapsed time to compute initial parameters |
| Full `reviewLog` | Replay history through own algorithm (most accurate) |

---

## 6. Card Pack Format (.passpack)

A `.passpack` file is a ZIP archive.

### 6.1 File Structure

```
unit_01_greetings.passpack
├── manifest.json
└── media/
    ├── a1b2c3d4.mp4
    ├── a1b2c3d4.m4a
    ├── b2c3d4e5.jpg
    └── ...
```

### 6.2 Manifest

The `manifest.json` file describes the pack and contains all cards.

#### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `schemaVersion` | string | `"passpack-v1"` |
| `cardCount` | integer | Number of cards (must equal `cards.length`) |
| `cards` | Card[] | Array of card objects |

#### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Pack name |
| `description` | string | What's in this pack |
| `author` | string | Who made it |
| `license` | string | Content license (recommended: `"CC BY-NC-SA 4.0"`) |
| `sourceLang` | string (BCP 47) | Pack-level source language (cards can override) |
| `targetLang` | string (BCP 47) | Pack-level target language (cards can override) |
| `generator` | string | Tool that created this pack |
| `generatedAt` | string (ISO 8601) | When the pack was created |

#### Example

```json
{
  "schemaVersion": "passpack-v1",
  "title": "Unit 1 — Daily Greetings",
  "description": "15 essential greeting expressions from The Middle",
  "author": "PathEnglish Team",
  "license": "CC BY-NC-SA 4.0",
  "sourceLang": "en",
  "targetLang": "zh-CN",
  "generator": "PathEnglish Studio v1.0",
  "generatedAt": "2026-02-19T10:30:00Z",
  "cardCount": 1,
  "cards": [
    {
      "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "schemaVersion": "passpack-v1",
      "text": "I'm gonna grab a bite.",
      "cardType": "sentence",
      "source": "The Middle S01E03",
      "media": {
        "visual": "a1b2c3d4.mp4"
      },
      "analysis": [
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
      ],
      "tags": ["daily_life", "eating"],
      "deck": "PathEnglish-900/Unit 1",
      "difficulty": "A2",
      "origin": "official"
    }
  ]
}
```

---

## 7. Import Rules

When an app imports a `.passpack` file:

| Scenario | Required Behavior |
|----------|-------------------|
| Card UUID not found | Insert as new card |
| Card UUID already exists | Update card content (text, media, analysis, tags, etc.) |
| User data (progress, notes) on update | **MUST NOT** be overwritten |
| Notes on first import (new card) | Import as-is |
| Notes on update (card exists) | Keep local notes; imported notes SHOULD be stored separately (e.g. `importedNotes`) for the user to manually merge if desired |

This rule is **mandatory** for all conforming implementations. Users' learning progress and personal notes are sacred.

---

## 8. Extensibility

### 8.1 Custom Fields

Implementers MAY add custom fields with an `x_` prefix:

```json
{
  "uuid": "...",
  "schemaVersion": "passpack-v1",
  "text": "...",
  "x_myapp_difficulty_score": 7,
  "x_myapp_pronunciation_url": "https://..."
}
```

Conforming readers MUST ignore fields they don't recognize.

### 8.2 Custom Analysis Types

Use a namespaced prefix to avoid collisions:

```json
{
  "type": "x-duolingo-skill-assessment",
  "version": "1.0",
  "data": { ... }
}
```

### 8.3 Forward Compatibility

- Unknown fields → ignore silently
- Unknown analysis types → ignore silently
- Unknown `schemaVersion` major version → reject with clear error message

---

## 9. Versioning

### 9.1 Version Format

`passpack-v{major}` — single integer, no minor/patch.

Current: `passpack-v1`

### 9.2 Evolution Rules

| Change type | Version impact |
|-------------|----------------|
| New optional field | Same version (add to changelog) |
| New official analysis type | Same version |
| Field type change | New major version |
| Field removal | New major version |
| Semantic change to existing field | New major version |

### 9.3 Deprecation

1. Mark field as deprecated in spec
2. Maintain for at least 1 major version
3. Remove in next major version

---

## 10. Naming Conventions

| Scope | Convention | Example |
|-------|------------|---------|
| JSON fields | camelCase | `sourceLang`, `cardType` |
| Media file names | UUID-based (recommended, not required) | `a1b2c3d4.mp4` |
| Enum values | lowercase | `sentence`, `known`, `ai+human` |
| Deck separator | `/` | `IELTS/Listening/Part 1` |
| Custom fields | `x_` prefix | `x_myapp_score` |
| Custom analysis types | `x-` prefix | `x-myapp-grammar` |
| Pack files | `.passpack` | `unit_01.passpack` |

---

## 11. Content Licensing

PassPack separates specification licensing from content licensing.

### 11.1 Two Layers

| Layer | License | Scope |
|-------|---------|-------|
| **This specification** | CC BY-SA 4.0 | Anyone can implement readers/writers |
| **Card content** | Author's choice | Set via `license` in manifest |

### 11.2 Recommended Default: CC BY-NC-SA 4.0

The recommended default for community-contributed content is [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) — personal use and sharing allowed, commercial resale prohibited.

Pack authors MAY choose any license by setting `manifest.license`.

---

## 12. File Identity

| Property | Value |
|----------|-------|
| File extension | `.passpack` |
| MIME type | `application/vnd.passpack+zip` |
| UTI (Apple) | `com.passpack.cardpack` (conforms to `com.pkware.zip-archive`) |

---

## Appendix A: Official Analysis Types

These are the initial analysis types defined by the PassPack project. Each has its own `data` schema.

### A.1 `logicBlocks` — Phrase-Level Breakdown

The signature PassPack analysis: break a sentence into semantic chunks with natural translation.

**data schema:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `blocks` | array | ✅ | Phrase-meaning pairs |
| `blocks[].phrase` | string | ✅ | Original phrase |
| `blocks[].meaning` | string | ✅ | Translation with optional notes in parentheses |
| `vibeTranslation` | string | ✅ | Natural, idiomatic full-sentence translation |

**Example:**

```json
{
  "type": "logicBlocks",
  "version": "1.0",
  "targetLang": "zh-CN",
  "data": {
    "blocks": [
      { "phrase": "I'm gonna", "meaning": "我将要（口语，I am going to 的缩写）" },
      { "phrase": "grab a bite", "meaning": "吃点东西（非正式）" }
    ],
    "vibeTranslation": "我去吃点东西。"
  }
}
```

### A.2 `usageGuide` — Contextual Usage

Deeper analysis of when, where, and how to use the expression.

**data schema:**

The `usageGuide` data object is intentionally flexible. Recommended fields:

| Field | Type | Description |
|-------|------|-------------|
| `formality` | string | `"formal"` / `"neutral"` / `"casual"` |
| `bestFor` | string[] | Suitable contexts |
| `avoidWith` | string[] | Contexts to avoid |
| `grammar` | object | `{ pattern, note }` |
| `commonMistakes` | array | `[{ wrong, why }]` |
| `culturalNote` | string | Cultural context |

Apps MAY include any subset of these fields. Apps reading `usageGuide` SHOULD render whatever fields are present and ignore unknown ones.

### A.3 `definition` — Vocabulary Definition

For vocabulary cards.

**data schema:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `pronunciation` | string | ❌ | IPA or other phonetic notation |
| `partOfSpeech` | string | ❌ | `"noun"`, `"verb"`, `"adj"`, etc. |
| `definitions` | array | ✅ | `[{ meaning, example? }]` |

**Example:**

```json
{
  "type": "definition",
  "version": "1.0",
  "targetLang": "zh-CN",
  "data": {
    "pronunciation": "/baɪt/",
    "partOfSpeech": "noun",
    "definitions": [
      { "meaning": "一口；咬", "example": "Take a bite of this cake." },
      { "meaning": "（蚊虫的）叮咬", "example": "I got a mosquito bite." }
    ]
  }
}
```

---

---

## Appendix B: Full Card Example

```json
{
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "schemaVersion": "passpack-v1",
  "text": "I'm gonna grab a bite.",
  "cardType": "sentence",
  "sourceLang": "en",
  "targetLang": "zh-CN",
  "source": "The Middle S01E03",
  "media": {
    "visual": "a1b2c3d4.mp4",
    "audio": "a1b2c3d4.m4a"
  },
  "analysis": [
    {
      "type": "logicBlocks",
      "version": "1.0",
      "targetLang": "zh-CN",
      "generatedBy": "ai+human",
      "data": {
        "blocks": [
          { "phrase": "I'm gonna", "meaning": "我将要（口语，I am going to 的缩写）" },
          { "phrase": "grab a bite", "meaning": "吃点东西（非正式）" }
        ],
        "vibeTranslation": "我去吃点东西。"
      }
    },
    {
      "type": "usageGuide",
      "version": "1.0",
      "targetLang": "zh-CN",
      "generatedBy": "ai",
      "data": {
        "formality": "casual",
        "bestFor": ["朋友", "家人", "同事"],
        "avoidWith": ["正式邮件", "面试"],
        "grammar": {
          "pattern": "be going to → gonna + verb",
          "note": "gonna 是口语标配，书面语用 going to"
        },
        "commonMistakes": [
          { "wrong": "I will gonna eat.", "why": "重复助动词" }
        ],
        "culturalNote": "grab a bite 是北美日常高频表达，比 have a meal 更随意。"
      }
    }
  ],
  "tags": ["daily_life", "eating"],
  "deck": "PathEnglish-900/Unit 1",
  "notes": "",
  "difficulty": "A2",
  "origin": "official",
  "progress": {
    "level": "known",
    "retention": {
      "probability": 0.85,
      "estimatedAt": "2026-02-19T10:00:00Z"
    },
    "reviewLog": [
      { "date": "2026-01-15", "rating": 3 },
      { "date": "2026-01-22", "rating": 4 },
      { "date": "2026-02-10", "rating": 2 }
    ]
  },
  "createdAt": "2026-01-20T08:00:00Z",
  "updatedAt": "2026-02-19T10:00:00Z"
}
```

---

*This specification is maintained by the PassPack project.*  
*GitHub: https://github.com/passpack-spec*  
*Feedback and contributions welcome.*
