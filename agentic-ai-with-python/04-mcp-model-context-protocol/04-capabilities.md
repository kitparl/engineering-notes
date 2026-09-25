**Capabilities** in MCP are simply the **features** that the Client and Server tell each other they support.

During the Initialization (handshake), both sides exchange a list of capabilities.  
This way they know:

- What the other side can do
- What features are allowed in this session
- What features should **not** be used

---

### Simple Analogy

Think of it like two people meeting:

- **Client** says: “I can do these things: share folders, ask for AI sampling…”
- **Server** says: “I can do these things: give tools, provide resources, send logs…”

After this, they only use the features both agreed on.

---

### Common Capabilities

| Side   | Capability  | Meaning                                            | Example                                                     |
| ------ | ----------- | -------------------------------------------------- | ----------------------------------------------------------- |
| Client | roots       | Can provide file/folder roots                      | Share a project folder                                      |
| Client | sampling    | Can handle AI sampling requests                    | Server asks Client to call an LLM                           |
| Client | elicitation | Can ask the user for extra information when needed | Server pauses and asks user for missing data / confirmation |
| Server | tools       | Offers tools the AI can call                       | get_weather, search_web                                     |
| Server | resources   | Offers readable resources                          | Files, documents, data                                      |
| Server | prompts     | Offers ready-made prompt templates                 | “Summarize this code” template                              |
| Server | logging     | Can send structured log messages                   | Debug / info logs                                           |
| Server | completions | Supports auto-completion                           | Suggest arguments for tools                                 |
---

### Example from the Handshake

**Client capabilities:**
```json
"capabilities": {
  "roots": {
    "listChanged": true
  },
  "sampling": {}
}
```

**Server capabilities:**
```json
"capabilities": {
  "logging": {},
  "prompts": {
    "listChanged": true
  },
  "resources": {
    "subscribe": true,
    "listChanged": true
  },
  "tools": {
    "listChanged": true
  }
}
```

---

### Extra Details (Sub-capabilities)

Some capabilities can have extra options:

- `listChanged: true` → “I will notify you when the list of tools/resources/prompts changes”
- `subscribe: true` → “You can subscribe to changes of individual resources”

---

### Why Capabilities Matter

- Prevents errors (don’t call a tool if the server didn’t offer tools)
- Makes the connection flexible (different servers offer different features)
- Allows future features without breaking old clients/servers

**In short:**  
Capabilities = the list of features each side supports, agreed during the handshake.