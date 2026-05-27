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
${NPMGLOBAL}/bin/claude
```

### proxied
```sh evaluate
p ${CLAUDE_PROXY}
app ${APPNAME} run
```

### serve
```sh evaluate
${NPMGLOBAL}/bin/claude mcp serve
```
