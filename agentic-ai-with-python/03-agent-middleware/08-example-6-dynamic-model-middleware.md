# Code Example

```
import sys

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import wrap_model_call
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool

load_dotenv()

# This line makes the output UTF-8.
sys.stdout.reconfigure(encoding="utf-8")


CHEAP = init_chat_model("openai:gpt-4o-mini")
STRONG = init_chat_model("openai:gpt-4o")

HARD_WORDS = (
    "hard",
    "difficult",
    "complex",
    "complicated",
    "challenging",
    "compare",
    "explain why",
    "analyze",
    "risk",
    "legal"
)

@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""

    return f"{order_id.upper()}: packed, ships tomorrow"


def latest_question(messages):
    """
    Get the customer's most recent message.

    We don't simply use messages[-1] because the last message
    might be a tool result or another agent message.

    We specifically want the latest human message.
    """

    for message in reversed(messages):

        if type(message).__name__ == "HumanMessage":
            return str(message.content).lower()

    return ""

@wrap_model_call
def route_model(request, handler):

    # Get the latest customer question.
    question = latest_question(request.messages)

    hard = (
        any(word in question for word in HARD_WORDS)
        or len(question) > 180
    )
    chosen = STRONG if hard else CHEAP

    print(f"   routed to {chosen.model_name}")

    return handler(
        request.override(model=chosen)
    )

agent = create_agent(
    model=CHEAP,
    tools=[order_status],
    system_prompt="You are a support agent for an online store.",
    middleware=[route_model],
)

def ask(question):

    print("Q:", question[:70])

    result = agent.invoke({
        "messages": [
            {
                "role": "user",
                "content": question
            }
        ]
    })

    print(
        "A:",
        result["messages"][-1].content[:200]
    )

    print("-" * 70)

ask("Where is my order ORD-1002?")
ask(
    "Compare buying a mechanical keyboard now against "
    "waiting for the sale, and explain why."
)
```

# LangChain Agent with Dynamic Model Routing – Code Explanation

This script shows how to **dynamically choose the model** for each request using the `@wrap_model_call` middleware.  
Simple questions go to a cheap/fast model (`gpt-4o-mini`), while hard or complex questions are routed to a stronger model (`gpt-4o`).

## 1. Imports & Setup

```python
import sys
from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import wrap_model_call
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")
```

- Loads environment variables and sets UTF-8 encoding.
- `init_chat_model` is used to create model instances that can be swapped at runtime.
- `@wrap_model_call` lets us intercept and modify the model call.

---

## 2. Define Two Models

```python
CHEAP = init_chat_model("openai:gpt-4o-mini")
STRONG = init_chat_model("openai:gpt-4o")
```

| Model   | Name            | Use Case                          |
|---------|-----------------|-----------------------------------|
| `CHEAP` | `gpt-4o-mini`   | Simple, short, routine questions  |
| `STRONG`| `gpt-4o`        | Complex, analytical, long questions |

---

## 3. Hard Question Keywords

```python
HARD_WORDS = (
    "hard",
    "difficult",
    "complex",
    "complicated",
    "challenging",
    "compare",
    "explain why",
    "analyze",
    "risk",
    "legal"
)
```

If the user’s question contains any of these words (or is longer than 180 characters), it is treated as a **hard** question and routed to the stronger model.

---

## 4. Tool Definition

```python
@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""
    return f"{order_id.upper()}: packed, ships tomorrow"
```

A simple tool used for order-related questions.

---

## 5. Helper – Get Latest Human Message

```python
def latest_question(messages):
    """
    Get the customer's most recent message.
    We don't simply use messages[-1] because the last message
    might be a tool result or another agent message.
    We specifically want the latest human message.
    """
    for message in reversed(messages):
        if type(message).__name__ == "HumanMessage":
            return str(message.content).lower()
    return ""
```

Safely extracts the latest message written by the human (customer).

---

## 6. Model Routing Middleware

```python
@wrap_model_call
def route_model(request, handler):
    # Get the latest customer question.
    question = latest_question(request.messages)

    hard = (
        any(word in question for word in HARD_WORDS)
        or len(question) > 180
    )

    chosen = STRONG if hard else CHEAP
    print(f"   routed to {chosen.model_name}")

    return handler(
        request.override(model=chosen)
    )
```

### How it works:

1. Looks at the latest human question.
2. Decides if the question is **hard**:
   - Contains any word from `HARD_WORDS`, **or**
   - Longer than 180 characters.
3. Chooses the appropriate model:
   - Hard → `STRONG` (`gpt-4o`)
   - Easy → `CHEAP` (`gpt-4o-mini`)
4. Overrides the model for **this call only** using `request.override(model=chosen)`.
5. Calls the model with `handler(...)`.

---

## 7. Creating the Agent

```python
agent = create_agent(
    model=CHEAP,                    # default model
    tools=[order_status],
    system_prompt="You are a support agent for an online store.",
    middleware=[route_model],
)
```

- Default model is set to `CHEAP`.
- The middleware can override it on every call based on the question difficulty.

---

## 8. Helper Function & Test Questions

```python
def ask(question):
    print("Q:", question[:70])
    result = agent.invoke({
        "messages": [
            {
                "role": "user",
                "content": question
            }
        ]
    })
    print(
        "A:",
        result["messages"][-1].content[:200]
    )
    print("-" * 70)

ask("Where is my order ORD-1002?")
ask(
    "Compare buying a mechanical keyboard now against "
    "waiting for the sale, and explain why."
)
```

### Expected Routing:

| Question                                      | Detected as | Routed to          |
|-----------------------------------------------|-------------|--------------------|
| "Where is my order ORD-1002?"                 | Easy        | `gpt-4o-mini`      |
| "Compare buying a mechanical keyboard..."     | Hard        | `gpt-4o`           |

---

## Execution Flow

```
User question arrives
        ↓
[wrap_model_call] → route_model
        ↓
Extract latest HumanMessage
        ↓
Is it hard? (keywords or length > 180)
   ├── Yes → override model = STRONG (gpt-4o)
   └── No  → override model = CHEAP  (gpt-4o-mini)
        ↓
Model is called with the chosen model
        ↓
Final answer is returned
```

---

## Key Takeaways

| Concept                        | Explanation                                                                 |
|--------------------------------|-----------------------------------------------------------------------------|
| `@wrap_model_call`             | Middleware that wraps every model call                                      |
| `request.override(model=...)`  | Temporarily changes the model for the current call                         |
| Dynamic model routing          | Use cheap model for simple tasks, strong model for complex ones             |
| Cost & Latency benefit         | Saves money and improves speed on easy questions                            |
| Quality benefit                | Uses the best model only when needed                                        |

### Why this pattern is useful:
- Reduces cost (most questions are simple)
- Improves response quality on hard questions
- Easy to extend with more routing rules (e.g., by user plan, language, topic, etc.)
