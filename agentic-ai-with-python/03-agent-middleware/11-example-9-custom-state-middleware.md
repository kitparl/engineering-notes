Custom state middleware lets you intercept an agent's execution, observe what happened, and update structured state that can influence later steps of the agent.

# Code Example

```
import sys

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt
from langchain.agents.middleware.types import AgentMiddleware, AgentState
from langchain_core.tools import tool

load_dotenv()

sys.stdout.reconfigure(encoding="utf-8")

# {
#    "messages": [....]
#     "tool_uses": 1
#     "refund_started": True
# }

class SupportState(AgentState):
    """The usual messages, plus two keys of our own."""

    tool_uses: int  # how many tools calls have happened so far
    refund_started: bool  # True once start_refund has been called

@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""
    return f"{order_id.upper()}: packed, ships tomorrow"



@tool
def delivery_estimate(pin_code: str) -> str:
    """Estimate delivery days for an Indian pin code."""
    return "5 days"


@tool
def start_refund(order_id: str, reason: str) -> str:
    """Start a refund for an order."""
    return f"Refund started for {order_id.upper()}, reason: {reason}"

#Middleware

class TrackWork(AgentMiddleware):
    """Counts tool calls and notices when a refund has been started."""

    state_schema = SupportState  # tell langchain that my state is like SupportState 

    # model after -->  tool execution

    def after_model(self, state, runtime):
        last = state["messages"][-1]
        calls = getattr(last, "tool_calls", []) or []
        if not calls:
            return None

        update = {"tool_uses": state.get("tool_uses", 0) + len(calls)}
        if any(call["name"] == "start_refund" for call in calls):
            update["refund_started"] = True

        print(f"   [state] tool_uses={update['tool_uses']} refund_started={update.get('refund_started', state.get('refund_started', False))}")
        return update

@dynamic_prompt
def prompt_with_budget(request):
    used = request.state.get("tool_uses", 0)

    prompt = "You are a support agent. Look things up with the tools, never guess."
    if used >= 3:
        prompt += " You have already used three tools, answer now with what you have."
    return prompt

agent = create_agent(
    model="gpt-4o-mini",
    tools=[order_status, delivery_estimate, start_refund],
    middleware=[TrackWork(), prompt_with_budget],
   
)

result = agent.invoke(
    {
    "messages": [{
        "role": "user",
        "content": "Where is ORD-1002, when does it reach pin 560034, and refund it because it is too late.",
    }],
    "tool_uses": 0,
    "refund_started": False,
}

)
print()
print("Answer:", result["messages"][-1].content)
print()
print("tool_uses      :", result["tool_uses"])
print("refund_started :", result["refund_started"])
if result["refund_started"]:
    print("Refund has been started,notify the finance team and support team")
```

# LangChain Agent with Custom State & Tracking Middleware – Code Explanation

This script shows how to extend the agent’s **state** with custom fields and use middleware to track tool usage and business events (like starting a refund).

---

## 1. Imports & Setup


```python
import sys
from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt
from langchain.agents.middleware.types import AgentMiddleware, AgentState
from langchain_core.tools import tool

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")
```

- `AgentState` → Base class for custom agent state.
- `AgentMiddleware` → Base class for writing class-based middleware.
- `dynamic_prompt` → Middleware that can change the system prompt based on current state.

---

## 2. Custom State Definition

```python
class SupportState(AgentState):
    """The usual messages, plus two keys of our own."""
    tool_uses: int          # how many tool calls have happened so far
    refund_started: bool    # True once start_refund has been called
```

### What this does:
- Extends the default agent state.
- Adds two extra fields:
  - `tool_uses` → Counter of how many tools have been called.
  - `refund_started` → Flag that becomes `True` when a refund is initiated.

---

## 3. Tools

```python
@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""
    return f"{order_id.upper()}: packed, ships tomorrow"

@tool
def delivery_estimate(pin_code: str) -> str:
    """Estimate delivery days for an Indian pin code."""
    return "5 days"

@tool
def start_refund(order_id: str, reason: str) -> str:
    """Start a refund for an order."""
    return f"Refund started for {order_id.upper()}, reason: {reason}"
```

Three tools are available:
- Check order status
- Get delivery estimate
- Start a refund

---

## 4. Custom Middleware – TrackWork

