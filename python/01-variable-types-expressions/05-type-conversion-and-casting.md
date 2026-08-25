- Python is strict about data types. It will not allow us to add a string of text to a number, nor will it intuitively know how to treat the word "True" as a boolean logic value. (**imp**)
- In certain obvious situations, Python handles the conversion for us. This is called implicit conversion or coercion.
- The most common scenario involves integers and floats. If we perform arithmetic mixing an integer (whole number) and a float (decimal), Python automatically promotes the integer to a float to prevent data loss.

```py
# Mixing integers and floats
integer_score = 50
bonus_multiplier = 1.5

# Python promotes integer_score to a float for the calculation
total_score = integer_score * bonus_multiplier

print(total_score)
print(type(total_score))
```

# Explicit conversion (casting)

- When the conversion is not obvious, like adding text to a number, Python raises a TypeError.

```py
# This raises a TypeError
print("Score: " + 100)
```
- we perform explicit conversion, also known as casting
- We use Python's built-in constructor functions to manually transform data: str(), int(), float(), and bool()

# Converting to strings

```py
user_id = 4096
# We convert the integer to a string to concatenate it
message = "User ID: " + str(user_id)

print(message)
print(type(message))
```

# Converting to numbers

```py
price_tag = "19.99"
qty_string = "5"

# Converting strings to numbers for math
price = float(price_tag)
quantity = int(qty_string)

total = price * quantity
print(total)

# optional: If I am trying to convert into the int(price_tag) getting an error because as I already mentioned python is strict with data types.
```

# Data loss: Floats to integers

- Converting a float to an integer using int() does not round the number. It truncates it.

```py
exact_value = 9.99

truncated = int(exact_value) # int() chops off the decimal
rounded = round(exact_value) # round() finds the nearest whole number

print(truncated)
print(rounded)
```
# The "Round half to even" rule

- It is important to note that Python's round() function uses a strategy called round half to even (or Banker's Rounding)
- If a number is exactly halfway between two integers (like 2.5), Python rounds to the nearest even number

```py
# Banker's rounding examples
print("Round 2.5:", round(2.5))
print("Round 3.5:", round(3.5))
```

# The boolean cast

- In Python, every value (not just booleans) has an inherent "truth" to it.
- This concept is known as truthy and falsy.

### Falsy: 
Values that represent "emptiness" or "nothing" convert to False. This includes the number 0, the float 0.0, empty strings "", and the special object None.

### Truthy:
Almost everything else converts to True. This includes any non-zero number (even negatives) and any string with at least one character.

```py
# Numbers
print(f"0 is: {bool(0)}")
print(f"100 is: {bool(100)}")

# Strings
print(f"Empty string is: {bool('')}")
print(f"Space string is: {bool(' ')}")  # Note the space inside
print(f"Text string is: {bool('Python')}")

# None
print(f"None is: {bool(None)}")
```

## Common pitfalls: Mixed type

- Casting is not magic. The source data must be valid for the target type
- If we try to cast a string that doesn't look like a number into an integer, Python will stop execution with a ValueError.

```py
bad_data = "100 USD" # This string contains non-numeric characters

conversion = int(bad_data) # This line causes a crash
```












