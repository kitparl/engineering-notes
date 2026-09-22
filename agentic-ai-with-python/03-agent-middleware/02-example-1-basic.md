# Code Example

```
import sys

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import (
    after_model,
    before_model,
    wrap_model_call,
    wrap_tool_call,
)

from langchain_core.tools import tool

load_dotenv()

sys.stdout.reconfigure(encoding="utf-8")

@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""

    return f"{order_id.upper()}: packed, ships tomorrow"


# ---------------------------------------------------------
# BEFORE MODEL
# ---------------------------------------------------------
# runs before the model is called

@before_model
def show_before(state, runtime):

    print(
        f"[before_model] "
        f"{len(state['messages'])} message(s) going to the model"
    )

    # None means:
    # "I am not changing the state."
    return None


# ---------------------------------------------------------
# WRAP MODEL CALL
# ---------------------------------------------------------
# around the model call 

@wrap_model_call
def time_the_model(request, handler):

    # We can inspect the request before sending it to
    # the model.
    print(
        f"[wrap_model_call] "
        f"tools offered: {[t.name for t in request.tools]}"
    )
    response = handler(request)
    print("[wrap_model_call] model has replied")

    # Return the model response so the agent can continue.
    return response



# ---------------------------------------------------------
# AFTER MODEL
# ---------------------------------------------------------
#runs immediately after the model responds

@after_model
def show_after(state, runtime):

    # Get the latest message produced by the model.
    last = state["messages"][-1]

    # Check whether the model requested any tools.
    asked = [
        call["name"]
        for call in getattr(last, "tool_calls", []) or []
    ]

    print(
        f"[after_model] asked for: "
        f"{asked or 'nothing, this is the final answer'}"
    )

    # We are only observing here.
    return None



# ---------------------------------------------------------
# WRAP TOOL CALL
# ---------------------------------------------------------

# # This middleware sits AROUND every tool execution.

@wrap_tool_call
def show_tool(request, handler):

    # We can see which tool the agent is about to execute
    # and what arguments are being passed to it.
    print(
        f"[wrap_tool_call] "
        f"running {request.tool_call['name']} "
        f"with {request.tool_call['args']}"
    )

    # Continue the execution and actually run the tool.
    result = handler(request)

    # We are back after the tool has completed.
    print("[wrap_tool_call] tool finished")

    # Return the tool result so the agent can continue.
    return result



agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[order_status],
    system_prompt="You are a support agent.",
    middleware=[
        show_before,
        time_the_model,
        show_after,
        show_tool,
    ],

)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Where is my order ORD-1002?"}]
})
print()
print("Answer:", result["messages"][-1].content)
```

# LangChain Agent with Middleware – Code Explanation

```markdown

This script demonstrates how to create a **LangChain agent** with custom **middleware** that hooks into different stages of the agent’s execution lifecycle (before the model, around the model call, after the model, and around tool calls).

## 1. Imports & Setup

---


```python
import sys
from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import (
    after_model,
    before_model,
    wrap_model_call,
    wrap_tool_call,
)
from langchain_core.tools import tool

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")
```

- `load_dotenv()` → Loads environment variables (usually contains `OPENAI_API_KEY`).
- `sys.stdout.reconfigure(encoding="utf-8")` → Ensures proper Unicode output.
- Imports the agent factory (`create_agent`) and the four middleware decorators.
- `@tool` decorator is used to turn a normal Python function into a LangChain tool.

---

## 2. Defining a Tool

```python
@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""
    return f"{order_id.upper()}: packed, ships tomorrow"
```

- This is a simple tool the agent can call.
- When the agent decides it needs order information, it will call this function with an `order_id`.
- The function returns a hardcoded status string.

---

## 3. Middleware Functions

Middleware lets you intercept and observe (or modify) the agent’s behavior at specific points.

### 3.1 `before_model` – Runs **before** the model is called

```python
@before_model
def show_before(state, runtime):
    print(
        f"[before_model] "
        f"{len(state['messages'])} message(s) going to the model"
    )
    return None   # None = “do not change the state”
