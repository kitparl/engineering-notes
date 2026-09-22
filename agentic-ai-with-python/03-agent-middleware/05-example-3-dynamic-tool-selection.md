# Example Code

```
import sys

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import wrap_model_call
from langchain_core.tools import tool

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")

@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""

    return f"{order_id.upper()}: packed, ships tomorrow"


@tool
def delivery_estimate(pin_code: str) -> str:
    """Estimate delivery days for an Indian pin code."""

    return "2 days" if pin_code.startswith("4") else "5 days"


@tool
def start_refund(order_id: str, reason: str) -> str:
    """Start a refund for an order."""

    return f"Refund started for {order_id.upper()}, reason: {reason}"


@tool
def cancel_order(order_id: str) -> str:
    """Cancel an order that has not shipped yet."""

    return f"{order_id.upper()} cancelled"

DELIVERY_TOOLS = [
    order_status,
    delivery_estimate
]

MONEY_TOOLS = [
    order_status,
    start_refund,
    cancel_order
]

MONEY_WORDS = (
    "refund",
    "cancel",
    "money",
    "return"
)

def latest_question(messages):
    """
    Get the customer's most recent message.

    We don't simply use messages[-1].

    During an agent run, the last message could be:
        - a tool result
        - the model's response
        - another internal message

    We specifically want the latest message written
    by the human/customer.
    """

    for message in reversed(messages):

        if type(message).__name__ == "HumanMessage":
            return str(message.content).lower()

    return ""


# dynamic tool selection middle ware --> @wrap_model_call
@wrap_model_call
def pick_tools(request, handler):

    # Get the customer's latest question.
    question = latest_question(request.messages)

    # -----------------------------------------------------
    # CHOOSE TOOLS BASED ON THE QUESTION
    # -----------------------------------------------------

    if any(word in question for word in MONEY_WORDS):

        # Money/refund-related question.
        #
        # Give the model only the money-related tools
        # for THIS model call.
        request = request.override(
            tools=MONEY_TOOLS
        )

    else:

        # Normal delivery/order question.
        #
        # Give the model only the delivery-related tools.
        request = request.override(
            tools=DELIVERY_TOOLS
        )
    print(
        "   offered:",
        [t.name for t in request.tools]
    )
    return handler(request)



agent = create_agent(
    model="gpt-4o-mini",

    tools=[
        order_status,
        delivery_estimate,
        start_refund,
        cancel_order
           ],
    middleware=[pick_tools],
    system_prompt="You are a support agent. Use the tools, never guess.",
)

# helper funcation to ask question

def ask(question):

    print("Q:", question)

    result = agent.invoke({
        "messages": [
            {
                "role": "user",
                "content": question
            }
        ]
    })

    print("A:", result["messages"][-1].content)
    print("-" * 70)



ask("Where is my order ORD-1002?")
ask("I want a refund for ORD-1002, it is too late for me.")
ask("what is the status of ORD-1002 and initiate refund if it is not shipped today")
```

# LangChain Agent with Dynamic Tool Selection – Code Explanation


This script demonstrates **dynamic tool selection** using the `@wrap_model_call` middleware.  
Instead of always giving the model *all* tools, the agent inspects the user’s latest question and offers only the relevant tools for that turn.

```markdown

---

## 1. Imports & Setup

```python
import sys
from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import wrap_model_call
from langchain_core.tools import tool

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")
```

- Loads environment variables and sets UTF-8 encoding.
- Imports `create_agent` and the `@wrap_model_call` middleware.
- `@tool` turns normal functions into LangChain tools.

---

## 2. Defining the Tools

```python
@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""
    return f"{order_id.upper()}: packed, ships tomorrow"

@tool
def delivery_estimate(pin_code: str) -> str:
    """Estimate delivery days for an Indian pin code."""
    return "2 days" if pin_code.startswith("4") else "5 days"

@tool
def start_refund(order_id: str, reason: str) -> str:
    """Start a refund for an order."""
    return f"Refund started for {order_id.upper()}, reason: {reason}"

@tool
def cancel_order(order_id: str) -> str:
    """Cancel an order that has not shipped yet."""
    return f"{order_id.upper()} cancelled"
```

Four tools are defined:

| Tool               | Purpose                          |
|--------------------|----------------------------------|
| `order_status`     | Check order status               |
| `delivery_estimate`| Estimate delivery time by pincode|
| `start_refund`     | Start a refund                   |
| `cancel_order`     | Cancel an order                  |

---

## 3. Tool Groups

```python
DELIVERY_TOOLS = [
    order_status,
    delivery_estimate
]

MONEY_TOOLS = [
    order_status,
    start_refund,
    cancel_order
]

