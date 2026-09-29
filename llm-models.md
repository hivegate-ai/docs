# Supported LLM Models

Set a model per agent, or per request with the `model` field. You can use a pinned model id or a [latest-of-a-tier alias](#latest-of-a-tier-aliases).

## Default model

When a request doesn't name a `model`, HiveGate uses `DEFAULT_CHAT_MODEL`, and falls back to `gemini-3-flash-preview` if that isn't set. The value must be a model id or alias from this page. An unknown value stops the server at startup, instead of failing every chat that omits `model`.

```sh
# .env
DEFAULT_CHAT_MODEL=anthropic:sonnet-latest
```

## Latest-of-a-tier aliases

An alias follows the vendor's newest model in a tier without a code change, and never moves to a pricier tier. HiveGate resolves an alias by listing the vendor's models and taking the newest one in the tier. It caches the answer for 24 hours and logs it (`Model alias anthropic:sonnet-latest -> ...`). If the vendor can't be reached, it uses the last known answer, or the fallback below, and retries after 5 minutes.

| Alias | Fallback |
|---|---|
| `anthropic:haiku-latest` | `claude-haiku-4-5` |
| `anthropic:sonnet-latest` | `claude-sonnet-5` |
| `anthropic:opus-latest` | `claude-opus-5` |
| `openai:luna-latest` | `gpt-6-luna` |
| `openai:terra-latest` | `gpt-5.6-terra` |
| `openai:sol-latest` | `gpt-6-sol` |
| `openai:astra-latest` | `gpt-6-astra` |
| `google:flash-lite-latest` | `gemini-3.5-flash-lite` |
| `google:flash-latest` | `gemini-3.8-flash` |
| `google:pro-latest` | `gemini-3.1-pro-preview` |
| `xai:grok-latest` | `grok-4.7` |

Prefer aliases for defaults, and pinned ids when you need a model that never changes.

## Pinned models

### OpenAI: GPT-5.4

| Model ID | Enum member |
|---|---|
| `gpt-5.4` | `gpt_5_4` |
| `gpt-5.4-mini` | `gpt_5_4_mini` |
| `gpt-5.4-nano` | `gpt_5_4_nano` |

### OpenAI: GPT-5.6

| Model ID | Enum member |
|---|---|
| `gpt-5.6-luna` | `gpt_5_6_luna` |
| `gpt-5.6-terra` | `gpt_5_6_terra` |
| `gpt-5.6-sol` | `gpt_5_6_sol` |

### OpenAI: GPT-6 (current flagship)

| Model ID | Enum member |
|---|---|
| `gpt-6-luna` | `gpt_6_luna` |
| `gpt-6-sol` | `gpt_6_sol` |
| `gpt-6-astra` | `gpt_6_astra` |

GPT-6 runs on the OpenAI Responses API. There is no `gpt-6-terra`, so `openai:terra-latest` resolves to `gpt-5.6-terra`.

### Google Gemini 3.x (stable)

| Model ID | Enum member |
|---|---|
| `gemini-3.8-flash` | `gemini_3_8_flash` |
| `gemini-3.7-flash` | `gemini_3_7_flash` |
| `gemini-3.6-flash` | `gemini_3_6_flash` |
| `gemini-3.5-flash` | `gemini_3_5_flash` |
| `gemini-3.5-flash-lite` | `gemini_3_5_flash_lite` |
| `gemini-3.1-flash-lite` | `gemini_3_1_flash_lite` |

### Google Gemini 2.5 (existing Google users only)

| Model ID | Enum member |
|---|---|
| `gemini-2.5-pro` | `gemini_2_5_pro` |
| `gemini-2.5-flash` | `gemini_2_5_flash` |
| `gemini-2.5-flash-lite` | `gemini_2_5_flash_lite` |

Google now serves Gemini 2.5 only to existing users: new API keys get **404 NOT_FOUND**. Prefer Gemini 3.x or a `google:*-latest` alias.

### Google Gemini (preview)

| Model ID | Enum member |
|---|---|
| `gemini-3.1-pro-preview` | `gemini_3_1_pro` |
| `gemini-3-flash-preview` | `gemini_3_flash` |

### Anthropic Claude

| Model ID | Enum member |
|---|---|
| `claude-fable-5-1` | `claude_fable_5_1` |
| `claude-opus-5-5` | `claude_opus_5_5` |
| `claude-sonnet-5-5` | `claude_sonnet_5_5` |
| `claude-haiku-4-5-20251001` | `claude_haiku_4_5` |
| `claude-haiku-4-5` | `claude_haiku_4_5_undated` |
| `claude-fable-5` | `claude_fable_5` |
| `claude-opus-5` | `claude_opus_5` |
| `claude-opus-4-8` | `claude_opus_4_8` |
| `claude-opus-4-7` | `claude_opus_4_7` |
| `claude-opus-4-6` | `claude_opus_4_6` |
| `claude-opus-4-5` | `claude_opus_4_5` |
| `claude-sonnet-5` | `claude_sonnet_5` |
| `claude-sonnet-4-6` | `claude_sonnet_4_6` |
| `claude-sonnet-4-5` | `claude_sonnet_4_5` |

Every Claude id here is a pinned snapshot, including the ones without a date. Only the `anthropic:*-latest` aliases move. The current lineup is `claude-fable-5-1`, `claude-opus-5-5`, `claude-sonnet-5-5` and `claude-haiku-4-5-20251001`; the rest are older models that are still served.

### xAI Grok

| Model ID | Enum member |
|---|---|
| `grok-4.7` | `grok_4_7` |
| `grok-4.6` | `grok_4_6` |
| `grok-4.5` | `grok_4_5` |
| `grok-4.3` | `grok_4_3` |
| `grok-4.20-0309-reasoning` | `grok_4_20_reasoning` |
| `grok-4.20-0309-non-reasoning` | `grok_4_20_non_reasoning` |

### Z.ai GLM

| Model ID | Enum member |
|---|---|
| `glm-5.3` | `glm_5_3` |
| `glm-5.3-flashx` | `glm_5_3_flashx` |
| `glm-5.3-flash` | `glm_5_3_flash` |
| `glm-5.2` | `glm_5_2` |

Served through Z.ai's OpenAI-compatible endpoint. Z.ai documents no endpoint for listing models, so there is no `-latest` alias. Only models with documented function calling are listed, because every HiveGate agent has tools.

### DeepSeek

| Model ID | Enum member |
|---|---|
| `deepseek-flash` | `deepseek_flash` |
| `deepseek-v4-pro` | `deepseek_v4_pro` |

`deepseek-flash` is a moving name (currently V4.1-Flash).

## Provider keys

| Provider | Model ids start with | API key variable |
|---|---|---|
| OpenAI | `gpt-`, `openai:` | `OPENAI_API_KEY` |
| Google Gemini | `gemini-`, `google:` | `GOOGLE_API_KEY` |
| Anthropic | `claude-`, `anthropic:` | `ANTHROPIC_API_KEY` |
| xAI | `grok-`, `xai:` | `XAI_API_KEY` |
| Z.ai | `glm-` | `ZAI_API_KEY` |
| DeepSeek | `deepseek-` | `DEEPSEEK_API_KEY` |

## Claude refusals

When a Claude safety classifier declines a request, Anthropic returns no content. HiveGate reports this as a run error with a stable message, `stop_reason=refusal, category=..., model=...`, which reaches streaming clients as an error event. The message doesn't include any request content. Previously a refusal showed up as an empty reply.
