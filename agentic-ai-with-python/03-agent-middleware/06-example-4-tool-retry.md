# Code Example

```
import sys

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import ToolRetryMiddleware
from langchain_core.tools import tool

load_dotenv()

# This line makes the output UTF-8.
sys.stdout.reconfigure(encoding="utf-8")

attempts = {"count": 0}

@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""

    # Increase the attempt counter every time the tool runs.
    attempts["count"] += 1

    print(
        f"   attempt {attempts['count']} "
        f"to reach the order service"
    )
    if attempts["count"] < 3:
       raise RuntimeError("order service unavailable")

    return f"{order_id.upper()}: packed, ships tomorrow"

agent = create_agent(
    model="openai:gpt-4o-mini",

    tools=[order_status],

    system_prompt=(
        "You are a support agent. "
        "Look up the order before answering."
    ),
    middleware=[
        ToolRetryMiddleware(
            max_retries=5,
            initial_delay=0.2,
            backoff_factor=1.5
            )

        ],
)
# this is ToolRetryMiddleware builtin middleware so far in our previous example we create our own custom middleware but now we used builting which is from framework



result = agent.invoke({
    "messages": [
        {
            "role": "user",
            "content": "Where is my order ORD-1002?"
        }
    ]
})


print()
print("Answer:", result["messages"][-1].content)
print("Total attempts:", attempts["count"])

```

# LangChain Agent with Tool Retry Middleware – Code Explanation

This script demonstrates the **built-in `ToolRetryMiddleware`**.  
It automatically retries a tool when it fails (raises an exception), instead of immediately giving up.

---

## 1. Imports & Setup

```markdown

```python
import sys
from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import ToolRetryMiddleware
from langchain_core.tools import tool

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")
```

- Loads environment variables and sets UTF-8 encoding.
- Imports `create_agent` and the built-in `ToolRetryMiddleware`.
- `@tool` turns a normal function into a LangChain tool.

---

## 2. Attempt Counter (for demonstration)

```python
attempts = {"count": 0}
```

- A simple dictionary used to track how many times the tool has been called.
- We use a dictionary (mutable) so the counter can be updated from inside the tool function.

---

## 3. The Tool (Simulates Temporary Failure)

```python
@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""
    # Increase the attempt counter every time the tool runs.
    attempts["count"] += 1
    print(
        f"   attempt {attempts['count']} "
        f"to reach the order service"
    )
    if attempts["count"] < 3:
       raise RuntimeError("order service unavailable")
    return f"{order_id.upper()}: packed, ships tomorrow"
```

### What this tool does:
1. Increments the attempt counter.
2. Prints the current attempt number.
3. **Fails on the first 2 attempts** by raising `RuntimeError`.
4. Succeeds only on the **3rd attempt** (and onwards).

This simulates a real-world situation where a service is temporarily unavailable.

---

## 4. Creating the Agent with ToolRetryMiddleware

```python
agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[order_status],
    system_prompt=(
        "You are a support agent. "
        "Look up the order before answering."
    ),
    middleware=[
        ToolRetryMiddleware(
            max_retries=5,
            initial_delay=0.2,
            backoff_factor=1.5
        )
    ],
)
```

### ToolRetryMiddleware Parameters:

| Parameter         | Value | Meaning                                      |
|-------------------|-------|----------------------------------------------|
| `max_retries`     | 5     | Maximum number of retries after the first failure |
| `initial_delay`   | 0.2   | Wait 0.2 seconds before the first retry      |
| `backoff_factor`  | 1.5   | Each retry waits 1.5× longer than the previous one |

**Retry timing example:**
- Attempt 1 → fails immediately
- Wait 0.2s → Attempt 2
- Wait 0.3s → Attempt 3
- Wait 0.45s → Attempt 4
- ... and so on (up to max_retries)

---

## 5. Running the Agent

```python
result = agent.invoke({
    "messages": [
        {
            "role": "user",
            "content": "Where is my order ORD-1002?"
        }
    ]
})

print()
print("Answer:", result["messages"][-1].content)
print("Total attempts:", attempts["count"])
```

### Expected Output Flow:

```
   attempt 1 to reach the order service
   attempt 2 to reach the order service
   attempt 3 to reach the order service

Answer: ORD-1002: packed, ships tomorrow
Total attempts: 3
```

- The tool fails twice.
- `ToolRetryMiddleware` automatically retries.
- On the 3rd attempt the tool succeeds.
- The agent continues and returns the final answer.

---

## Execution Flow

```
User asks: "Where is my order ORD-1002?"
        ↓
Model decides to call order_status
        ↓
Tool runs → Attempt 1 → raises RuntimeError
        ↓
ToolRetryMiddleware catches the error
        ↓
Waits (initial_delay) → retries
        ↓
Tool runs → Attempt 2 → raises RuntimeError again
        ↓
Waits longer (backoff) → retries
        ↓
Tool runs → Attempt 3 → SUCCESS
        ↓
Model receives the successful result
        ↓
Final answer is returned
```

---

## Key Takeaways

| Concept                      | Explanation                                                                 |
|------------------------------|-----------------------------------------------------------------------------|
| `ToolRetryMiddleware`        | Built-in middleware that automatically retries failed tools                 |
| `max_retries`                | How many times to retry after the first failure                             |
| `initial_delay`              | Delay before the first retry                                                |
| `backoff_factor`             | Multiplier that increases the wait time between retries                     |
| Why useful                   | Handles temporary network issues, rate limits, or flaky external services   |

### Difference from previous examples:
- In earlier examples we wrote **custom middleware**.
- Here we are using a **ready-made middleware** provided by LangChain.
- You can still combine both custom + built-in middleware in the same agent.