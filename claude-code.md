# Claude Code

### config
```ini
[claude]
  proxy="ndisguise"

[openrouter]
  url = "https://openrouter.ai/api"

[fireworks]
  url = "https://api.fireworks.ai/inference"

[deepseek]
  url = "https://api.deepseek.com/anthropic"
```

### install
```sh
npm install -g @anthropic-ai/claude-code
```

### default alias run
```sh evaluate
${NPMGLOBAL}/bin/claude
```

### proxied
```sh evaluate
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=global
p ${CLAUDE_PROXY}
app ${APPNAME} run
```

### serve
```sh evaluate
${NPMGLOBAL}/bin/claude mcp serve
```
