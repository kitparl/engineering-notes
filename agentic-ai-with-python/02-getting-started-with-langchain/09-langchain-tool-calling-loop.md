```
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool


# ============================================================
# 1. CREATE TOOLS
# ============================================================

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    result = a * b

    print(f"\n🔧 multiply tool called")
    print(f"   {a} * {b} = {result}")

    return result


@tool
def word_count(text: str) -> int:
    """Count the number of words in a text."""
    result = len(text.split())

    print(f"\n🔧 word_count tool called")
    print(f"   Text: {text}")
    print(f"   Word count: {result}")

    return result


# ============================================================
# 2. CREATE TOOL LIST
# ============================================================

tools = [
    multiply,
    word_count
]


# ============================================================
# 3. CREATE TOOL DICTIONARY
# ============================================================

tools_by_name = {
    tool.name: tool
    for tool in tools
}


# After this, tools_by_name looks like:
#
# {
#     "multiply": <multiply tool>,
#     "word_count": <word_count tool>
# }


# ============================================================
# 4. CREATE MODEL
# ============================================================

model = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0
)

model = model.bind_tools(tools)


# ============================================================
# 5. USER MESSAGE
# ============================================================

messages = [
    {
        "role": "user",
        "content": """
        Multiply 10 by 5.

        Also count the words in:
        "LangChain makes agents simple"
        """
    }
]


# ============================================================
# 6. START TOOL CALLING LOOP
# ============================================================

step = 1

while True:

    # --------------------------------------------------------
    # STEP 1: ASK THE MODEL
    # --------------------------------------------------------

    response = model.invoke(messages)


    # Add the model response to conversation
    messages.append(response)


    # --------------------------------------------------------
    # STEP 2: EXIT BRANCH
    # --------------------------------------------------------
    #
    # If the model does NOT request any tools,
    # the agent is finished.
    #
    # --------------------------------------------------------

    if not response.tool_calls:

        print("\nFinal answer:")
        print(response.content)

        break


    # --------------------------------------------------------
    # STEP 3: MODEL REQUESTED TOOLS
    # --------------------------------------------------------

    print(
        f"\nStep {step}: the model asked for "
        f"{len(response.tool_calls)} tool call(s)"
    )


    # --------------------------------------------------------
    # STEP 4: EXECUTE EACH TOOL
    # --------------------------------------------------------

    for call in response.tool_calls:

        print("\nTool call:")
        print(call)


        # Get the tool name
        tool_name = call["name"]


        # Find the actual tool
        tool_to_run = tools_by_name[tool_name]


        # Execute the tool
        #
        # IMPORTANT:
        # Passing the complete `call` object allows LangChain
        # to create the ToolMessage associated with this call.
        #
        tool_message = tool_to_run.invoke(call)


        # Add tool result to conversation
        messages.append(tool_message)


    # --------------------------------------------------------
    # STEP 5: NEXT ROUND
    # --------------------------------------------------------

    step += 1
```

> Keep running the agent until it reaches the exit branch.

# Step 1 — Call the LLM

```
response = model.invoke(messages)
```

The model looks at the user's request and decides:

> "Do I need a tool?"
For our example, it might decide:

```
I need multiply
I need word_count
```

So:

```
response.tool_calls
```

might contain:

```
[
    multiply(10, 5),
    word_count("LangChain makes agents simple")
]
```

# Step 2 — Add the LLM Response

messages.append(response)

This is important because we want to preserve the conversation.

The messages now contain approximately:

```
User
 ↓
"Multiply 10 by 5 and count words..."

LLM
 ↓
"I want to call multiply and word_count"
```

# Step 3 — Check the Exit Branch

```
if not response.tool_calls:
    print("\nFinal answer:")
    print(response.content)
    break
```

This is the exit branch.

It asks:

```
Did the LLM request a tool?
```

- If YES
```
response.tool_calls
       ↓
   has tools
       ↓
Execute tools
```

- If NO
```
response.tool_calls
       ↓
      []
       ↓
Final answer
       ↓
EXIT
```
That's why: break is there

