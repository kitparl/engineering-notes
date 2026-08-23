# Defining sets and uniqueness

- A set is an unordered collection of unique elements.
- We create a set using curly braces {} containing items separated by commas, similar to a dictionary but without key-value pairs.
- we can use the set() constructor to convert other collections into a set.

> There is one critical syntax trap: because dictionaries also use curly braces, using empty braces {} creates an empty dictionary, not a set. To create an empty set, we must use set().

```py
# Creating a set with curly braces
colors = {"red", "green", "blue", "red"}  # Duplicate "red" is ignored

# Creating a set from a list using the constructor
numbers = set([1, 2, 2, 3, 3, 4])

# The empty set trap
empty_dict = {}      # This is a dict
empty_set = set()    # This is a set

print(f"Colors: {colors}")
print(f"Numbers: {numbers}")
print(f"Type of empty_dict: {type(empty_dict)}")
print(f"Type of empty_set: {type(empty_set)}")
```

# Membership testing
- Because sets are unordered, we cannot access items by position using an index like colors[0].
- The computer does not keep track of where items are stored; it only knows they exist.
- Attempting to index a set raises TypeError.
- Similar to dictionaries, sets are optimized for membership testing.
- We use the in keyword to ask this question.
- While lists must be scanned from start to finish to find an item, sets use a technique called hashing to find items almost instantly, regardless of how large the set becomes.

```py
blocked_users = {"user_101", "user_205", "bot_99"}

user_id = "user_205"

if user_id in blocked_users:
    print(f"Access denied for {user_id}")
else:
    print(f"Welcome {user_id}")

# Attempting to index a set raises an error
# print(blocked_users[0])  # TypeError: 'set' object is not subscriptable
```

# Mathematical set operations
- Mathematical sets inspire sets in Python, which support logical operations such as union, intersection, and difference.
- These allow us to efficiently compare two groups of data.
  - Union (|): Combines elements from both sets (all unique items).
  - Intersection (&): Keeps only items found in both sets.
  - Difference (-): Keeps items in the first set but not the second.
  - Symmetric difference (^): Keeps items found in either set, but not both (opposites of intersection).

```py
kitchen_a_spices = {"salt", "pepper", "cumin", "turmeric"}
kitchen_b_spices = {"salt", "pepper", "cinnamon", "nutmeg"}

# Union: What spices do we have in total?
all_spices = kitchen_a_spices | kitchen_b_spices

# Intersection: What spices do both kitchens share?
common_spices = kitchen_a_spices & kitchen_b_spices

# Difference: What does Kitchen A have that B is missing?
unique_to_a = kitchen_a_spices - kitchen_b_spices

# Symmetric Difference: What spices are unique to each kitchen (not shared)?
distinct_spices = kitchen_a_spices ^ kitchen_b_spices

print(f"All spices: {all_spices}")
print(f"Common: {common_spices}")
print(f"Only in A: {unique_to_a}")
print(f"Distinct: {distinct_spices}")
```

# Modifying sets
- Sets are mutable
- To add elements to a set, we use the .add() method.
- .remove(item): This method deletes the specified item from the set. However, if the item is not present, Python raises a KeyError, which can interrupt program execution.
- .discard(item): This method also attempts to delete the specified item, but if the item is not found, it does nothing. Because it does not raise an error, .discard() is generally the safer choice when you are unsure whether the item exists.



