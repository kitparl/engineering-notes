# Defining a dictionary
- A dictionary is a collection of key-value pairs.
- When learning how to create a dictionary in Python, the most common method is using curly braces {}, separating keys and values with a colon :.
- Unlike lists, dictionaries are unordered in terms of access (though Python 3.7+ preserves insertion order for display).
- We do not access items by position; we access them by their key
  - **Keys** must be unique and immutable (usually strings or numbers).
  - **Values** can be any type: strings, integers, lists, or even other dictionaries.

# Looking up data
To retrieve a value, we place the key inside square brackets [] immediately after the dictionary name.

```py
product = {"name": "Mechanical Keyboard", "price": 89.99}

# Accessing values by their keys
product_name = product["name"]
product_price = product["price"]

print(f"Item: {product_name}")
print(f"Cost: ${product_price}")
```

> If we try to access a key that does not exist, Python raises a KeyError. This strict behavior ensures we don't accidentally work with missing data. Try accessing product[brand] in the above code, and notice the output.

# Inserting and updating data
- Dictionaries are mutable, which means their contents can be modified after creation.
- Python uses the same square bracket syntax, dict[key] = value, for both adding new key–value pairs and updating existing ones.
  - If the specified key already exists, Python updates the associated value.
  - If the specified key does not exist, Python creates a new key–value pair and adds it to the dictionary
- This behavior allows dictionaries to be extended and modified dynamically as needed.

```py
user_settings = {"theme": "light", "notifications": True}

# Updating an existing key
user_settings["theme"] = "dark"

# Inserting a new key-value pair
user_settings["language"] = "English"

print(user_settings)
```

# Checking for keys
- Because attempting to access a nonexistent key in a dictionary raises a KeyError
- Because attempting to access a nonexistent key in a dictionary raises a KeyError
- This provides safe, predictable access to dictionary data.
```py
inventory = {"apples": 10, "oranges": 5}

item = "oranges"

if item in inventory:
    inventory[item] = 10

print(inventory)
```

# How to iterate over a dictionary in Python
- When we loop over a dictionary using a standard for loop, Python iterates over the keys by default.
  - **.keys()**: Returns only the values.
  - **.values()**: Returns only the values.
  - **.items()**: Returns pairs of (key, value), which allows us to access both variables directly without using [].

```py
scores = {"Alice": 88, "Bob": 92, "Charlie": 79}

# Iterating over key-value pairs
for student, score in scores.items():
    print(f"{student} scored {score}")
```








  