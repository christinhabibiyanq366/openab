# Qwen Code

Qwen Code is an AI-powered coding CLI adapted from [Gemini CLI](https://github.com/google-gemini/gemini-cli),
optimized for the [Qwen3-Coder](https://github.com/QwenLM/Qwen3-Coder) family of models.
It supports ACP natively via the `--acp` flag — no adapter needed.

## Docker Image

```bash
docker build -f Dockerfile.qwen -t openab-qwen:latest .
```

The image installs `@qwen-code/qwen-code` globally via npm.

## Helm Install

```bash
helm install openab openab/openab \
  --set agents.kiro.enabled=false \
  --set agents.qwen.discord.enabled=true \
  --set agents.qwen.discord.botToken="$DISCORD_BOT_TOKEN" \
  --set-string 'agents.qwen.discord.allowedChannels[0]=YOUR_CHANNEL_ID' \
  --set agents.qwen.image=ghcr.io/openabdev/openab-qwen:latest \
  --set agents.qwen.command=qwen \
  --set agents.qwen.args='{--acp}' \
  --set agents.qwen.workingDir=/home/node
```

> Set `agents.kiro.enabled=false` to disable the default Kiro agent.

To authenticate with an API key rather than OAuth, pass the credentials as env vars:

```bash
  --set agents.qwen.env.OPENAI_API_KEY="$OPENAI_API_KEY" \
  --set agents.qwen.env.OPENAI_BASE_URL="https://dashscope-intl.aliyuncs.com/compatible-mode/v1" \
  --set agents.qwen.env.OPENAI_MODEL="qwen3-coder-plus"
```

## Manual config.toml

```toml
[agent]
command = "qwen"
args = ["--acp"]
working_dir = "/home/node"
```

### API Key Authentication (optional)

If you prefer to use an API key instead of Qwen OAuth, set the following env vars:

```toml
[agent]
command = "qwen"
args = ["--acp"]
working_dir = "/home/node"
# ⚠️ SECURITY WARNING: any env var listed here is accessible to the agent.
# International (Alibaba Cloud ModelStudio or OpenRouter):
env = { OPENAI_API_KEY = "${OPENAI_API_KEY}", OPENAI_BASE_URL = "https://dashscope-intl.aliyuncs.com/compatible-mode/v1", OPENAI_MODEL = "qwen3-coder-plus" }
# Mainland China (Alibaba Cloud Bailian):
# env = { OPENAI_API_KEY = "${OPENAI_API_KEY}", OPENAI_BASE_URL = "https://dashscope.aliyuncs.com/compatible-mode/v1", OPENAI_MODEL = "qwen3-coder-plus" }
```

## Authentication

### Option 1 — Qwen OAuth (recommended, free)

Run the interactive login inside the pod and follow the browser device flow:

```bash
kubectl exec -it deployment/openab-qwen -- qwen auth login
kubectl rollout restart deployment/openab-qwen
```

This grants **2,000 requests/day** at no cost with automatic credential refresh.

### Option 2 — API Key

Set `OPENAI_API_KEY`, `OPENAI_BASE_URL`, and `OPENAI_MODEL` in `[agent].env` (see above).

Available providers:

| Region | Provider | Endpoint |
|--------|----------|----------|
| International | [Alibaba Cloud ModelStudio](https://modelstudio.console.alibabacloud.com/) | `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` |
| International | [OpenRouter](https://openrouter.ai/) (free tier available) | `https://openrouter.ai/api/v1` |
| Mainland China | [Alibaba Cloud Bailian](https://bailian.console.aliyun.com/) | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| Mainland China | [ModelScope](https://modelscope.cn/docs/model-service/API-Inference/intro) (2,000 free calls/day) | `https://api-inference.modelscope.cn/v1` |

### Persisted Paths (PVC)

| Path | Contents |
|------|----------|
| `/home/node/.qwen/` | OAuth credentials and CLI settings |
