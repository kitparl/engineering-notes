online-store customer-support agent

```
import sys

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain_core.tools import tool


load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")


# our data --> imagine its our order data base
ORDERS = {
    "ORD-1001": {
        "item": "Wireless mouse",
        "status": "shipped",
        "amount": 1499
    },

    "ORD-1002": {
        "item": "Mechanical keyboard",
        "status": "packed",
        "amount": 4999
    },
}
# imagine we have warehouse stock
STOCK = {
    "wireless mouse": 12,
    "mechanical keyboard": 0,
    "usb hub": 34
}

# our tool of this app

# =========================================================
# TOOL 1
# =========================================================

@tool
def order_status(order_id: str) -> str:
    """
    Get the status and amount of an order using its id,
    for example ORD-1001.
    """

    order = ORDERS.get(order_id.upper())

    if order is None:
        return f"No order found with id {order_id}."

    return (
        f"{order['item']}, "
        f"status {order['status']}, "
        f"amount {order['amount']} rupees"
    )


# =========================================================
# TOOL 2
# =========================================================

@tool
def check_stock(item: str) -> str:
    """
    Check how many units of an item are left in the warehouse.
    """

    count = STOCK.get(item.lower())

    if count is None:
        return f"{item} is not in the catalogue."

    return f"{count} units of {item} in stock"


# =========================================================
# TOOL 3
# =========================================================

@tool
def apply_discount(amount: float, percent: float) -> float:
    """
    Apply a discount percentage to an amount
    and return the new amount.
    """

    return round(
        amount - (amount * percent / 100),
        2
    )


# =========================================================
# TOOL 4
# =========================================================

@tool
def delivery_days(pin_code: str) -> str:
    """
    Estimate delivery time for an Indian pin code.
    """

    metro = {
        "400001",
        "110001",
        "560001"
    }

    return "2 days" if pin_code in metro else "5 days"

agent = create_agent(
   model="openai:gpt-4o-mini", 
   tools=[
        order_status,
        check_stock,
        apply_discount,
        delivery_days
   ],
   system_prompt=(
        "You are a support assistant for an online store. "
        "Use the tools for anything about orders, stock, "
        "pricing or delivery"
        "Never guess a number, look it up."
   ),

)

#Helper function
def ask(question):

    # Print the user's question.
    print("Q:", question)
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
        result["messages"][-1].content
    )


    print("-" * 60)

ask("What is the status of order ORD-1001")
ask("Do you have a mechanical keyboard in stock")
ask("when the mechanical keyboard will be available again ?")
ask(
    "For order ORD-1002, what would the price be "
    "after a 10 percent discount?"
)
```
# LangChain Multi-Tool Agent — Online Store Example

This example demonstrates a **realistic multi-tool customer-support agent** using LangChain's `create_agent()`.

The agent has four tools:

- `order_status` → checks an order
- `check_stock` → checks warehouse stock
- `apply_discount` → calculates discounted price
- `delivery_days` → estimates delivery time

The important concept is that **the LLM decides which tool(s) to use**, while `create_agent()` manages the tool-calling loop.

---

## 1. Application Data

The example uses Python dictionaries as fake databases.

```python
ORDERS = {
    "ORD-1001": {
        "item": "Wireless mouse",
        "status": "shipped",
        "amount": 1499
    },
    "ORD-1002": {
        "item": "Mechanical keyboard",
        "status": "packed",
        "amount": 4999
    },
}
```

Think of this as an **Order Database**.

```text
Order Database
     │
     ├── ORD-1001 → Wireless mouse → shipped → ₹1499
     │
     └── ORD-1002 → Mechanical keyboard → packed → ₹4999
```

The stock data is:

```python
STOCK = {
    "wireless mouse": 12,
    "mechanical keyboard": 0,
    "usb hub": 34
}
```

Think of this as a **Warehouse Database**.

---

# 2. The Four Tools

## Tool 1 — `order_status`

```python
@tool
def order_status(order_id: str) -> str:
```

Purpose:

> Look up an order using its order ID.

Example:

```text
order_status("ORD-1001")
        ↓
Wireless mouse, status shipped, amount 1499 rupees
```

## Tool 2 — `check_stock`

```python
@tool
def check_stock(item: str) -> str:
```

Purpose:

> Check how many units of an item are available.

Example:

```text
check_stock("mechanical keyboard")
        ↓
0 units of mechanical keyboard in stock
```

## Tool 3 — `apply_discount`

```python
@tool
def apply_discount(amount: float, percent: float) -> float:
```

Purpose:

> Calculate the price after applying a discount.

Example:

```text
apply_discount(4999, 10)
        ↓
4499.10
```

## Tool 4 — `delivery_days`

```python
@tool
def delivery_days(pin_code: str) -> str:
```

Purpose:

> Estimate delivery time for an Indian PIN code.

Example:

```text
delivery_days("400001")
        ↓
2 days
```

---

# 3. Creating the Agent

```python
agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[
        order_status,
        check_stock,
        apply_discount,
        delivery_days
    ],
    system_prompt=(
        "You are a support assistant for an online store. "
        "Use the tools for anything about orders, stock, "
        "pricing or delivery. "
        "Never guess a number, look it up."
    ),
)
```

We are telling LangChain:

> Create an agent with this model and give it these tools.

The LLM can then decide which tool is appropriate for the user's question.

---

# 4. `create_agent()` Manages the Tool-Calling Loop

Earlier, we manually wrote:

```python
while True:

    response = model.invoke(messages)

    if not response.tool_calls:
        break

    for call in response.tool_calls:
        ...
```

With:

```python
create_agent(...)
```

LangChain manages this process for us.

