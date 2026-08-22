# What are functions
- a function is a named block of code that groups related instructions together. 
- The code inside a function does not run immediately; instead, it runs only when the function is explicitly called.

`def <function_name>():`

- A function must be defined before it can be called

> If I create a function in another Python file, how does Python decide which code is executed first—what is ‘up’ or ‘down’?
Answer: Python executes the importing file top-to-bottom, and when it reaches an import, it executes the imported file top-to-bottom first, then continues the original file.

# Arguments vs. parameters

```py
def greet(name):      # name = parameter
    print(name)

greet("John")         # "John" = argument
```
> Easy way to remember: parameter = placeholder, argument = actual value.

## parameter a default value

```py
def greet(name="User"):
    print("Hello", name)
```


