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
ANTHROPIC_MODEL="google/gemma-4-31b-it:free"
p ${CLAUDE_PROXY}
app ${APPNAME} run
```

### deepseek
```sh evaluate
#app ai/services deepseek
app ai/services openrouter
ANTHROPIC_MODEL="deepseek/deepseek-v4-pro"
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