MONEY_WORDS = (
    "refund",
    "cancel",
    "money",
    "return"
)
```

- **DELIVERY_TOOLS** → Used for normal order / delivery questions.
- **MONEY_TOOLS** → Used when the customer talks about refunds, cancellation, money, or returns.
- `order_status` is available in both groups (common need).

---

## 4. Helper Function – Get Latest Human Message

```python
def latest_question(messages):
    """
    Get the customer's most recent message.
    We don't simply use messages[-1].
    During an agent run, the last message could be:
        - a tool result
        - the model's response
        - another internal message
    We specifically want the latest message written
    by the human/customer.
    """
    for message in reversed(messages):
        if type(message).__name__ == "HumanMessage":
            return str(message.content).lower()
    return ""
```

### Why this is needed:
During an agent loop the message list contains:
- Human messages
- AI messages
- Tool results

We only care about the **latest human question**, so we walk backwards until we find a `HumanMessage`.

---

## 5. Dynamic Tool Selection Middleware

```python
@wrap_model_call
def pick_tools(request, handler):
    # Get the customer's latest question.
    question = latest_question(request.messages)

    # -----------------------------------------------------
    # CHOOSE TOOLS BASED ON THE QUESTION
    # -----------------------------------------------------
    if any(word in question for word in MONEY_WORDS):
        # Money/refund-related question.
        request = request.override(
            tools=MONEY_TOOLS
        )
    else:
        # Normal delivery/order question.
        request = request.override(
            tools=DELIVERY_TOOLS
        )

    print(
        "   offered:",
        [t.name for t in request.tools]
    )
    return handler(request)
```

### How it works:

1. Extracts the latest customer question.
2. Checks if any money-related keyword (`refund`, `cancel`, `money`, `return`) appears.
3. **Overrides** the tools for *this model call only*:
   - Money keywords found → give `MONEY_TOOLS`
   - Otherwise → give `DELIVERY_TOOLS`
4. Prints which tools are being offered (for debugging).
5. Calls `handler(request)` → the actual model call happens with the filtered tools.

> Important: `request.override(tools=...)` only affects the **current** model call.  
> It does not permanently change the agent’s tool list.

---

## 6. Creating the Agent

```python
agent = create_agent(
    model="gpt-4o-mini",
    tools=[
        order_status,
        delivery_estimate,
        start_refund,
        cancel_order
    ],
    middleware=[pick_tools],
    system_prompt="You are a support agent. Use the tools, never guess.",
)
```

- All four tools are registered with the agent (so they are available).
- The middleware decides **which subset** the model is allowed to see on each turn.
- System prompt instructs the model to always use tools and never guess.

---

## 7. Helper Function to Ask Questions

```python
def ask(question):
    print("Q:", question)
    result = agent.invoke({
        "messages": [
            {
                "role": "user",
                "content": question
            }
        ]
    })
    print("A:", result["messages"][-1].content)
    print("-" * 70)
```

Simple helper that:
1. Prints the question
2. Invokes the agent
3. Prints the final answer
4. Adds a separator line

---

## 8. Test Questions

```python
ask("Where is my order ORD-1002?")
ask("I want a refund for ORD-1002, it is too late for me.")
ask("what is the status of ORD-1002 and initiate refund if it is not shipped today")
```

### Expected Behavior:

| Question | Detected Type | Tools Offered |
|----------|---------------|---------------|
| "Where is my order ORD-1002?" | Delivery | `order_status`, `delivery_estimate` |
| "I want a refund for ORD-1002..." | Money | `order_status`, `start_refund`, `cancel_order` |
| "status of ORD-1002 and initiate refund..." | Money | `order_status`, `start_refund`, `cancel_order` |

---

## Execution Flow

```
User question arrives
        ↓
[wrap_model_call] → pick_tools
        ↓
Look at latest HumanMessage
        ↓
Does it contain money keywords?
   ├── Yes → override tools = MONEY_TOOLS
   └── No  → override tools = DELIVERY_TOOLS
        ↓
Model is called with only the selected tools
        ↓
Model may call one or more tools
        ↓
Final answer is returned
```

---

## Key Takeaways

| Concept                    | Explanation                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| `@wrap_model_call`         | Middleware that wraps every model call                                      |
| `request.override(tools=)` | Temporarily changes which tools the model can see for this call             |
| Dynamic tool selection     | Gives the model only relevant tools → reduces confusion & token usage       |
| `latest_question()`        | Safely extracts the latest human message from the conversation history      |

### Why this pattern is useful:
- Prevents the model from calling irrelevant tools
- Reduces prompt size (fewer tool descriptions)
- Improves reliability for multi-domain agents (support, sales, billing, etc.)
- Easy to extend with more categories (e.g. `TECH_SUPPORT_TOOLS`, `BILLING_TOOLS`)
```
