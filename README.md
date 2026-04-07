# codex-cursor-proxy

A small CLI proxy that lets you use your ChatGPT Plus/Pro subscription to access Codex models in third-party clients like [Cursor](https://cursor.sh). It's hacky but it works.

## Prerequisites

- [Bun](https://bun.sh) >= 1.0
- [Codex CLI](https://github.com/openai/codex) authenticated with your ChatGPT account (`codex` must have been run at least once so `~/.codex/auth.json` exists)

## Quick start

```bash
bunx codex-cursor-proxy
```

This starts a local proxy on port 3000 and opens a public tunnel via [localtunnel](https://theboroer.github.io/localtunnel-www/). On startup it prints:

```
OpenAI Base URL for Cursor: https://<subdomain>.loca.lt
API Key: Anything you like!
```

## Cursor setup

1. Open Cursor settings
2. Under **Models > OpenAI**, set:
   - **Base URL** to the tunnel URL printed at startup
   - **API Key** to any non-empty string (e.g. `x`)
3. Select a model name that matches what Codex supports (e.g. `gpt-5.4`)

## How it works

```
Cursor ──POST /chat/completions──> Proxy ──POST Responses API──> chatgpt.com
Cursor <──Chat Completions SSE──── Proxy <──Responses API SSE──── chatgpt.com
```

1. Cursor sends requests in the OpenAI Responses API format
2. The proxy sanitizes the payload (extracts system message into `instructions`, strips unsupported params, forces `stream: true` and `store: false`)
3. Forwards to `chatgpt.com/backend-api/codex/responses` using your Codex access token from `~/.codex/auth.json`
4. Translates the streaming Responses API events (`response.output_text.delta`, etc.) into Chat Completions SSE chunks (`data: {"choices":[...]}`)
5. Streams back to Cursor with `data: [DONE]` sentinel

## Configuration

The proxy stores a persistent tunnel subdomain in `~/.codex/cursor-proxy/config.json` so the URL stays the same across restarts.

## License

MIT