# Step 4 — The for Loop

```
for call in response.tool_calls:
```

Suppose the LLM returns:

```
response.tool_calls

    ┌─────────────────────┐
    │ multiply(10, 5)     │
    │ word_count("...")   │
    └─────────────────────┘
```

The for loop processes them one by one:

```
for call in response.tool_calls

        ↓

call = multiply(10, 5)

        ↓

execute multiply

        ↓

50
```

Then:

```
call = word_count("...")

        ↓

execute word_count

        ↓

4
```

So the for loop means:
> Execute every tool requested by the current LLM response.


# Step 5 — Find the Correct Tool

```
tool_to_run = tools_by_name[call["name"]]
```

Suppose: `call["name"]` is "word_count"
Then: tools_by_name["word_count"]

gives you the actual Python tool: `word_count`

```
call["name"]
      ↓
"word_count"
      ↓
tools_by_name["word_count"]
      ↓
word_count tool
```

This dictionary:

```
tools_by_name = {
    tool.name: tool
    for tool in tools
}
```
is basically a lookup table.

# Step 6 — Execute the Tool

```
tool_message = tool_to_run.invoke(call)
```

This is particularly useful to understand.
The call contains something like:

```
{
    "name": "multiply",
    "args": {
        "a": 10,
        "b": 5
    },
    "id": "call_123"
}
```

Then: 

```
tool_to_run.invoke(call)
```

execute:

```
multiply(a=10, b=5)
```

and produces a tool message containing the result.

Conceptually:

```
call
 ↓
multiply(a=10, b=5)
 ↓
50
 ↓
ToolMessage
```

# Step 7 — Add Tool Result Back
This line is extremely important:
```
messages.append(tool_message)
```

Now the conversation becomes:

```
User
 ↓
"Multiply 10 by 5..."

LLM
 ↓
"I want to call multiply"

Tool
 ↓
50
```

Then the while loop goes back to:

```
response = model.invoke(messages)
```

So the LLM gets another chance to respond.

# Complete Flow for My Example

```
USER
 │
 │ "Multiply 10 × 5 and count words..."
 ▼
LLM
 │
 │ tool_calls:
 │ ├── multiply(10, 5)
 │ └── word_count("LangChain makes agents simple")
 ▼
FOR LOOP
 │
 ├── multiply
 │      ↓
 │      50
 │
 └── word_count
        ↓
        4
 │
 ▼
messages.append(tool_message)
 │
 ▼
LLM AGAIN
 │
 │ sees:
 │ multiply result = 50
 │ word count = 4
 ▼
LLM
 │
 │ "10 × 5 = 50.
 │    The sentence has 4 words."
 ▼
response.tool_calls == []
 │
 ▼
EXIT
```

# The Most Important Part

```
while True:

    # Ask LLM
    response = model.invoke(messages)

    # Save LLM response
    messages.append(response)

    # EXIT?
    if not response.tool_calls:
        print(response.content)
        break

    # Execute requested tools
    for call in response.tool_calls:

        tool_to_run = tools_by_name[call["name"]]

        tool_message = tool_to_run.invoke(call)

        # Give tool result back to LLM
        messages.append(tool_message)
```

Remember it as:

```
             ┌──────────┐
             │   LLM    │
             └────┬─────┘
                  │
            tool_calls?
             /         \
           YES          NO
            │            │
            ▼            ▼
       ┌─────────┐      EXIT
       │  FOR    │
       │  LOOP   │
       └────┬────┘
            │
       Execute tools
            │
            ▼
       ToolMessage
            │
            ▼
           LLM
            │
            └───────────→ check again
```

- One line for each concept
while True:

- Keep the agent loop running.

if not response.tool_calls:
    break

- Exit when the LLM doesn't need another tool.

for call in response.tool_calls:

- Handle multiple tool calls from one LLM response.

tool_to_run = tools_by_name[call["name"]]

- Find which Python tool the LLM requested.

tool_message = tool_to_run.invoke(call)

- Actually execute the tool.

messages.append(tool_message)


