# Code Example

```

import sys

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware

load_dotenv()

# This line makes the output UTF-8.
sys.stdout.reconfigure(encoding="utf-8")

agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[],
    system_prompt="You are a support agent for an online store.",
    middleware=[
        SummarizationMiddleware(
            model="openai:gpt-4o-mini",
            trigger=("messages", 6),# trigger the sumamrization after 5 messages
            keep=("messages", 2)  # after summarizing keep latest 2 messsages and before older that summarise

        )
     
    ],
)
conversation = [
    "Hi, my name is Asha and my customer id is C-9087.",

    "I ordered a mechanical keyboard last week, order ORD-1002.",

    "My pin code is 560034, in Bengaluru.",

    "I also have an older order, ORD-1001, a wireless mouse.",

    "The keyboard is a gift, so the date matters to me.",

    "Tell me everything you remember about me and my orders.",
]

messages = []

for question in conversation:

    messages.append({"role": "user", "content": question})

    result = agent.invoke({"messages": messages})

    messages = result["messages"]

    print("Customer:", question)

    print(
        "Agent   :",
        result["messages"][-1].content[:160]
    )


    # how many messages currently remain in history
    print("\n========== COMPLETE MESSAGE LIST ==========")

    for i, message in enumerate(messages):

        print(f"\n[{i}] {type(message).__name__}")
        print("Content:", message.content)

    print("\n============================================")

    print(
        f"   history now holds {len(messages)} messages"
    )

    print("-" * 70)
```

# LangChain Agent with Summarization Middleware – Code Explanation

This script demonstrates the **built-in `SummarizationMiddleware`**.  
It automatically summarizes older messages when the conversation becomes too long, so the model doesn’t run out of context window.

---

## 1. Imports & Setup

```python
import sys
from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")
```

- Loads environment variables and sets UTF-8 encoding.
- Imports `create_agent` and the built-in `SummarizationMiddleware`.
- No tools are used in this example (focus is only on conversation history management).

---

## 2. Creating the Agent with SummarizationMiddleware

```python
agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[],
    system_prompt="You are a support agent for an online store.",
    middleware=[
        SummarizationMiddleware(
            model="openai:gpt-4o-mini",
            trigger=("messages", 6),   # trigger summarization after 6 messages
            keep=("messages", 2)       # after summarizing, keep only the latest 2 messages
        )
    ],
)
```

### SummarizationMiddleware Parameters:

| Parameter   | Value              | Meaning                                                                 |
|-------------|--------------------|-------------------------------------------------------------------------|
| `model`     | `"openai:gpt-4o-mini"` | The model used to generate the summary                               |
| `trigger`   | `("messages", 6)`  | When the conversation reaches **6 messages**, summarization starts     |
| `keep`      | `("messages", 2)`  | After summarizing, keep only the **latest 2 messages** + the summary   |

### How it works conceptually:

1. Conversation grows normally.
2. When total messages ≥ 6 → middleware is triggered.
3. Older messages are summarized into a short summary.
4. The summary replaces the old messages.
5. Only the summary + the latest 2 messages remain in history.

---

## 3. Simulated Conversation

```python
conversation = [
    "Hi, my name is Asha and my customer id is C-9087.",
    "I ordered a mechanical keyboard last week, order ORD-1002.",
    "My pin code is 560034, in Bengaluru.",
    "I also have an older order, ORD-1001, a wireless mouse.",
    "The keyboard is a gift, so the date matters to me.",
    "Tell me everything you remember about me and my orders.",
]
```

Six customer messages are prepared.  
The last one asks the agent to recall everything — this is where we can observe the effect of summarization.

---

## 4. Running the Conversation Loop

```python
messages = []
for question in conversation:
    messages.append({"role": "user", "content": question})
    result = agent.invoke({"messages": messages})
    messages = result["messages"]          # update history with full result

    print("Customer:", question)
    print(
        "Agent   :",
        result["messages"][-1].content[:160]
    )

    # Debug: show the complete current message list
    print("\n========== COMPLETE MESSAGE LIST ==========")
    for i, message in enumerate(messages):
        print(f"\n[{i}] {type(message).__name__}")
        print("Content:", message.content)
    print("\n============================================")
    print(
        f"   history now holds {len(messages)} messages"
    )
    print("-" * 70)
```

### What happens in each loop:

1. Add the new user message to `messages`.
2. Call `agent.invoke()`.
3. The middleware may summarize older messages if the trigger condition is met.
4. Replace `messages` with the updated history returned by the agent.
5. Print the agent’s reply and the full current message list (for learning/debugging).

---

## Expected Behavior

| Step | Messages before call | What happens                                      | Messages after call |
|------|----------------------|---------------------------------------------------|---------------------|
| 1    | 1                    | Normal reply                                      | ~2                  |
| 2    | 3                    | Normal reply                                      | ~4                  |
| 3    | 5                    | Normal reply                                      | ~6                  |
| 4    | 7                    | **Trigger reached** → older messages summarized   | Summary + last 2    |
| 5    | ...                  | Continues with compact history                    | Summary + latest    |
| 6    | ...                  | Agent can still answer using the summary          | Compact history     |

> Note: Exact message counts can vary slightly because each model reply also adds messages to the history.

---

## Why Summarization is Useful

| Problem without summarization          | Solution with SummarizationMiddleware      |
|----------------------------------------|--------------------------------------------|
| Context window fills up quickly        | Older parts are compressed into a summary  |
| High token cost on long chats          | Token usage stays under control            |
| Model forgets early details            | Important facts are preserved in the summary |
| Hard to run long support conversations | Conversation can continue for many turns   |

---

## Key Takeaways

| Concept                     | Explanation                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| `SummarizationMiddleware`   | Built-in middleware that automatically summarizes old messages              |
| `trigger=("messages", 6)`   | Start summarizing when there are 6 or more messages                         |
| `keep=("messages", 2)`      | After summarizing, keep only the latest 2 messages + the summary            |
| `model`                     | The LLM used to create the summary (can be the same or a cheaper/faster one)|
| Benefit                     | Keeps long conversations manageable without losing important context        |

This pattern is especially useful for:
- Customer support bots
- Long multi-turn chats
- Agents that need to remember user details over many messages
- Controlling token costs in production
