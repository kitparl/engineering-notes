# How LLMs Select Tools in LangChain

## Does LangChain search tools one by one?

**Not exactly.**

When multiple tools are available, LangChain provides the tools and their descriptions/schema to the **LLM**. The LLM then determines which tool is appropriate for the user's request.

It is **not normally implemented as a simple `O(n)` loop** like:

```python
for tool in tools:
    if matches(tool, user_request):
        return tool
```

Instead, the LLM receives the available tool definitions as part of its context and uses its learned language representations and attention mechanisms to decide which tool to call.

---

## Example

Suppose we have these tools:

```python
@tool
def get_weather(city: str):
    """Get the current weather for a city."""
    ...


@tool
def search_database(query: str):
    """Search customer records in the database."""
    ...
```

The LLM effectively receives information similar to:

```text
Available tools:

1. get_weather
   Description: Get the current weather for a city.
   Input: city (string)

2. search_database
   Description: Search customer records in the database.
   Input: query (string)
```

User:

```text
What's the weather in Mumbai?
```

The LLM recognizes that the request matches `get_weather` and generates a structured tool call:

```json
{
  "name": "get_weather",
  "arguments": {
    "city": "Mumbai"
  }
}
```

LangChain then executes the selected tool.

---

# What is actually happening?

Conceptually:

```text
User request
     |
     v
+-------------+
|     LLM     |
+-------------+
     |
     | Understands request
     | and available tools
     v
Selects appropriate tool
     |
     v
+-------------+
| Weather     |
| Tool        |
+-------------+
     |
     v
Tool result
     |
     v
LLM
     |
     v
Final answer
```

---

# Is it O(n)?

It depends on what part of the system we are talking about.

### 1. Traditional programmatic search

A normal program might do:

```python
for tool in tools:
    check_if_matches(tool)
```

This is approximately:

```text
O(n)
```

where `n` = number of tools.

### 2. LLM tool selection

The LLM does **not normally perform an explicit sequential loop**:

```text
tool 1 → check
tool 2 → check
tool 3 → check
...
tool n → check
```

Instead, all supplied tool definitions are represented in the model's input context, and the model uses its neural attention/representations to generate the appropriate tool call.

Therefore, saying:

> "The LLM searches the tools using O(n)"

is an oversimplification.

---

# The important scaling problem

Although it isn't a simple `O(n)` search, giving the LLM more tools still has a cost.

For example:

```text
10 tools
   ↓
User request + 10 tool definitions
   ↓
LLM
```

versus:

```text
10,000 tools
   ↓
User request + 10,000 tool definitions
   ↓
LLM
```

With thousands of tools:

* Context size increases
* Token usage increases
* Cost can increase
* Latency can increase
* Tool selection can become less reliable

Therefore, large systems often **don't give every tool to the LLM**.

---

# Tool Retrieval / Routing

A common architecture is:

```text
                    User request
                         |
                         v
                +----------------+
                | Tool Retriever |
                +----------------+
                         |
                         v
               Top relevant tools
                  (e.g. 10)
                         |
                         v
                     +-----+
                     | LLM |
                     +-----+
                         |
                         v
                  Select tool
                         |
                         v
                  Execute tool
```

For example, suppose there are 10,000 tools.

Instead of:

```text
10,000 tools → LLM
```

we can do:

```text
10,000 tools
      |
      v
Retriever / Vector Search
      |
      v
10 relevant tools
      |
      v
LLM
      |
      v
One selected tool
```

The retriever can use techniques such as:

* Embeddings
* Vector databases
* Semantic search
* Metadata filtering
* Tool categories
* Hierarchical routing

---

# Example

Suppose we have 10,000 tools:

```text
Weather tools
Payment tools
Database tools
Email tools
Calendar tools
GitHub tools
CRM tools
...
```

User asks:

```text
"Send an email to John."
```

A retrieval/router layer might identify:

```text
Relevant tools:

1. send_email
2. create_email_draft
3. search_contacts
4. get_email
5. ...
```

Only these tools are given to the LLM.

The LLM then chooses:

```text
send_email
```

---

# Key distinction

There are two separate problems:

## Problem 1 — Tool selection

> "Which tool should I call?"

The **LLM** can solve this by looking at the available tool descriptions and schemas.

```text
User request
     ↓
LLM
     ↓
Tool selection
```

## Problem 2 — Tool discovery

> "Out of 10,000 tools, which tools should I show the LLM?"

This is often solved using a **retrieval/router layer**.

```text
10,000 tools
     ↓
Retriever
     ↓
Top 10 relevant tools
     ↓
LLM
     ↓
Exact tool
```

---

# Simple Mental Model

Remember this:

```text
LangChain
    |
    +-- Provides tools to the LLM
    |
    +-- Executes tool calls
    |
    +-- Manages the agent/tool-calling loop
             |
             v
           LLM
             |
             +-- Understands user request
             |
             +-- Chooses appropriate tool
             |
             +-- Generates tool arguments
```

And for very large tool collections:

```text
User
 |
 v
Tool Retriever
 |
 |---- Find relevant tools
 |
 v
LLM
 |
 |---- Choose exact tool
 |
 v
Tool Execution
 |
 v
Result
```

## One-line takeaway

> **LangChain doesn't normally linearly search every tool with an `O(n)` loop. The LLM selects among the tools provided to it; when there are too many tools, a retrieval/routing layer can first narrow the tool set.**
