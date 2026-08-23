1. How does grouping with parentheses change the logic in the following cases?

# Case A:
if not A and B: ...

Python evaluates this as: if (not A) and B:

# Case B:
if not (A and B): ...

not applies to the entire expression A and B.

2. What happens if the start value is greater than the stop value in range() with a positive step?

It prints nothing.

The logic is “start at 10, keep going while < 5”. Since 10 is not < 5, the body never executes.




