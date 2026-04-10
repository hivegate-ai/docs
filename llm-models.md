# Supported LLM Models


The `Model` enum defines all supported models:

### OpenAI

| Enum Value | Model ID | Description |
|------------|----------|-------------|
| `gpt_5_4` | `gpt-5.4` | GPT-5.4 full |
| `gpt_5_4_mini` | `gpt-5.4-mini` | GPT-5.4 mini |
| `gpt_5_4_nano` | `gpt-5.4-nano` | GPT-5.4 nano |

### Google Gemini (Stable)

| Enum Value | Model ID | Description |
|------------|----------|-------------|
| `gemini_2_5_pro` | `gemini-2.5-pro` | Gemini 2.5 Pro (default) |
| `gemini_2_5_flash` | `gemini-2.5-flash` | Gemini 2.5 Flash |
| `gemini_2_5_flash_lite` | `gemini-2.5-flash-lite` | Gemini 2.5 Flash Lite |

### Google Gemini (Preview)

| Enum Value | Model ID | Description |
|------------|----------|-------------|
| `gemini_3_1_pro` | `gemini-3.1-pro-preview` | Gemini 3.1 Pro Preview |
| `gemini_3_flash` | `gemini-3-flash-preview` | Gemini 3 Flash Preview |

### Anthropic Claude

| Enum Value | Model ID | Description |
|------------|----------|-------------|
| `claude_opus_4_6` | `claude-opus-4-6` | Claude Opus 4.6 |
| `claude_sonnet_4_6` | `claude-sonnet-4-6` | Claude Sonnet 4.6 |
| `claude_haiku_4_5` | `claude-haiku-4-5-20251001` | Claude Haiku 4.5 |

### Provider Detection

The provider is determined automatically from the model ID prefix:
- `gpt-*` -> OpenAI
- `gemini-*` -> Google Gemini
- `claude-*` -> Anthropic

---

