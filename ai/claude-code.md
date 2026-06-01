# Claude Code

### config
```ini
[claude]
  proxy="ndisguise"
```

### install
```sh
npm install -g @anthropic-ai/claude-code
```

### default alias run
```sh evaluate
echo "Using ${ANTHROPIC_MODEL}"
${NPMGLOBAL}/bin/claude --model ${ANTHROPIC_MODEL}
```

### gemma4
```sh evaluate
app ai/services openrouter
ANTHROPIC_MODEL="google/gemma-4-26b-a4b-it:free"
p ${CLAUDE_PROXY}
app ${APPNAME} run
```

### nemotron
```sh evaluate
app ai/services openrouter
ANTHROPIC_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
p ${CLAUDE_PROXY}
app ${APPNAME} run
```

### `deepseek-pro`
```sh evaluate
app ai/services deepseek
ANTHROPIC_MODEL="deepseek-v4-pro"
#app ai/services openrouter
#ANTHROPIC_MODEL="deepseek/deepseek-v4-pro"
app ${APPNAME} run
```

### `deepseek-flash`
```sh evaluate
app ai/services deepseek
ANTHROPIC_MODEL="deepseek-v4-flash"
#app ai/services openrouter
#ANTHROPIC_MODEL="deepseek/deepseek-v4-flash"
app ${APPNAME} run
```

### proxied
```sh evaluate
ANTHROPIC_MODEL="default"
p ${CLAUDE_PROXY}
app ${APPNAME} run
```

### serve
```sh evaluate
${NPMGLOBAL}/bin/claude mcp serve
```
