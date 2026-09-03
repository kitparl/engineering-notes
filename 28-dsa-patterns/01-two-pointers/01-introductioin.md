The two pointers pattern is a versatile technique used in problem-solving to efficiently traverse or manipulate sequential data structures, such as **arrays** or **linked lists**.

As the name suggests, it involves maintaining two pointers that traverse the data structure in a coordinated manner, typically starting from different positions or moving in opposite directions.

These pointers dynamically adjust based on specific conditions or criteria, allowing for the efficient exploration of the data and enabling solutions with optimal time and space complexity.

Whenever there’s a requirement to find two data elements in an array that satisfy a certain condition, the two pointers pattern should be the first strategy to come to mind.

The pointers can be used to iterate through the data structure in one or both directions, depending on the problem statement.

Eg:

1. ![image.png](./assets/d53e7c12-dd97-49fb-bf54-58a1a9a0fc68-image.png)

2. ![image.png](./assets/eb02bf86-2ade-449f-b1cb-166a160fc1cb-image.png)

3. ![image.png](./assets/6e843cfe-c7d9-42d6-981c-f2645d720f56-image.png)

4. ![image.png](./assets/72c20971-68ff-4437-8961-50a96af264c1-image.png)

5. ![image.png](./assets/b0839d41-0ff2-4edd-8039-6563fc4ba32d-image.png)

The naive approach to solving this problem would be using nested loops, with a time complexity of O(n**2)

However, by using two pointers moving toward the middle from either end, we exploit the symmetry property of palindromic strings.

This allows us to compare the elements in a single loop, making the algorithm more efficient with a time complexity of O(n)


