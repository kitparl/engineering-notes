# Iterating over sequences

```py
message = "Code"

# The variable 'char' automatically takes the value of each letter
for char in message:
    print(f"Current letter: {char}")

print("Loop finished.")
```

# Generating sequences with range()

- The syntax relies on the in keyword. We define a temporary variable (the loop variable) that Python automatically updates with the next item in the collection at the start of every iteration.
- Python provides the built-in function range() to generate a sequence of numbers on the fly.

### The basic range(<stop>)

```py
# Generates numbers: 0, 1, 2
for number in range(3):
    print(f"Iteration {number}")
```

### Controlling boundaries: start, stop, and step

- For more control, range() accepts up to three arguments: start, stop, and step.
- By default, range() begins counting from 0. If we want to generate a sequence that starts at a different value, such as counting from 1 to 5, we can provide two arguments in the form range(start, stop). In this case, the start value is inclusive, while the stop value is exclusive.

#### start and stop

- By default, range() begins counting from 0
-  we can provide two arguments in the form range(start, stop)

```py
# Starts at 1, stops before 6
for i in range(1, 6):
    print(f"Counting: {i}")
```

#### The step value

- The third argument of the range() function, known as step, determines the increment between consecutive values in the sequence
- By default, the step value is 1, meaning the sequence increases by one each time

```py
# Counts 0, 2, 4, 6, 8 (stops before 10)
for number in range(0, 10, 2):
    print(f"Even number: {number}")
```

## Reverse iteration

```py
# Countdown: 5, 4, 3, 2, 1 (stops before 0)
for count in range(5, 0, -1):
    print(f"T-minus {count}")
```

# The repeat pattern
Sometimes we need to run a block of code a specific number of times, but we do not actually use the loop variable inside the block. In Python, it is a common convention to name such an unused loop variable _ (underscore).

```py
attempts = 3

# We just want to repeat the print statement 3 times
for _ in range(attempts):
    print("Connection failed. Retrying...")
```
# The accumulator pattern

```py
total = 0

# Sum numbers from 1 to 10
for number in range(1, 11):
    total = total + number  # Add current number to running total
    print(f"Added {number}, new total is {total}")

print(f"Final Sum: {total}")
```






