# AI Services

### config
```ini
[openrouter]
  url = "https://openrouter.ai/api"
  key = "openrouter_key"

[deepseek]
  url = "https://api.deepseek.com/anthropic"
  key = "deepseek_key"

[fireworks]
  url = "https://api.fireworks.ai/inference"
```

### openrouter
```sh evaluate
export ANTHROPIC_API_KEY=$(get_secret ${OPENROUTER_KEY})
export ANTHROPIC_BASE_URL=${OPENROUTER_URL}
```

### deepseek
```sh evaluate
export ANTHROPIC_API_KEY=$(get_secret ${DEEPSEEK_KEY})
export ANTHROPIC_BASE_URL=${DEEPSEEK_URL}
```

