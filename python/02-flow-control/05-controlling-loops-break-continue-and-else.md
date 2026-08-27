# Stopping early with break

```py
target = 7

print("Starting search...")

for number in range(1, 11):
    if number == target:
        print(f"Found {target}! Stopping loop.")
        break  # Exit the loop immediately
    print(f"Checked {number}...")

print("Search complete.")
```

# Skipping iterations with continue

```py
for server_id in range(0, 5):
    if server_id == 0:
        print(f"Server {server_id} is inactive. Skipping.")
        continue  # Skip the rest of this iteration
    
    # Complex processing logic would go here
    print(f"Connecting to Server {server_id}...")
    print(f"Server {server_id} maintenance complete.")
```

# The else loop construct

```py
search_limit = 5
target_value = 10  # This value is outside our range

for number in range(1, search_limit + 1):
    print(f"Checking {number}...")
    if number == target_value:
        print("Target found!")
        break
else:
    print("Loop finished: Target was not found in the range.")
```

# Reinforcement: Control in while loops

```py
attempt = 1
max_retries = 3

while attempt <= max_retries:
    print(f"Connection attempt {attempt}...")
    
    # Simulate a successful connection on the 2nd try
    if attempt == 2:
        print("Connection successful!")
        break
    
    attempt += 1
else:
    print("Failed to connect after max retries.")
```


