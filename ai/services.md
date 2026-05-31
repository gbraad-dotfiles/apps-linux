# AI Services

### config
```ini
[openrouter]
  url = "https://openrouter.ai/api"
  keyname = "openrouter_api_key"

[deepseek]
  url = "https://api.deepseek.com/anthropic"
  keyname = "deepseek_api_key"

[fireworks]
  url = "https://api.fireworks.ai/inference"
```

### openrouter
```sh evaluate
if [[ -z "${(P)${(U)OPENROUTER_KEYNAME}}" ]]; then
  secrets var ${OPENROUTER_KEYNAME}
fi
export ANTHROPIC_API_KEY="${(P)OPENROUTER_KEYNAME}"
export ANTHROPIC_BASE_URL=${OPENROUTER_URL}
```

### deepseek
```sh evaluate
if [[ -z "${(P)${(U)DEEPSEEK_KEYNAME}}" ]]; then
  secrets var ${DEEPSEEK_KEYNAME}
fi
export ANTHROPIC_API_KEY=${DEEPSEEK_API_KEY}
export ANTHROPIC_BASE_URL=${DEEPSEEK_URL}
```

