# ChatFate Agent Plugin

ChatFate's public Codex plugin package. It contains only the client-side plugin manifest, skills, brand assets, and the declaration for the hosted ChatFate MCP service.

The calculation engine, report application, authentication, payments, databases, deployment configuration, and service source code are not part of this repository.

## Install in Codex Desktop

Use the Codex CLI bundled with the desktop app:

```sh
"/Applications/ChatGPT.app/Contents/Resources/codex" plugin marketplace add https://github.com/ChatFate/agent-plugin.git --ref main
"/Applications/ChatGPT.app/Contents/Resources/codex" plugin add chatfate@chatfate
```

Then open a new Codex task and select ChatFate.

## Service

- Website: https://chatfate.cc
- MCP endpoint: https://mcp.chatfate.cc/
- Privacy: https://chatfate.cc/privacy
- Terms: https://chatfate.cc/terms
