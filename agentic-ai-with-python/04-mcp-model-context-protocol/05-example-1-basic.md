# Code Example

## `my_mcp_client.py`

```python
from dotenv import load_dotenv
import sys
import asyncio
from langchain_core.messages import HumanMessage, SystemMessage, ToolMessage
from langchain_openai import ChatOpenAI
from pathlib import Path
from langchain_mcp_adapters.client import MultiServerMCPClient

load_dotenv()

server_params = {
    "FirstMCPServer": {
        "command": sys.executable,
        "args": [str(Path(__file__).parent / "my_mcp_server.py")],
        "transport": "stdio",
    }
}


async def main():
    mcp_server = MultiServerMCPClient(server_params)

    tools = await mcp_server.get_tools()
    print(tools)

    llm = ChatOpenAI(model="gpt-4o-mini").bind_tools(tools)

    history = [
        HumanMessage(content="Add 4 and 5"),
    ]

    response = llm.invoke(history)
    history.append(response)

    toolcall = response.tool_calls[0]

    tool = next(
        t for t in tools
        if t.name == toolcall["name"]
    )

    result = await tool.ainvoke(toolcall["args"])

    print(result[0]["text"])

    history.append(
        ToolMessage(
            content=result[0]["text"],
            tool_call_id=toolcall["id"],
        )
    )

    response = await llm.ainvoke(history)
    print(response.content)


if __name__ == "__main__":
    asyncio.run(main())
```

## `my_mcp_server.py`

```python
from fastmcp import FastMCP

mcp_server = FastMCP("FirstMCPServer")


@mcp_server.tool
def greet():
    """This is the tool to greet users"""
    return "Hello from Telusko, we hope you are fine?"


@mcp_server.tool
def add(num1, num2):
    """This is the tool to add two numbers and give the output"""
    return 20


def main():
    mcp_server.run(transport="stdio")


if __name__ == "__main__":
    main()
```

# MCP Client + Server Example – Code Explanation

This is a simple example of using **Model Context Protocol (MCP)** with LangChain.  
It has two files:

- `my_mcp_server.py` → The MCP Server (provides tools)
- `my_mcp_client.py` → The MCP Client (uses those tools with an LLM)

---

## 1. MCP Server (`my_mcp_server.py`)

```python
from fastmcp import FastMCP

mcp_server = FastMCP("FirstMCPServer")

@mcp_server.tool
def greet():
    '''This is the tool to greet users'''
    return "Hello from Telusko, we hope you are fine?"

@mcp_server.tool
def add(num1, num2):
    '''This is the tool to add two numbers and give the output'''
    return 20

def main():
   mcp_server.run(transport="stdio")

if __name__ == "__main__":
    main()
```

### What this does:

| Part                        | Explanation |
|----------------------------|-----------|
| `FastMCP("FirstMCPServer")` | Creates an MCP server with the name **FirstMCPServer** |
| `@mcp_server.tool`          | Registers a function as an MCP tool |
| `greet()`                   | A simple tool that returns a greeting message |
| `add(num1, num2)`           | A tool that should add two numbers (but currently always returns `20`) |
| `mcp_server.run(transport="stdio")` | Starts the server using **stdio** transport (communicates through standard input/output) |

**Note:** The `add` tool currently ignores the inputs and always returns `20`. In a real tool it should return `num1 + num2`.

---

## 2. MCP Client (`my_mcp_client.py`)

```python
from dotenv import load_dotenv
import sys
import asyncio
from langchain_core.messages import HumanMessage, SystemMessage, ToolMessage
from langchain_openai import ChatOpenAI
from pathlib import Path
from langchain_mcp_adapters.client import MultiServerMCPClient

load_dotenv()
```

- Loads environment variables (for OpenAI API key)
- Imports LangChain message types and the MCP client adapter

---

### Server Configuration

```python
server_params = {
    "FirstMCPServer": {
        "command": sys.executable,
        "args": [str(Path(__file__).parent / "my_mcp_server.py")],
        "transport": "stdio",
    }
}
```

This tells the client:

- Start the server by running `my_mcp_server.py` using the current Python interpreter
- Communicate using **stdio** (standard input/output)

---

### Main Logic

```python
async def main():
    mcp_server = MultiServerMCPClient(server_params)
    tools = await mcp_server.get_tools()
    print(tools)
```

1. Creates a multi-server MCP client
2. Connects to the server and fetches all available tools
3. Prints the tools (you will see `greet` and `add`)

---

### Binding Tools to LLM

```python
    llm = ChatOpenAI(model="gpt-4o-mini").bind_tools(tools)
```

- Creates an OpenAI model (`gpt-4o-mini`)
- Binds the MCP tools so the model can call them

---

### First LLM Call

```python
    history = [
        HumanMessage(content="Add 4 and 5"),
    ]
    response = llm.invoke(history)
    history.append(response)
```

- Sends the user question: **"Add 4 and 5"**
- The model decides to call the `add` tool
- Adds the model’s response (which contains the tool call) to history

---

### Executing the Tool

```python
    toolcall = response.tool_calls[0]
    tool = next(t for t in tools if t.name == toolcall["name"])
    result = await tool.ainvoke(toolcall["args"])
    print(result[0]['text'])
```

1. Gets the first tool call from the model
2. Finds the matching tool object
3. Calls the tool with the arguments the model provided
4. Prints the result (currently `20` because of the hardcoded return)

---

### Sending Tool Result Back to LLM

```python
    history.append(ToolMessage(content=result[0]['text'], tool_call_id=toolcall["id"]))
    response = await llm.ainvoke(history)
    print(response.content)
```

1. Adds the tool result as a `ToolMessage`
2. Calls the LLM again with the full history
3. The model now generates a final natural language answer

---

## Full Flow Summary

```
1. Client starts the MCP Server (my_mcp_server.py)
2. Client fetches tools → [greet, add]
3. User asks: "Add 4 and 5"
4. LLM decides to call the `add` tool
5. Client executes the tool → gets result (20)
6. Client sends tool result back to LLM
7. LLM gives final answer based on the tool result
```

---

## Key Concepts

| Concept                    | Explanation |
|---------------------------|-----------|
| **MCP Server**            | Provides tools (like a remote function library) |
| **MCP Client**            | Connects to the server and uses its tools |
| **stdio transport**       | Communication happens through standard input/output |
| **Tool Binding**          | LLM is given the tools so it can decide when to call them |
| **ToolMessage**           | Special message type used to send tool results back to the LLM |

---

## Current Limitation

In `add` tool:

```python
return 20
```

It always returns 20, even if you ask to add 4 and 5.  
To make it correct, change it to:

```python
return num1 + num2
```