Conceptually:

```text
                 LLM
                  │
                  ▼
           Tool requested?
             /         \
           YES          NO
            │            │
            ▼            ▼
       Execute tool     EXIT
            │
            ▼
       Tool result
            │
            ▼
           LLM
            │
            └──────────→ check again
```

So:

> **`create_agent()` manages the LLM → tool → LLM → tool → ... → final answer loop.**

---

# 5. Example 1 — Order Status

```python
ask("What is the status of order ORD-1001")
```

The LLM understands that it needs order information.

It chooses:

```text
order_status
```

Flow:

```text
User
 ↓
"What is the status of ORD-1001?"
 ↓
LLM
 ↓
order_status("ORD-1001")
 ↓
"Wireless mouse, status shipped, amount 1499 rupees"
 ↓
LLM
 ↓
Final answer
```

The Python runtime executes the actual tool.

The LLM only **requests** the tool.

---

# 6. Example 2 — Stock

```python
ask("Do you have a mechanical keyboard in stock")
```

The LLM chooses:

```text
check_stock
```

Flow:

```text
User
 ↓
"Do you have a mechanical keyboard in stock?"
 ↓
LLM
 ↓
check_stock("mechanical keyboard")
 ↓
"0 units..."
 ↓
LLM
 ↓
Final answer
```

The LLM does not need us to manually call:

```python
check_stock("mechanical keyboard")
```

It chooses the tool based on the available tool descriptions.

---

# 7. Example 3 — When the Tool Does Not Have Enough Information

```python
ask("when the mechanical keyboard will be available again ?")
```

The stock tool can tell us:

```text
mechanical keyboard → 0 units
```

But our application has **no restock-date data** and no `restock_date()` tool.

Therefore, the agent cannot reliably know when it will be available again.

Important principle:

> **An agent cannot magically get information that is not available in its data or tools.**

The system prompt also says:

```text
Never guess a number, look it up.
```

So the agent should not invent a restock date.

---

# 8. Example 4 — Multiple Tools

```python
ask(
    "For order ORD-1002, what would the price be "
    "after a 10 percent discount?"
)
```

The agent needs two steps.

First, it needs the order amount:

```text
order_status("ORD-1002")
        ↓
₹4999
```

Then it can calculate the discount:

```text
apply_discount(4999, 10)
        ↓
₹4499.10
```

Conceptually:

```text
                    LLM
                     │
                     ▼
              order_status
                ORD-1002
                     │
                     ▼
                   ₹4999
                     │
                     ▼
                    LLM
                     │
                     ▼
              apply_discount
                 4999, 10
                     │
                     ▼
                 ₹4499.10
                     │
                     ▼
                    LLM
                     │
                     ▼
                Final Answer
```

This demonstrates why an agent loop is useful: the agent can **use the result of one tool as input to another tool**.

---

# 9. The `ask()` Helper Function

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

    print(
        "A:",
        result["messages"][-1].content
    )

    print("-" * 60)
```

This is simply a helper function.

Instead of repeating:

```python
agent.invoke(...)
```

for every question, we can write:

```python
ask("What is the status of order ORD-1001")
ask("Do you have a mechanical keyboard in stock")
ask("when the mechanical keyboard will be available again?")
```

---

# 10. `result["messages"]`

The agent returns a result containing the conversation messages:

```python
result["messages"]
```

Conceptually:

```text
HumanMessage
    ↓
User question

AIMessage
    ↓
Tool call

ToolMessage
    ↓
Tool result

AIMessage
    ↓
Final answer
```

The last message is normally the final AI response:

```python
result["messages"][-1].content
```

So:

```python
print(result["messages"][-1].content)
```

prints the final answer.

---

# 11. Manual Loop vs `create_agent()`

## Manual approach

You explicitly write the loop:

```python
while True:

    response = model.invoke(messages)

    if not response.tool_calls:
        break

    for call in response.tool_calls:

        tool_to_run = tools_by_name[call["name"]]

        tool_message = tool_to_run.invoke(call)

        messages.append(tool_message)
```

This is useful for **learning how agents work internally**.

## `create_agent()` approach

You simply write:

```python
agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[
        order_status,
        check_stock,
        apply_discount,
        delivery_days
    ]
)
```

LangChain manages the loop.

This is more convenient for building the actual application.

---

# 12. `while` vs `for`

These loops have different jobs.

### `while`

```python
while True:
```

Controls the **overall agent loop**:

```text
LLM
 ↓
Tool
 ↓
LLM
 ↓
Tool
 ↓
LLM
 ↓
EXIT
```

### `for`

```python
for call in response.tool_calls:
```

Handles **multiple tools requested in one LLM response**:

```text
One LLM response
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
Tool1 Tool2 Tool3
```

Easy way to remember:

> **`while` = keep the agent going**

> **`for` = execute every tool requested by the current LLM response**

With `create_agent()`, LangChain handles both parts for you.

---

# 13. Main Mental Model

```text
                         USER
                           │
                           ▼
                          LLM
                           │
                    Which tool?
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
    order_status      check_stock      apply_discount
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                      Tool result
                           │
                           ▼
                          LLM
                           │
                    Need another tool?
                     /             \
                   YES              NO
                    │                │
                    ▼                ▼
              Execute tool        FINAL ANSWER
                    │
                    └──────→ LLM
```

## Key Takeaway

> **Build a multi-tool customer-support agent where the LLM decides which tools to use, LangChain manages the tool-calling loop, tool results are returned to the LLM, and the agent continues until it can produce the final answer.**

The four tools are the agent's **capabilities**, the LLM is the **decision maker**, and `create_agent()` manages the **agent loop**.



