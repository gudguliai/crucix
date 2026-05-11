# Crucix Local Changes

Patches to connect Crucix's LLM layer to a local OpenAI-compatible proxy (deepseek-v4-flash on port 4000).

## Changes

### 1. `crucix.config.mjs` — config base URL

**Line 13:** Added `OPENAI_BASE_URL` env var with `OLLAMA_BASE_URL` fallback.

```js
baseUrl: process.env.OPENAI_BASE_URL || process.env.OLLAMA_BASE_URL || null,
```

### 2. `lib/llm/openai.mjs` — custom endpoint support

**Line 11:** Accept `baseUrl` in constructor, default to OpenAI.

```js
this.baseUrl = config.baseUrl || 'https://api.openai.com/v1';
```

**Line 17:** Use configurable base URL instead of hardcoded.

```js
const res = await fetch(`${this.baseUrl}/chat/completions`, {
```

### 3. `lib/llm/index.mjs` — pass baseUrl to provider

**Line 38:** Pass `baseUrl` from config to OpenAIProvider.

```js
return new OpenAIProvider({ apiKey, model, baseUrl: llmConfig.baseUrl });
```

## Configuration (`.env`)

```env
LLM_PROVIDER=openai
LLM_API_KEY=litellm-local
LLM_MODEL=deepseek-v4-flash
OPENAI_BASE_URL=http://localhost:4000/v1
TELEGRAM_BOT_TOKEN=<your-bot-token>
TELEGRAM_CHAT_ID=<your-chat-id>
```

---

## Session: 2026-05-11

### 4. `dashboard/public/jarvis.html` — fix stale data on server mode

**Bug:** The `DOMContentLoaded` handler only fetched `/api/data` when `hasInlineData` was false. Since `jarvis.html` has data baked-in from the last CLI inject run, `hasInlineData` was always true — so the dashboard never fetched live data from the server, never connected SSE, and showed stale Gold/Silver prices and old news.

**Fix:** Changed the condition to always fetch from the API when served via HTTP.

```js
// Before
if (canProbeApi && !hasInlineData) {

// After
if (canProbeApi) {
```

### 5. `.env` — fix LLM API key to match LiteLLM master key

The proxy at `localhost:4000` requires `Authorization: Bearer litellm-local` (set as `master_key` in `~/.litellm/deepseek-config.yaml`). The old value `sk-local` was rejected with a 400 auth error.

```env
LLM_API_KEY=litellm-local
```

### 6. `~/.litellm/deepseek-config.yaml` — disable DB requirement

LiteLLM returned `"No connected db."` on every request because spend logging requires a Postgres/Prisma database. Added one line to disable it.

```yaml
general_settings:
  master_key: "litellm-local"
  disable_spend_logs: True
```

Also applied the same fix to `~/dotfiles/litellm/deepseek-config.yaml` (the source-of-truth copy).
