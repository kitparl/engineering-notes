wrap_model_call is a middleware wrapper that gives you control before and after an LLM call.

```
Agent
  ↓
wrap_model_call
  ↓
   MODEL
  ↓
wrap_model_call
  ↓
Agent
```

**`wrap_model_call` in LangChain** is a middleware hook (and decorator) that lets you intercept and control every model call made by an agent.

It is part of LangChain’s agent middleware system (available in recent versions, roughly LangChain 1.0+). There is both a sync version and an async counterpart (`awrap_model_call`).

### What it does
- Runs **around** each model invocation.
- You receive a `ModelRequest` (contains messages, model, tools, state, runtime context, etc.).
- You get a `handler` callback that actually executes the model call and returns a `ModelResponse` (or `AIMessage`).
- You decide:
  - Whether to call the handler (normal flow)
  - How many times to call it (retry logic)
  - Whether to skip it entirely (short-circuit)
  - Whether to modify the request before calling the model
  - Whether to rewrite the response after the call
- Multiple middleware layers compose (first in the list is the outermost).

### Basic usage (decorator form)

```python
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
from typing import Callable

@wrap_model_call
def retry_model(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse],
) -> ModelResponse:
    for attempt in range(3):
        try:
            return handler(request)
        except Exception as e:
            if attempt == 2:
                raise
            print(f"Retry {attempt + 1}/3 after error: {e}")
```

### Common patterns

**1. Dynamic model selection**
```python
@wrap_model_call
def dynamic_model_selection(request: ModelRequest, handler) -> ModelResponse:
    message_count = len(request.state["messages"])
    model = advanced_model if message_count > 10 else basic_model
    return handler(request.override(model=model))
```

**2. Modify tools / prompt / response format**
```python
@wrap_model_call
def filter_tools(request: ModelRequest, handler) -> ModelResponse:
    # e.g. only allow certain tools based on state
    filtered = request.override(tools=[t for t in request.tools if ...])
    return handler(filtered)
```

**3. State updates from the model-call layer** (via `ExtendedModelResponse` + `Command`)
```python
from langchain.agents.middleware import ExtendedModelResponse
from langgraph.types import Command

@wrap_model_call
def track_usage(request: ModelRequest, handler) -> ExtendedModelResponse:
    response = handler(request)
    return ExtendedModelResponse(
        model_response=response,
        command=Command(update={"last_model_call_tokens": 150}),
    )
```

### Class-based form
You can also implement it as a method on a class that inherits from `AgentMiddleware`:

```python
class RetryMiddleware(AgentMiddleware):
    def wrap_model_call(self, request, handler):
        # same logic as above
        ...
```

### Key related pieces
- `ModelRequest` / `ModelResponse`
- `request.override(...)` to create a modified request
- Async support via `@wrap_model_call` on an async function or by implementing `awrap_model_call`
- Companion hook: `wrap_tool_call` (same idea, but for tool execution)

Official docs live under the agent middleware / custom middleware sections on the LangChain docs site.

Would you like a concrete example for a specific use case (retries, dynamic model routing, tool filtering, caching, token tracking, etc.)?


# usecases

wrap_model_call — Agent Use Cases
- Model selection — choose different models based on the task.
- Model routing — route simple tasks to a cheaper model and complex tasks to a stronger model.
- Logging — log every model request and response.
Timing — measure how long each model call takes.
- Token/cost tracking — monitor token usage and model costs.
- Guardrails — validate or block model requests before execution.
- Request modification — dynamically modify prompts, tools, or model configuration.
- Retry logic — retry failed model calls.
- Fallback models — switch to another model if the primary model fails.
- Dynamic tool control — change which tools are available to the model.
- Caching — reuse previous model responses when appropriate.
- Monitoring — collect metrics about model performance.
- Error handling — catch and handle model/API failures.
- Rate limiting — control how frequently the agent can call the model.
- Human approval — pause a model action and require human approval before continuing.