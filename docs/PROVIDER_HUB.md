# Provider Hub

[Provider Hub](https://github.com/DigitalRavagers/provider-hub) is an OpenAI-compatible gateway
where each customer connects the model access they already pay for:

| Section | Options |
| --- | --- |
| Coding agents | Claude Code, Codex (the CLI login already on the Hub's machine), any custom ACP agent (opencode, Hermes, OpenClaw, pi) |
| Sign in with a subscription | ChatGPT, GitHub Copilot, Qwen Code |
| Bring your own API key | OpenAI, Anthropic, Google, OpenRouter, xAI, Moonshot, DeepSeek, Mistral, Groq, Kilo Code, Azure, Bedrock, any OpenAI-compatible endpoint |
| Runs on this machine | Ollama, LMStudio |

OpenBot needs no code change for this. It already moves the whole deployment with one
`OPENAI_BASE_URL` (see [configuration](configuration.md)), and the Hub answers there.

## Relation to OpenBot's own sign-ins

OpenBot's desktop setup already signs in natively to:
- a Claude plan, through the `agent-claude-sdk` harness
- a ChatGPT plan
- Google and xAI, through OAuth
- OpenAI, Anthropic and Google, with API keys

Keep using those where they fit. The Hub adds:
- GitHub Copilot and Qwen Code sign-in
- OpenRouter, Moonshot, DeepSeek, Mistral, Groq, Kilo, Azure and Bedrock
- local Ollama and LMStudio
- coding agents over ACP
- one screen where the provider can be switched without restarting OpenBot

## Connect

1. Start the Hub (`npm start` in the provider-hub repository). On first start it prints a hub token.
2. Open `http://127.0.0.1:4848`, paste the token, add a provider, and mark it default.
3. Point OpenBot at it.

### Desktop

In model setup, choose **OpenAI-compatible endpoint** and enter:

| Field | Value |
| --- | --- |
| Endpoint | `http://127.0.0.1:4848/v1` |
| Container endpoint | `http://host.docker.internal:4848/v1` |
| Model | `default` |
| API key | the hub token |

### Server (`.env`)

```
OPENAI_BASE_URL=http://127.0.0.1:4848/v1
OPENAI_CONTAINER_BASE_URL=http://host.docker.internal:4848/v1
OPENAI_API_KEY=<hub token>
BOT_MODEL=default
AGENT_BOT_MODEL=default
```

Use `<provider name>` or `<provider name>/<model>` in `BOT_MODEL` to pin one provider instead of
following the Hub's default.

## Containers

The Bots run in containers. They reach the Hub at `OPENAI_CONTAINER_BASE_URL`:
- **Docker Desktop (Windows, macOS):** `host.docker.internal` reaches the host.
- **Linux:** add `extra_hosts: ["host.docker.internal:host-gateway"]` and start the Hub with
  `HUB_HOST=0.0.0.0` behind a firewall. Alternatively, run the Hub in the same compose network and
  use its service name.

## Model choice

Bots that use tools need a model that is good at tool calls. For OpenAI's newest models (gpt-5.6
and later), which accept tools only through the Responses API, the Hub converts requests itself,
so the Bots keep speaking `/v1/chat/completions`. `BOT_RESPONSES_API` can stay unset.

Coding agents (Claude Code, Codex) answer with text and use their own tools on the Hub's machine.
They cannot call OpenBot's gateway tools. For coworkers that act through OpenBot's computer and
MCP tools, pick a model provider instead.

Voice (`VOICE_API_KEY`) and dictation (`TRANSCRIPTION_*`) keep their own settings and do not go
through the Hub.

## Verified

`agent-bot` started with `OPENAI_BASE_URL` pointed at the Hub and `BOT_MODEL=default`. It streamed a
complete AG-UI run (`RUN_STARTED` → text → `RUN_FINISHED`) answered by the provider selected in the
Hub, using that provider's own model and key.
