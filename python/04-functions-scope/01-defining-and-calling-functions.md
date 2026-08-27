# What are functions

- In Python, a function is a named block of code that groups related instructions together.
- The code inside a function does not run immediately; instead, it runs only when the function is explicitly called.
- We define a function using the def keyword.
- A function must be defined before it can be called, since Python executes code from top to bottom.

`def <function_name>():`

```py

# The function is defined above, but runs here
# welcome_message() If we call this 

def welcome_message():
    """Prints a standard welcome message."""
    print("Welcome to the System.")
    print("Initializing modules...")

welcome_message()
```

> If I create a function in another Python file, how does Python decide which code is executed first—what is ‘up’ or ‘down’?
**Answer** Python executes the importing file top-to-bottom, and when it reaches an import, it executes the imported file top-to-bottom first, then continues the original file.

# Arguments vs. parameters

```py
def greet(name):      # name = parameter
    print(name)

greet("John")         # "John" = argument
```

## parameter a default value

```py
def greet(name="User"):
    print("Hello", name)
```
Now I can also call this in two ways
```py
greet("John")   # Hello John
greet()         # Hello User
```
If the caller doesn't give me a value for name, use "User" instead.

# Returning results with return
- functions usually return a value using the return statement, so the caller can use the result in further computations.
- If a function does not contain an explicit return statement, Python automatically returns the special value, None, at the end of execution.

```py
def calculate_area(length, width):
    """Calculates the area of a rectangle and returns the result."""
    area = length * width
    return area

# 1. Call the function and store the returned result in a variable
pool_area = calculate_area(length=10, width=5)

# 2. Use the result in another calculation
tile_cost = pool_area * 15

print(f"Pool Area: {pool_area} sq meters")
print(f"Total tile cost: ${tile_cost}")
```

# Function execution flow and variable scope

```py
def process_data():
    result = 20  # Defined inside the function
    return result

final_output = process_data()
print(final_output)

# ERROR: 'result' does not exist here!
print(result)
```

## Why did this fail?
The isolation of the local scope is key: variables defined inside a function cannot be accessed from outside it.

# Functions calling themselves: Recursion
Sometimes, the most elegant way to solve a complex problem is to break it down into smaller, identical problems. In programming, we achieve this through Python recursion, which occurs when a function calls itself from within its own body.

While loops allow for standard iteration, recursion provides a unique approach to repeating code, especially when dealing with hierarchical data or complex math.

However, just like an infinite loop, a recursive function will run forever if we don't tell it when to stop.

1. **The base case:** The condition that stops the recursion (the simplest instance of the problem).

2. **The recursive case:** The part where the function calls itself with a modified argument, inching closer to the base case.

```py
def countdown(n):
    if n <= 0:
        print("Liftoff!")
    else:
        print(n)
        countdown(n - 1)

countdown(3)
```

# PEP 8 Function Naming
We use these guidelines so that any other Python developer can immediately understand the purpose of our functions.

- Use lowercase letters.
- Use underscores between words (snake_case).
- Use descriptive names.
- Usually use a verb for functions.
- Avoid CamelCase.
```py
def calculate_total():
    pass
```







