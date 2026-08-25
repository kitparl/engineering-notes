# Numeric types

## Integers (int)


- As one of the most powerful Python data types, integers here have arbitrary precision. This means they can be as large as our computer's memory allows
- When writing large numbers, it can be difficult to count the zeros. Python allows us to use underscores (_) as visual separators
- Making Python excellent for heavy mathematical computation.

```py
# Defining integers
population = 7_900_000_000  # 7.9 billion
count = -5
large_math = 2 ** 100       # 2 to the power of 100

print(population)
print(count)
print(large_math)
```
## Floating-point numbers (float)

- Any number containing a decimal point is a floating-point number. Even if the decimal part is zero, e.g., 1.0, Python treats it as a float, not an int.
- This distinction matters because floats are stored differently in memory to handle fractional precision
- We can also define floats using scientific notation with e or E.

```py
# Defining floats
pi_approx = 3.14159
exact_float = 10.0      # Looks like an int, but the decimal makes it a float
scientific = 2.5e-3     # 2.5 times 10 to the power of -3

print(pi_approx)
print(exact_float)
print(scientific)
```
## The floating-point precision trap

- Floating-point numbers in computers are approximations. While we typically count in base-10 (decimal), computers store numbers in base-2 (binary).
- Some simple decimals, like 0.1, cannot be represented perfectly in binary fractions, just as 1/3 cannot be represented perfectly in decimal (0.3333...)
- This leads to tiny precision errors that can cause logical bugs if we expect exact equality. For example, look at the code below

```py
# The precision problem
result = 0.1 + 0.2

print(result)
```

- Managing precision with round()
- To handle these approximations, we use the round(number, ndigits) function

```py
raw_sum = 0.1 + 0.2
rounded_sum = round(raw_sum, 1)  # The precision problem

print(rounded_sum)
```





