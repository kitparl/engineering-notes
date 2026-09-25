
# MCP over JSON-RPC 2.0 example.

## 1 Example - tools

Suppose an MCP server exposes a tool called get_weather.

1. Client asks what tools are available

## Request:

```
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {}
}
```

## Response

```
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "get_weather",
        "description": "Get current weather for a city",
        "inputSchema": {
          "type": "object",
          "properties": {
            "city": {
              "type": "string"
            }
          },
          "required": ["city"]
        }
      }
    ]
  }
}
```

So the client now knows:

> There is a tool called get_weather, and it accepts a city.

2. Client calls the tool

## Request

```
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "get_weather",
    "arguments": {
      "city": "Mumbai"
    }
  }
}
```

The MCP server might internally do:

```
tools/call
    ↓
get_weather("Mumbai")
    ↓
Weather API
    ↓
result
```

## Response
```
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Mumbai: 29°C, partly cloudy"
      }
    ]
  }
}
```

3. Notice the layers

The interesting part is that JSON-RPC doesn't know anything about weather.

```
┌─────────────────────────────┐
│ JSON-RPC 2.0                │
│ request / response / id     │
├─────────────────────────────┤
│ MCP                         │
│ tools/call, tools/list, ... │
├─────────────────────────────┤
│ Your MCP server             │
│ get_weather()               │
├─────────────────────────────┤
│ Actual data layer           │
│ Weather API / DB / files    │
└─────────────────────────────┘
```

Yes. MCP has three important primitives to understand:

* **Tools** → things the model can *do*
* **Resources** → data/context the model can *read*
* **Prompts** → reusable interaction templates

Here are concrete JSON-RPC examples.

## 2. Resources

Imagine an MCP server exposes a company's customer data.

### List resources

**Request**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "resources/list",
  "params": {}
}
```

**Response**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "resources": [
      {
        "uri": "customer://123",
        "name": "Customer 123",
        "description": "Customer profile for customer 123",
        "mimeType": "application/json"
      },
      {
        "uri": "customer://456",
        "name": "Customer 456",
        "description": "Customer profile for customer 456",
        "mimeType": "application/json"
      }
    ]
  }
}
```

The important thing here is the **URI**:

```text
customer://123
```

It identifies a piece of data exposed by the MCP server.

### Read a resource

**Request**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "resources/read",
  "params": {
    "uri": "customer://123"
  }
}
```

**Response**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "contents": [
      {
        "uri": "customer://123",
        "mimeType": "application/json",
        "text": "{\"id\":123,\"name\":\"Alice\",\"plan\":\"premium\"}"
      }
    ]
  }
}
```

So conceptually:

```text
Client
   │
   │ resources/read
   │ customer://123
   ▼
MCP Server
   │
   │ fetch data
   ▼
Database
```

---

# 3. Prompts

Prompts are **reusable templates** exposed by an MCP server.

For example, your MCP server could provide:

```text
analyze_customer
```

### List prompts

**Request**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "prompts/list",
  "params": {}
}
```

**Response**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "prompts": [
      {
        "name": "analyze_customer",
        "description": "Analyze a customer's account and activity",
        "arguments": [
          {
            "name": "customer_id",
            "description": "ID of the customer",
            "required": true
          }
        ]
      }
    ]
  }
}
```

### Get a prompt

The client can then request that prompt with arguments:

**Request**

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "prompts/get",
  "params": {
    "name": "analyze_customer",
    "arguments": {
      "customer_id": "123"
    }
  }
}
```

**Response**

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {
    "description": "Analyze customer 123",
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Analyze customer 123. Review their profile, recent activity, and subscription status. Identify important observations."
        }
      }
    ]
  }
}
```

The host/model can then use that message as part of the conversation.

---

# 3. Tools vs Resources vs Prompts

This distinction is probably the most useful way to think about MCP:

| Primitive    | Purpose                                 | Example            |
| ------------ | --------------------------------------- | ------------------ |
| **Tool**     | Perform an action                       | `create_invoice()` |
| **Resource** | Provide/read data                       | `customer://123`   |
| **Prompt**   | Provide a reusable interaction template | `analyze_customer` |

For example:

```text
                    MCP Server
                        │
          ┌─────────────┼─────────────┐
          │             │             │
        Tools        Resources      Prompts
          │             │             │
          ▼             ▼             ▼
     "Do something"  "Read data"   "Use this
                                   template"
```

### A realistic workflow

Suppose the user says:

> "Analyze customer 123 and tell me whether they might churn."

The MCP interaction could conceptually be:

```text
1. prompts/list
       ↓
2. prompts/get
   analyze_customer
       ↓
3. resources/read
   customer://123
       ↓
4. resources/read
   customer://123/activity
       ↓
5. Model analyzes the data
       ↓
6. Maybe tools/call
   send_customer_alert(...)
```

So **MCP isn't simply "an API over JSON-RPC."** It defines a standardized way for an AI application to discover and interact with **tools, resources, and prompts**, with JSON-RPC providing the underlying RPC message structure.

