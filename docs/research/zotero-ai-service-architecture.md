# Zotero AI service architecture constraints

Research for [调研 Zotero AI 阅读产品与服务端架构约束](https://github.com/clementineyyy/BookMind/issues/4).

## Findings

### Windows plugin to service

Zotero plugins run with access to Zotero's internal JavaScript API and privileged platform APIs. Zotero documents plugin development and the JavaScript API, while warning that the API is incomplete and under-documented. A plugin can communicate with a developer-hosted service over HTTPS; it should handle request timeouts, cancellation, retries, and errors. Zotero's local API at `localhost:23119` is a local library interface, not a remote AI service.

Sources: [Zotero plugin development](https://github.com/zotero/zotero-docs/blob/main/content/dev/client_coding/plugin_development.md), [Zotero JavaScript API](https://github.com/zotero/zotero-docs/blob/main/content/dev/client_coding/javascript_api.md), [Zotero Web API basics](https://github.com/zotero/zotero-docs/blob/main/content/dev/web_api/v3/basics.md).

### Accounts, authorization, and provider keys

Provider keys must stay on the developer's server. OpenAI explicitly advises against embedding API keys in client-side apps; a static credential shipped in the plugin can be extracted and abused. Authenticate plugin users with credentials that can be revoked and expire, then enforce authorization, rate limits, and usage attribution per account.

Sources: [OpenAI API key safety](https://help.openai.com/en/articles/5112595-best-practices-for-api-key-safety), [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html).

### Streaming and long tasks

OpenAI Responses and Anthropic Messages support incremental streaming; Anthropic streams events over server-sent events. Anthropic recommends streaming or its batch API for long requests, particularly those exceeding ten minutes, because long-lived connections can fail. Decide which work should stream interactively and which should use resumable background jobs with job IDs, progress, polling, and cancellation.

Sources: [OpenAI Responses migration guide](https://platform.openai.com/docs/guides/migrate-to-responses), [Anthropic API errors and long requests](https://docs.anthropic.com/en/api/errors).

### Cost and abuse controls

Provider rate and spend limits do not enforce product-level budgets per user. The service needs its own request and token limits, per-account and global rate limits, concurrency caps, and usage metering. Decide whether expensive whole-book processing requires confirmation, a quota, or a queue.

Sources: [Anthropic API errors and limits](https://docs.anthropic.com/en/api/errors), [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html).

### Provider API capabilities

OpenAI Responses, Anthropic Messages, and Google's Gemini `generateContent` and streaming APIs differ in request formats, streaming events, limits, and model lifecycle. Decide whether the product offers one provider or provider choice/fallback. A provider-neutral internal request/response model and provider adapters can be implementation details once that product promise is set.

Sources: [OpenAI Responses API](https://platform.openai.com/docs/api-reference/responses), [Anthropic Messages API](https://docs.anthropic.com/en/api/messages), [Gemini API](https://ai.google.dev/api).

## Decisions before architecture selection

1. Whether accounts are required, including identity, recovery, revocation, and whether anonymous use is allowed.
2. Whether the developer-hosted service is the sole model route and which providers/models are supported initially.
3. What book content is sent, retained, logged, and deleted, including user controls for opting out or removal.
4. Whether book summaries run interactively with streaming or as resumable background jobs.
5. What quota, abuse, and concurrency rules bound developer-funded model usage.

These choices determine identity and authorization boundaries, provider connectivity, data persistence and retention, job orchestration, and metering. HTTP retry behavior, provider adapter interfaces, and the particular token-storage mechanism can be decided during implementation after those boundaries are chosen.
