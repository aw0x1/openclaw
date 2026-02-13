# Agent Capabilities

## Primary Model

Provider: OpenRouter — openrouter/auto (free tier, auto-selects best available model)

## Available Tools

| Tool | Purpose |
|------|---------|
| read | Read files from the workspace |
| write | Write files to the workspace |
| edit | Edit existing files |
| web_search | Search the web |
| web_fetch | Fetch a URL |
| message | Send messages via connected channels |
| tts | Text-to-speech output |

## Channels (configure post-deploy)

- Telegram: disabled (token not yet provisioned)
- Discord: disabled (token not yet provisioned)

## Concurrency Limits

Enforce a maximum of 5 concurrent top-level agents and 5 subagents each.
Queue additional requests rather than spawning beyond this limit.
