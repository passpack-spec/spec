# PassPack

**Open Card Format for Language Learning**

PassPack is an open standard for rich, AI-ready language learning flashcards. It defines how cards, media, AI-generated analysis, and learning progress are structured, packaged, and exchanged between applications.

## Why PassPack?

The language learning space has no open standard for rich flashcards:

- **Anki (.apkg)** — No official spec, undocumented format
- **Quizlet** — Completely proprietary
- **Mochi** — Private format
- **CSV** — Plain text, no structure

Meanwhile, AI is generating structured linguistic analysis at scale — but every app stores it differently.

**PassPack lets learning cards flow freely — anyone can create them, any tool can open them, and they always belong to the learner.**

## Key Features

| Feature | Description |
|---------|-------------|
| 📄 **Structured cards** | JSON-based, not raw HTML or plain text |
| 🎬 **Rich media** | Video clips, audio, images bundled in .passpack files |
| 🤖 **AI analysis layer** | Extensible, typed analysis — the first open standard for AI-generated learning content |
| 📊 **CEFR difficulty** | International standard difficulty rating |
| 🔄 **Portable progress** | Algorithm-agnostic learning history (level + retention + review log) |
| 🏷️ **Tags + Deck hierarchy** | Flexible organization with `/` separated deck paths |
| 📝 **User notes** | Dedicated field, protected during import |
| 🔓 **Truly open** | CC BY-SA 4.0 spec — anyone can implement |

## Specification

- 📖 **[English (Full)](./PASSPACK_SPEC.md)** — Complete technical specification
- 📖 **[中文版](./PASSPACK_SPEC.zh-CN.md)** — Chinese version for the community

## Quick Example

A minimal PassPack card:

```json
{
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "schemaVersion": "passpack-v1",
  "text": "I'm gonna grab a bite."
}
```

A rich card with media and AI analysis:

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
  "deck": "American TV/Unit 1",
  "difficulty": "A2",
  "tags": ["daily_life", "eating"]
}
```

## .passpack File

A `.passpack` file is a ZIP archive:

```
unit_01.passpack
├── manifest.json
└── media/
    ├── a1b2c3d4.mp4
    └── ...
```

## Comparison

| Feature | Anki | Mochi | Quizlet | **PassPack** |
|---------|------|-------|---------|-------------|
| Has formal spec | ❌ | ❌ | ❌ | **✅** |
| AI analysis layer | ❌ | ❌ | ❌ | **✅** |
| Structured fields | ❌ | ❌ | ❌ | **✅** |
| Rich media | ✅ | ✅ | ❌ | **✅** |
| Progress separated | ❌ | ❌ | ❌ | **✅** |
| Open format | ✅ | Partial | ❌ | **✅** |

## Status

🚧 **Draft** — We're finalizing v1. Feedback welcome via [Issues](https://github.com/passpack-spec/spec/issues).

## License

This specification is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

## Links

- 🌐 Website: [passpack.dev](https://passpack.dev) (coming soon)
- 💬 Feedback: [GitHub Issues](https://github.com/passpack-spec/spec/issues)
