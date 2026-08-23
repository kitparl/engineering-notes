# Adding items to a list (list.append())
- The append() method adds a single item to the absolute end.

> list.append()

- It is fast and efficient. If we need to place an item at a specific position, we use insert().
which takes an index and the value to add

```py
# Create an initial playlist
playlist = ["Track A", "Track B"]

# Add a track to the end
playlist.append("Track C")

# Insert a track at the beginning (index 0)
# Everything else shifts to the right
playlist.insert(0, "Intro")

print(playlist)
```
# Appending lists

```py
# A list of numbers
batch_1 = [1, 2, 3]
batch_2 = [4, 5]

# Appending a list treats it as a SINGLE item
batch_1.append(batch_2)

print(f"Length: {len(batch_1)}")
print(f"Content: {batch_1}")
```

What happens if we pass a list into append?
Since append adds exactly one item to the end, it treats the entire new list as that single item. This creates a "nested list" (a list inside a list).

```py
# A list of numbers
batch_1 = [1, 2, 3]
batch_2 = [4, 5]

# Appending a list treats it as a SINGLE item
batch_1.append(batch_2)

print(f"Length: {len(batch_1)}")
print(f"Content: {batch_1}")
```

# Merging lists (extend())

The manual way to solve this is by using a loop

```py
batch_1 = [1, 2, 3]
batch_2 = [4, 5]

# Naive approach: Loop and append
for number in batch_2:
    batch_1.append(number)

print(f"Merged with loop: {batch_1}")
```

Python offers a cleaner, faster way

The `extend()` method does this exact loop for us internally.

```py
batch_1 = [1, 2, 3]
batch_2 = [4, 5]

# Pythonic approach: Use extend
batch_1.extend(batch_2)

print(f"Merged with extend: {batch_1}")
```
# Organizing data
There are two common operations performed on a collection of data: sorting and reversing. Python provides built-in support for each.

- The sort() method arranges the elements.
-  To sort the list in descending order, the method can be called with the argument reverse=True.
-  In contrast, the reverse() method does not perform any comparison-based sorting. Instead, it simply reverses the order of the elements in the list, flipping them from the end to the beginning regardless of their values.

```py
scores = [88, 42, 99, 15]

# Sort in ascending order
scores.sort()
print(f"Ascending: {scores}")

# Sort in descending order
scores.sort(reverse=True)
print(f"Descending: {scores}")

# Reverse the current order (no sorting involved)
scores.reverse()
print(f"Reversed: {scores}")
```

> Both sort() and reverse() modify the list directly and return None. A common mistake is assigning the result of these methods to a new variable, such as sorted_list = my_list.sort(). This does not create a sorted list; instead, it assigns None to sorted_list. As a result, the new variable does not reference the list at all.

# Locating data

When working with lists, it is often necessary to determine where a particular item is located or how frequently it appears. Python provides two built-in methods for these purposes: index() and count()

- index(value): The index() method returns the position of the first occurrence of the value in the list and raises a ValueError if it is not present.

- count(value): The count() method, on the other hand, returns the total number of times the value appears in the list. If it is not present, the count() method returns 0.

```py
guests = ["Alice", "Bob", "Charlie", "Alice"]

# Find location of 'Bob'
bob_index = guests.index("Bob")

# Count how many times 'Alice' appears
alice_count = guests.count("Alice")

print(f"Bob is at index: {bob_index}")
print(f"Alice appears: {alice_count} times")
```
