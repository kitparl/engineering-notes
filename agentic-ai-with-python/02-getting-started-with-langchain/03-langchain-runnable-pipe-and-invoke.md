# LangChain: `Runnable`, `|` Pipe, and `invoke()`

## 1. What is `invoke()`?

`invoke()` is **not a default Python method**.

It is part of LangChain's **Runnable interface**.

Many LangChain components behave as `Runnable`, for example:

* `ChatPromptTemplate`
* `ChatOpenAI`
* `StrOutputParser`
* Retrievers
* Chains

They provide a common interface such as:

```python
runnable.invoke(input)
runnable.stream(input)
runnable.batch(inputs)
```

### Common execution methods

| Method     | Purpose                                         |
| ---------- | ----------------------------------------------- |
| `invoke()` | Execute with one input and get one final result |
| `stream()` | Get the output progressively                    |
| `batch()`  | Execute multiple inputs                         |

---

## 2. What does `|` mean in LangChain?

Consider:

```python
chain = prompt | model | parser
```

The `|` is Python's **OR operator**, but LangChain objects customize its behavior using Python's special method:

```python
__or__()
```

Conceptually:

```python
prompt | model
```

becomes something like:

```python
RunnableSequence(prompt, model)
```

So:

```python
prompt | model | parser
```

creates a **pipeline/sequence**.

It does **not** execute the components immediately.

---

## 3. When does execution happen?

Execution happens when we call:

```python
chain.invoke(input)
```

For example:

```python
chain = prompt | model | parser

result = chain.invoke({"topic": "Python"})
```

Conceptually, LangChain does:

```python
output1 = prompt.invoke({"topic": "Python"})
output2 = model.invoke(output1)
output3 = parser.invoke(output2)

return output3
```

This is a simplified mental model, not the exact internal implementation.

---

## 4. Data flows through every pipe

The pipeline:

```python
prompt | model | parser
```

can be visualized as:

```text
Input
  ↓
Prompt
  ↓
ChatPromptValue
  ↓
Model
  ↓
AIMessage
  ↓
Parser
  ↓
String
```

Each component's output becomes the next component's input.

---

## 5. Example with `ChatPromptTemplate`

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "Explain {topic} in simple terms."
)

model = ChatOpenAI()

parser = StrOutputParser()

chain = prompt | model | parser

result = chain.invoke({
    "topic": "Python decorators"
})

print(result)
```

The execution flow is:

```text
{"topic": "Python decorators"}
              ↓
       ChatPromptTemplate
              ↓
   "Explain Python decorators
      in simple terms."
              ↓
          ChatOpenAI
              ↓
          AIMessage
              ↓
       StrOutputParser
              ↓
            String
```

---

## 6. Important distinction: `|` vs `invoke()`

These two things have different jobs.

### `|` → Build the pipeline

```python
chain = prompt | model | parser
```

Meaning:

> Connect these components together.

### `invoke()` → Execute the pipeline

```python
chain.invoke(input)
```

Meaning:

> Run the connected components with this input.

So:

```text
prompt | model | parser
        ↓
   Build pipeline
        ↓
chain.invoke(input)
        ↓
   Execute pipeline
```

---

## 7. Why does LangChain use `invoke()`?

LangChain wants different components to have a **common execution interface**.

Without a common interface, every component could have different methods:

```python
prompt.generate(...)
model.call(...)
parser.parse(...)
retriever.search(...)
```

That would make composition difficult.

Instead, LangChain provides a common Runnable interface:

```python
component.invoke(...)
```

So different components can be connected easily:

```python
prompt | model | parser
```

---

## 8. Why can different components be piped together?

Because they follow the **Runnable contract**.

Conceptually:

```text
             Runnable
                │
     ┌──────────┼──────────┐
     ↓          ↓          ↓
   Prompt      Model      Parser
     │          │          │
   invoke     invoke     invoke
```

As long as the output of one component is compatible with the input expected by the next component, they can be composed.

For example:

```text
Prompt output
     ↓
Model input

Model output
     ↓
Parser input
```

---

## 9. Simple mental model

Remember these three concepts:

### `Runnable`

A LangChain component that follows a common execution interface.

### `|`

Connects/composes Runnables into a pipeline.

### `invoke()`

Executes the pipeline with an input.

```text
Runnable + Runnable + Runnable
             │
             │  |
             ↓
      RunnableSequence
             │
             │ invoke()
             ↓
          Execute
```

### Final takeaway

When you see:

```python
chain = prompt | model | parser
```

think:

> **"Create a pipeline where the output of each step becomes the input of the next step."**

When you see:

```python
chain.invoke(input)
```

think:

> **"Execute that pipeline from beginning to end."**
