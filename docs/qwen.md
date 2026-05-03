# Qwen Code

Qwen Code supports ACP natively via the `--acp` flag — no additional wrapper or protocol translation layer is needed beyond OpenAB itself.

Qwen Code is an open-source coding CLI (Apache 2.0) forked from [gemini-cli](https://github.com/google-gemini/gemini-cli) and adapted for the [Qwen](https://qwenlm.github.io/) family of models. It can be used with the Qwen API (DashScope) or any compatible OpenAI-style endpoint.

## Docker Image

```bash
docker build -f Dockerfile.qwen -t openab-qwen:latest .
```

The image installs `@qwen-code/qwen-code` globally via npm on `node:22-bookworm-slim`.

## Helm Install

```bash
helm install openab openab/openab \
  --set agents.kiro.enabled=false \
  --set agents.qwen.discord.enabled=true \
  --set-string 'agents.qwen.discord.allowedChannels[0]=YOUR_CHANNEL_ID' \
  --set agents.qwen.image=ghcr.io/openabdev/openab-qwen:latest \
  --set agents.qwen.command=qwen \
  --set 'agents.qwen.args={--acp}' \
  --set agents.qwen.workingDir=/home/node \
  --set agents.qwen.env.QWEN_API_KEY="$QWEN_API_KEY"
```

> Set `agents.kiro.enabled=false` to disable the default Kiro agent.

## Manual config.toml

```toml
[agent]
command = "qwen"
args = ["--acp"]
working_dir = "/home/node"
env = { QWEN_API_KEY = "${QWEN_API_KEY}" }
```

## Authentication

Qwen Code supports API key authentication via the `QWEN_API_KEY` environment variable (DashScope key):

1. Obtain a DashScope API key from [dashscope.aliyun.com](https://dashscope.aliyun.com/).
2. Pass it as an environment variable in the agent config or via a Kubernetes secret.

Alternatively, you can authenticate interactively inside the pod:

```bash
kubectl exec -it deployment/openab-qwen -- qwen auth login
```

## Notes

- **Binary name**: The CLI binary is `qwen` (installed by the `@qwen-code/qwen-code` npm package).
- **ACP flag**: Pass `--acp` to start Qwen Code in ACP stdio mode.
- **Model selection**: Set the default model via Qwen Code's settings file in the working directory (e.g. `~/.qwen/settings.json`). The `QWEN_API_KEY` env var is only for authentication.
