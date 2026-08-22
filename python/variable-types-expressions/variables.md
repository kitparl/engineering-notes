# How to create a variable

`<name> = <value>`

```py
# Creating variables by assigning values
current_score = 100
player_name = "Alex"

print(current_score)
print(player_name)
```
- Python variables do not store values directly; they store references to objects
- When one variable is assigned to another, Python does not create a copy of the underlying data. Rather, it assigns an additional name to the same object, effectively allowing multiple variables to refer to a single object.
- Python does not have primitives in the same sense as languages like Java, C, or C++.
- We can verify this via the built-in id() function, which returns the unique memory identifier of an object.

| Value       | Object type    |
| ----------- | -------------- |
| `10`        | `int` object   |
| `"Alice"`   | `str` object   |
| `3.14`      | `float` object |
| `True`      | `bool` object  |
| `[1, 2, 3]` | `list` object  |

```py
x = 10

print(type(x))
print(isinstance(x, object))
```

- Even functions, classes, and types are objects in Python:

```py
def hello():
    pass

print(type(hello))
```

- Everything in Python is an object, and variables are names that refer to objects.
  
  
