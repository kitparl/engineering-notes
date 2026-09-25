# Code Example

```
import re
import sys

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import PIIMatch, PIIMiddleware, PIIDetectionError

load_dotenv()

sys.stdout.reconfigure(encoding="utf-8")

def indian_phone(text: str) -> list[PIIMatch]:
    """Find 10 digit Indian mobile numbers, with or without the country code."""
    matches = []
    for found in re.finditer(r"(?:\+91[\s-]?)?[6-9]\d{9}", text):  # +91-88  , +91 88  
        matches.append(PIIMatch(type="phone", value=found.group(), start=found.start(), end=found.end()))
    return matches


agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[],
    system_prompt="You are a support agent. Confirm what the customer told you in one line.",
    middleware=[
        PIIMiddleware("email", strategy="redact"),
        PIIMiddleware("credit_card", strategy="mask"),
        PIIMiddleware("phone", detector=indian_phone, strategy="redact"),
    ],
)
message = (
    "Hi, I am Asha. Mail me at asha.k@example.com or call 9876543210. "
    "I paid with card 4111 1111 1111 1111."
)
result =agent.invoke(
    {
        "messages": [
    {
        "role": "user", 
        "content": message
        }
    ]
    })
print("What customer typed:")
print(" ", message)
print()

print("What the model received:")
print(" ", result["messages"][0].content)
print()

print("Answer:")
print(" ", result["messages"][-1].content)

print()
strict =create_agent(
    model="openai:gpt-4o-mini",
    tools=[],
    system_prompt="You are a support agent. Confirm what the customer told you in one line.",
    middleware=[
     
        PIIMiddleware("credit_card", strategy="block")

    ],
)
try:
    strict.invoke({"messages": [{"role": "user", "content": "my card is 4111 1111 1111 1111"}]})
except PIIDetectionError:
    print("Blocked, ask the customer never to send card numbers in chat.")
```

# LangChain Agent with PII Middleware – Code Explanation

This script demonstrates the **built-in `PIIMiddleware`**.  
It automatically detects and handles sensitive information (PII) such as emails, credit cards, and phone numbers before the data reaches the model.

---

## 1. Imports & Setup

```python
import re
from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import PIIMatch, PIIMiddleware, PIIDetectionError

load_dotenv()
sys.stdout.reconfigure(encoding="utf-8")
```

- `re` → Used for custom regex-based PII detection.
- `PIIMiddleware` → Built-in middleware that scans messages for sensitive data.
- `PIIMatch` → Object that represents a detected piece of PII.
- `PIIDetectionError` → Exception raised when strategy is set to `"block"`.

---

## 2. Custom Detector – Indian Phone Numbers

```python
def indian_phone(text: str) -> list[PIIMatch]:
    """Find 10 digit Indian mobile numbers, with or without the country code."""
    matches = []
    for found in re.finditer(r"(?:\+91[\s-]?)?[6-9]\d{9}", text):
        matches.append(
            PIIMatch(
                type="phone",
                value=found.group(),
                start=found.start(),
                end=found.end()
            )
        )
    return matches
```

### What this does:
- Uses regex to find Indian mobile numbers:
  - Optional `+91` with space or hyphen
  - Starts with 6–9
  - Followed by 9 more digits
- Returns a list of `PIIMatch` objects containing:
  - `type` → `"phone"`
  - `value` → the matched number
  - `start` / `end` → position in the text

---

## 3. Agent with Multiple PII Rules

```python
agent = create_agent(
    model="openai:gpt-4o-mini",
    tools=[],
    system_prompt="You are a support agent. Confirm what the customer told you in one line.",
    middleware=[
        PIIMiddleware("email", strategy="redact"),
        PIIMiddleware("credit_card", strategy="mask"),
        PIIMiddleware("phone", detector=indian_phone, strategy="redact"),
    ],
)
```

### PII Strategies Used:

| Type          | Strategy   | Effect                                      |
|---------------|------------|---------------------------------------------|
| `email`       | `redact`   | Completely removes the email                |
| `credit_card` | `mask`     | Replaces digits with `*` (keeps last 4)     |
| `phone`       | `redact`   | Completely removes the phone number         |

- Built-in detectors are used for `email` and `credit_card`.
- Custom detector (`indian_phone`) is used for phone numbers.

---

## 4. Test Message

```python
message = (
    "Hi, I am Asha. Mail me at asha.k@example.com or call 9876543210. "
    "I paid with card 4111 1111 1111 1111."
)
```

Contains three types of PII:
- Email → `asha.k@example.com`
- Phone → `9876543210`
- Credit card → `4111 1111 1111 1111`

---

## 5. Running the Agent

```python
result = agent.invoke({
    "messages": [{"role": "user", "content": message}]
})

print("What customer typed:")
print(" ", message)
print()
print("What the model received:")
print(" ", result["messages"][0].content)
print()
print("Answer:")
print(" ", result["messages"][-1].content)
```

### Expected Behavior:

**Original message:**
```
Hi, I am Asha. Mail me at asha.k@example.com or call 9876543210. I paid with card 4111 1111 1111 1111.
```

**What the model actually receives (after middleware):**
```
Hi, I am Asha. Mail me at [REDACTED] or call [REDACTED]. I paid with card **** **** **** 1111.
```

(The exact redaction/masking format may vary slightly depending on the middleware version.)

---

## 6. Strict Mode – Blocking Credit Cards

```python
strict = create_agent(
    model="openai:gpt-4o-mini",
    tools=[],
    system_prompt="You are a support agent. Confirm what the customer told you in one line.",
    middleware=[
        PIIMiddleware("credit_card", strategy="block")
    ],
)

try:
    strict.invoke({
        "messages": [{"role": "user", "content": "my card is 4111 1111 1111 1111"}]
    })
except PIIDetectionError:
    print("Blocked, ask the customer never to send card numbers in chat.")
```

### What happens:
- Strategy is set to `"block"`.
- When a credit card number is detected, the middleware **raises `PIIDetectionError`**.
- The agent does **not** call the model.
- You can catch the exception and show a friendly message to the user.

---

## Available Strategies Summary

| Strategy   | Behavior                                      | Use Case                              |
|------------|-----------------------------------------------|---------------------------------------|
| `redact`   | Completely removes the PII                    | Emails, phone numbers                 |
| `mask`     | Partially hides the value (e.g. `**** 1111`)  | Credit cards, account numbers         |
| `block`    | Raises `PIIDetectionError` and stops execution| Strict compliance (PCI-DSS, etc.)     |

---

## Key Takeaways

| Concept                  | Explanation                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| `PIIMiddleware`          | Built-in middleware that scans and protects sensitive data                  |
| Built-in detectors       | `email`, `credit_card`, etc. work out of the box                            |
| Custom detector          | You can write your own function that returns `list[PIIMatch]`               |
| `strategy="redact"`      | Removes the sensitive value                                                 |
| `strategy="mask"`        | Partially hides the value                                                   |
| `strategy="block"`       | Stops the request and raises `PIIDetectionError`                            |
| Why useful               | Helps meet privacy/compliance requirements and protects user data           |

This pattern is essential for production support agents where customers may accidentally share sensitive information.
