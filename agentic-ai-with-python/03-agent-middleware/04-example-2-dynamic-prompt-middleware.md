If many users are asking something to the model. I want to give dynamic prompt.

# Code Example

```
import sys
from dataclasses import dataclass

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt
from langchain_core.tools import tool

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")

@dataclass
class Customer:
    """What we know about the person asking, before the model sees anything."""

    name: str
    plan: str
    language: str


@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""

    return f"{order_id.upper()}: packed, ships tomorrow"

@dynamic_prompt  # moddleware hook# before the model --> create or chnage the system prompt
def support_prompt(request):

    # this will give us that Customer object we passed in when we called agent.invoke()
    # Customer(
    #     name="Alice",
    #     plan="premium",
    #     language="English",
    # )
    customer = request.runtime.context

    lines = [
        f"You are a support agent for an online store. The customer is {customer.name}.",
        "Look up the order before answering, never guess.",
        "Write like a chat reply, no email signature.",
    ]

    if customer.plan == "premium":
        lines.append("This is a premium customer, apologise for any delay and offer a callback.")
    else:
        lines.append("This is a free plan customer, keep the answer to two lines.")

    lines.append(f"Reply in {customer.language}.")

    prompt = " ".join(lines)
    print("[prompt used]", prompt)
    print()
    return prompt

agent = create_agent(
    model="gpt-4o-mini",

    tools=[order_status],
    middleware=[support_prompt],
    context_schema=Customer,
)

question = {
    "messages": [
        {
            "role": "user",
            "content": "Where is my order ORD-1002?"
        }
    ]
}
for customer in [
    Customer(name="Asha", plan="premium", language="English"),
    Customer(name="Ramesh", plan="free", language="Hindi"),
]:


    result = agent.invoke(question, context=customer)
    print(f"To {customer.name} ({customer.plan}):")
    print(result["messages"][-1].content)
```

# LangChain Agent with Dynamic Prompt Middleware – Code Explanation

```markdown

This script shows how to create a **LangChain agent** that generates a **different system prompt** for each customer using the `@dynamic_prompt` middleware.  
The prompt is built **before every model call** based on customer data (name, plan, language).

---

## 1. Imports & Setup

```python
import sys
from dataclasses import dataclass
from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt
from langchain_core.tools import tool

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")
```

- `load_dotenv()` → Loads environment variables (e.g. `OPENAI_API_KEY`).
- `sys.stdout.reconfigure(encoding="utf-8")` → Ensures proper Unicode printing.
- `@dataclass` → Used to create a clean `Customer` object.
- `@dynamic_prompt` → Middleware that lets you **create or change the system prompt** right before the model is called.
- `@tool` → Turns a normal function into a LangChain tool.

---

## 2. Customer Context (Dataclass)

```python
@dataclass
class Customer:
    """What we know about the person asking, before the model sees anything."""
    name: str
    plan: str
    language: str
```

- This is the **context object** we will pass to the agent.
- It holds information about the customer (name, plan type, preferred language).
- The agent never sees this object directly — the middleware uses it to build the system prompt.

---

## 3. Defining a Tool

```python
@tool
def order_status(order_id: str) -> str:
    """Get the status of an order using its id."""
    return f"{order_id.upper()}: packed, ships tomorrow"
```

- Simple tool the agent can call when it needs order information.
- Returns a hardcoded status for demonstration.

---

## 4. Dynamic Prompt Middleware

```python
@dynamic_prompt
def support_prompt(request):
    customer = request.runtime.context   # ← gets the Customer object we passed

    lines = [
        f"You are a support agent for an online store. The customer is {customer.name}.",
        "Look up the order before answering, never guess.",
        "Write like a chat reply, no email signature.",
    ]

    if customer.plan == "premium":
        lines.append("This is a premium customer, apologise for any delay and offer a callback.")
    else:
        lines.append("This is a free plan customer, keep the answer to two lines.")

    lines.append(f"Reply in {customer.language}.")

    prompt = " ".join(lines)
    print("[prompt used]", prompt)
    print()
    return prompt
```

### What this does:
- Runs **before every model call**.
- Reads the `Customer` object from `request.runtime.context`.
- Builds a **custom system prompt** based on:
  - Customer name
  - Whether they are **premium** or **free**
  - Preferred language
- Returns the final prompt string (this becomes the system message).

### Example prompts generated:

**For Asha (premium, English):**
> You are a support agent for an online store. The customer is Asha. Look up the order before answering, never guess. Write like a chat reply, no email signature. This is a premium customer, apologise for any delay and offer a callback. Reply in English.

**For Ramesh (free, Hindi):**
> You are a support agent for an online store. The customer is Ramesh. Look up the order before answering, never guess. Write like a chat reply, no email signature. This is a free plan customer, keep the answer to two lines. Reply in Hindi.

---

## 5. Creating the Agent

```python
agent = create_agent(
    model="gpt-4o-mini",
    tools=[order_status],
    middleware=[support_prompt],
    context_schema=Customer,   # ← tells the agent what context object to expect
)
```

- Uses `gpt-4o-mini`.
- Registers the `order_status` tool.
- Attaches the `support_prompt` middleware.
- `context_schema=Customer` → Required so the agent knows the structure of the context we will pass.

---

## 6. Running the Agent for Multiple Customers

```python
question = {
    "messages": [
        {
            "role": "user",
            "content": "Where is my order ORD-1002?"
        }
    ]
}

for customer in [
    Customer(name="Asha", plan="premium", language="English"),
    Customer(name="Ramesh", plan="free", language="Hindi"),
]:
    result = agent.invoke(question, context=customer)
    print(f"To {customer.name} ({customer.plan}):")
    print(result["messages"][-1].content)
```

### What happens:
1. Same user question is asked twice.
2. First time → context = **Asha (premium, English)**  
   → Dynamic prompt is generated for premium customer → model replies accordingly.
3. Second time → context = **Ramesh (free, Hindi)**  
   → Completely different prompt is generated → model replies in Hindi and keeps it short.

---

## Execution Flow

```
agent.invoke(..., context=Customer(...))
        ↓
[dynamic_prompt] runs
        ↓
Builds custom system prompt using Customer data
        ↓
Model receives the new system prompt + user message
        ↓
Model may call the order_status tool
        ↓
Final answer is returned (personalized)
```

---

## Key Takeaways

| Concept                | Explanation                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| `@dynamic_prompt`      | Middleware that lets you generate/change the system prompt before the model |
| `request.runtime.context` | Access to the custom context object you passed to `agent.invoke()`       |
| `context_schema`       | Tells the agent what type of context object to expect                       |
| Benefit                | Same agent can behave differently for different users without rewriting code |

This pattern is very useful for:
- Personalizing responses (language, tone, plan type)
- Role-based instructions
- Multi-tenant support agents
- A/B testing different prompts
```