```python
class TrackWork(AgentMiddleware):
    """Counts tool calls and notices when a refund has been started."""
    state_schema = SupportState   # tell LangChain to use our custom state

    def after_model(self, state, runtime):
        last = state["messages"][-1]
        calls = getattr(last, "tool_calls", []) or []

        if not calls:
            return None

        update = {
            "tool_uses": state.get("tool_uses", 0) + len(calls)
        }

        if any(call["name"] == "start_refund" for call in calls):
            update["refund_started"] = True

        print(
            f"   [state] tool_uses={update['tool_uses']} "
            f"refund_started={update.get('refund_started', state.get('refund_started', False))}"
        )
        return update
```

### How it works:
- Runs **after every model response**.
- Looks at the latest message to see if the model requested any tools.
- Updates the state:
  - Increments `tool_uses` by the number of tool calls.
  - Sets `refund_started = True` if `start_refund` was called.
- Returning a dictionary **merges** those values into the agent state.

---

## 5. Dynamic Prompt Based on State

```python
@dynamic_prompt
def prompt_with_budget(request):
    used = request.state.get("tool_uses", 0)
    prompt = "You are a support agent. Look things up with the tools, never guess."

    if used >= 3:
        prompt += " You have already used three tools, answer now with what you have."

    return prompt
```

### Behavior:
- Reads the current `tool_uses` value from the state.
- If the agent has already used **3 or more tools**, it adds an instruction telling the model to stop calling tools and answer with the information it has.

This acts as a simple **tool budget / cost control** mechanism.

---

## 6. Creating the Agent

```python
agent = create_agent(
    model="gpt-4o-mini",
    tools=[order_status, delivery_estimate, start_refund],
    middleware=[TrackWork(), prompt_with_budget],
)
```

- Uses the custom `TrackWork` middleware (class-based).
- Also uses the `prompt_with_budget` dynamic prompt.
- The agent now carries the extra state fields (`tool_uses` and `refund_started`).

---

## 7. Invoking the Agent with Initial State

```python
result = agent.invoke(
    {
        "messages": [{
            "role": "user",
            "content": "Where is ORD-1002, when does it reach pin 560034, and refund it because it is too late.",
        }],
        "tool_uses": 0,
        "refund_started": False,
    }
)
```

- Starts with `tool_uses = 0` and `refund_started = False`.
- The user asks for:
  1. Order status
  2. Delivery estimate
  3. A refund

The agent will likely call all three tools.

---

## 8. Final Output & Business Logic

```python
print("Answer:", result["messages"][-1].content)
print("tool_uses      :", result["tool_uses"])
print("refund_started :", result["refund_started"])

if result["refund_started"]:
    print("Refund has been started, notify the finance team and support team")
```

After the run you can:
- See how many tools were used.
- Check whether a refund was started.
- Trigger side effects (notify finance/support team) based on the final state.

---

## Execution Flow Summary

```
User question
    ↓
Model decides to call tools
    ↓
[after_model] → TrackWork updates tool_uses & refund_started
    ↓
Tools execute
    ↓
Model is called again (with updated state)
    ↓
[dynamic_prompt] may tighten the prompt if tool_uses ≥ 3
    ↓
Final answer + final state returned
```

---

## Key Takeaways

| Concept                     | Explanation                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| Custom `AgentState`         | Lets you add your own fields to the agent’s memory                          |
| Class-based Middleware      | More powerful than function middleware; can define multiple hooks           |
| `after_model` hook          | Perfect place to inspect tool calls and update state                        |
| `state_schema`              | Tells LangChain which state structure this middleware expects               |
| Dynamic prompt + state      | You can change instructions based on how much work has already been done    |
| Business logic after run    | You can react to final state values (e.g. notify teams when refund starts)  |

This pattern is very useful for:
- Tracking cost / tool usage
- Enforcing tool budgets
- Detecting important business events
- Building more controllable production agents


| Component        | Job                                               |
| ---------------- | ------------------------------------------------- |
| `SupportState`   | Defines what information the agent remembers      |
| `TrackWork`      | Observes events and modifies that state           |
| `dynamic_prompt` | Reads state and changes instructions to the model |
| Tools            | Actually perform business operations              |
| LLM              | Decides what to do                                |
