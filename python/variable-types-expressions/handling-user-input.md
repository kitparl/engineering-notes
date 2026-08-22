# Capturing user text

```py
# Capturing a string input with a clear prompt
name = input("Enter your name: ")
age = input("Enter your age: ")

# Displaying the captured data
print(f"User Profile: {name}, {age}")
```

- Crucially, input() always returns a string

# Handling numeric input

```py
# Demonstrating the necessity of type conversion
str_value = input("Enter a number: ")  # User types 5
print(f"String repetition: {str_value * 3}")

# Converting input immediately for math
int_value = int(input("Enter the same number again: ")) # User types 5
print(f"Integer math: {int_value * 3}")
```

> output using print() to display information to the console
