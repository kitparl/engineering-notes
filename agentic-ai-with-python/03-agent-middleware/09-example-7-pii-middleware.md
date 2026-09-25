built-in middleware classes provided by LangChain for agents

| Middleware                  | Purpose                                                                       |
| --------------------------- | ----------------------------------------------------------------------------- |
| `HumanInTheLoopMiddleware`  | Pause execution and ask a human for approval before sensitive tool calls      |
| `SummarizationMiddleware`   | Automatically summarize conversation history when context gets too large      |
| `PIIMiddleware`             | Detect and handle PII such as emails, phone numbers, or credit-card-like data |
| `ModelCallLimitMiddleware`  | Limit how many times the model can be called                                  |
| `ToolCallLimitMiddleware`   | Limit how many times tools can be called                                      |
| `ModelFallbackMiddleware`   | Fall back to another model when the primary model fails                       |
| `ModelRetryMiddleware`      | Retry failed model calls                                                      |
| `ToolRetryMiddleware`       | Retry failed tool calls                                                       |
| `LLMToolSelectorMiddleware` | Use an LLM to select relevant tools                                           |
| `ToModelMessagesMiddleware` | Transform/intercept agent state before the model call                         |
| `ContextEditingMiddleware`  | Modify/edit conversation context before it reaches the model                  |
| `ShellToolMiddleware`       | Middleware for controlling shell-tool execution                               |
| `TodoListMiddleware`        | Gives an agent todo/task-list capabilities                                    |
| `FilesystemMiddleware`      | Provides filesystem-oriented agent capabilities                               |
| `SkillsMiddleware`          | Allows agents to discover/use predefined skills                               |
| `ModelCallLimitMiddleware`  | Enforces model-call budgets                                                   |
| `ToolCallLimitMiddleware`   | Enforces tool-call budgets                                                    |
