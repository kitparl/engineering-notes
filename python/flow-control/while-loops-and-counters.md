# The structure of a while loop

```py
current_temperature = 25
target_temperature = 28

# Keep heating until the target is reached
while current_temperature < target_temperature:
    print(f"Heating... Current temp: {current_temperature}")
    current_temperature += 1  # Change the condition variable

print(f"Target reached: {current_temperature}")
```

# Controlled iteration with counters

```py
attempts = 1
max_retries = 3

while attempts <= max_retries:
    print(f"Connection attempt {attempts} of {max_retries}...")
    # Simulate connection logic here
    attempts += 1

print("Finished attempts.")
```

# The infinite loop pitfal

- The most common error with while loops is the infinite loop.
