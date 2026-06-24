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
export ANTHROPIC_BASE_URL=${OPENROUTER_URL}
if [[ -z "${(P)${(U)OPENROUTER_KEYNAME}}" ]]; then
  secrets var ${OPENROUTER_KEYNAME}
fi
export ANTHROPIC_AUTH_TOKEN="${(P)${(U)OPENROUTER_KEYNAME}}"
export ANTHROPIC_API_KEY="" # Important: Must be explicitly empty

```

### deepseek
```sh evaluate
export ANTHROPIC_BASE_URL=${DEEPSEEK_URL}
if [[ -z "${(P)${(U)DEEPSEEK_KEYNAME}}" ]]; then
  secrets var ${DEEPSEEK_KEYNAME}
fi
export ANTHROPIC_AUTH_TOKEN=${(P)${(U)DEEPSEEK_KEYNAME}}
export ANTHROPIC_API_KEY=""
```

### default alias
```sh
echo "This commands needs ro run with an option"
```