```

- Executes **before** every LLM call.
- Receives the current agent `state` (contains the message history).
- Here it only prints how many messages will be sent to the model.
- Returning `None` means “don’t modify anything”.

### 3.2 `wrap_model_call` – Wraps **around** the model call

```python
@wrap_model_call
def time_the_model(request, handler):
    print(
        f"[wrap_model_call] "
        f"tools offered: {[t.name for t in request.tools]}"
    )
    response = handler(request)          # ← actual model call happens here
    print("[wrap_model_call] model has replied")
    return response
```

- Acts like a decorator around the model invocation.
- You can inspect the `request` (which tools are available, messages, etc.) **before** calling the model.
- `handler(request)` is the real model call.
- After the model responds, you can inspect or modify the response before returning it.

### 3.3 `after_model` – Runs **immediately after** the model responds

```python
@after_model
def show_after(state, runtime):
    last = state["messages"][-1]
    asked = [
        call["name"]
        for call in getattr(last, "tool_calls", []) or []
    ]
    print(
        f"[after_model] asked for: "
        f"{asked or 'nothing, this is the final answer'}"
    )
    return None
```

- Runs right after the model produces a response.
- Checks whether the model requested any tool calls.
- Prints either the list of tools the model wants to call, or “final answer” if no tools were requested.

### 3.4 `wrap_tool_call` – Wraps **around** every tool execution

```python
@wrap_tool_call
def show_tool(request, handler):
    print(
        f"[wrap_tool_call] "
        f"running {request.tool_call['name']} "
        f"with {request.tool_call['args']}"
    )
    result = handler(request)            # ← actual tool runs here
    print("[wrap_tool_call] tool finished")
    return result
```

- Intercepts every tool call the agent makes.
- You can see the tool name and arguments **before** the tool runs.
- `handler(request)` executes the real tool.
- After the tool finishes, you can inspect or modify the result.

---

## 4. Creating the Agent

```python
agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[order_status],
    system_prompt="You are a support agent.",
    middleware=[
        show_before,
        time_the_model,
        show_after,
        show_tool,
    ],
)
```

- Uses `gpt-4o-mini` as the LLM.
- Registers the `order_status` tool.
- Sets a simple system prompt.
- Attaches all four middleware functions in the order they will be applied.

---

## 5. Running the Agent

```python
result = agent.invoke({
    "messages": [{"role": "user", "content": "Where is my order ORD-1002?"}]
})

print()
print("Answer:", result["messages"][-1].content)
```

- Sends a user question about order `ORD-1002`.
- The agent will:
  1. Go through `before_model`
  2. Go through `wrap_model_call` → model decides to call the tool
  3. Go through `after_model` (sees tool call)
  4. Go through `wrap_tool_call` → tool executes
  5. Model is called again with the tool result
  6. Finally returns the answer

---

## Execution Flow Summary

```
User message
    ↓
[before_model]          ← observe messages
    ↓
[wrap_model_call]       ← before model
    ↓
Model decides to call tool
    ↓
[after_model]           ← sees tool call request
    ↓
[wrap_tool_call]        ← before tool
    ↓
Tool runs (order_status)
    ↓
[wrap_tool_call]        ← after tool
    ↓
[before_model]          ← again (second model call)
    ↓
[wrap_model_call]       ← second model call
    ↓
Model produces final answer
    ↓
[after_model]           ← “nothing, this is the final answer”
    ↓
Final answer printed
```

---

## Key Takeaways

| Middleware          | When it runs                  | Purpose                              |
|---------------------|-------------------------------|--------------------------------------|
| `before_model`      | Before every LLM call         | Inspect / modify state               |
| `wrap_model_call`   | Around the LLM call           | Logging, timing, request/response modification |
| `after_model`       | Right after LLM responds      | Inspect tool calls or final answer   |
| `wrap_tool_call`    | Around every tool execution   | Logging, validation, result modification |

This pattern is extremely useful for debugging, logging, rate-limiting, authentication checks, or adding custom logic without changing the core agent code.
```