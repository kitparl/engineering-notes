- Standard math: Addition (+), subtraction (-), and multiplication (*) work exactly as expected.

- Exponents: To raise a number to a power (like squaring or cubing), use the double asterisk (**).

- True Division (/): Always returns a float (decimal), even if the result is a whole number (e.g., 5 / 2 is 2.5).

- Floor Division (//): Divides and rounds down to the nearest whole integer (e.g., 5 // 2 is 2).

eg: Floor division // rounds down toward negative infinity. Since -2.5 is the exact result, the next lower integer is -3.


- Modulo (%): Returns the remainder of the division (e.g., 5 % 2 is 1). This is useful for checking if numbers are even/odd or cycling through values.

```py
# 1. Standard Math & Exponents
price = 100
tax = price * 0.20           # Multiplication
total = price + tax - 5      # Addition & Subtraction

side = 4
area = side ** 2             # Exponent: 4 squared is 16

# 2. The Three Divisions
total_seconds = 130
minutes = total_seconds // 60 # Floor: 2 whole minutes
seconds = total_seconds % 60  # Modulo: 10 remaining seconds
precise = total_seconds / 60  # True: 2.166...

print(f"Total Cost: {total}")
print(f"Time: {minutes} min, {seconds} sec")
```

# Comparison operators

- Equality: == (note the double equals; single = is for the assignment.

- Inequality: != (checks if values are not equal).

- Ordering: <, >, <=, >=.

```py
current_score = 105
high_score = 100

# Check relationships
is_new_record = current_score > high_score   # True
is_qualifying = current_score >= 100         # True (Greater or Equal)
is_exact_tie  = current_score == high_score  # False

print(f"New Record: {is_new_record}")
print(f"Qualified: {is_qualifying}")
```

# Logical operators

- and: Returns True only if both operands are true.

- or: Returns True if at least one operand is true.

- not: Inverts the value (True becomes False, and vice versa).

```py
has_ticket = True
is_vip = False
has_id = True

# AND: Both must be true
can_enter = has_ticket and has_id

# OR: Only one needs to be true
has_access = is_vip or has_ticket

# NOT: Inverts the state
is_banned = not has_access

print(f"Can Enter: {can_enter}")
print(f"Has Access: {has_access}")
```

# Operator precedence and grouping

| Priority | Operator Type             | Operators           |
| -------: | ------------------------- | ------------------- |
|        1 | Parentheses               | `()`                |
|        2 | Exponentiation            | `**`                |
|        3 | Multiplication / Division | `*`, `/`, `//`, `%` |
|        4 | Addition / Subtraction    | `+`, `-`            |
|        5 | Comparison                | `==`, `>`, etc.     |
|        6 | Logical                   | `not`, `and`, `or`  |


```py
# Default: Multiplication (*) happens before Addition (+)
result_default = 10 + 5 * 2   # 10 + 10 = 20

# Grouped: Parentheses force Addition (+) first
result_grouped = (10 + 5) * 2 # 15 * 2 = 30

print(f"Default: {result_default}")
print(f"Grouped: {result_grouped}")
```





