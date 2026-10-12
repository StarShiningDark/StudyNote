# 1、HashMap

`HashMap` does not have a standalone `contains()` method. Instead, it is explicitly divided into **`containsKey()`** and **`containsValue()`**.

This is because the data structures are designed for different purposes:

- A **`Set`** stores only individual elements, so you can simply use `set.contains(element)` to check whether an element exists.
- A **`Map`** (such as `HashMap`) stores **key-value pairs**. If there were only one `contains` method, the system would not know whether you wanted to search for a Key or a Value. Therefore, Java provides two specific methods:
    - **`map.containsKey(key)`**: checks whether a specific **Key** exists (time complexity: O(1), and this is the most commonly used one).
    - **`map.containsValue(value)`**: checks whether a specific **Value** exists (requires traversing the entire Map, so the time complexity is O(n)).

_(Also note that the method name ends with `s`: it is `contains`, not `contain`.)_

### Why does this problem use `Set` instead of `Map`?

In the "Happy Number" problem, the core requirement is simply to "record whether a previously calculated number appears again in order to detect an infinite loop."

- Each number only needs to be stored as an independent element in a collection. There is no need to associate it with any additional `Value`.
- If you force the use of a `HashMap`, you would need to write `Map<Integer, Boolean> map = new HashMap<>()`, store values using `map.put(n, true)`, and check them using `map.containsKey(n)`. This not only makes the code more redundant, but also wastes additional memory for storing the Value.
- Therefore, when you only need to record "whether something exists," **`HashSet` is the most appropriate and memory-efficient data structure**.

### A Small Coding Trick: Combine Checking and Adding

In Java, `Set.add()` itself returns a `boolean` value:

- If the element **does not exist** in the set, it is successfully inserted and **`true`** is returned.
- If the element **already exists** in the set, the insertion fails and **`false`** is returned.

Using this behavior, `!record.contains(n)` in the loop condition and `record.add(n)` inside the loop can be combined directly:

```
class Solution {
    public boolean isHappy(int n) {
        Set<Integer> record = new HashSet<>();
        // If add(n) returns false, it means a cycle has been detected, so exit the loop directly
        while (n != 1 && record.add(n)) {
            n = getNextNumber(n);
        }
        return n == 1;
    }

    private int getNextNumber(int n) {
        int res = 0;
        while (n > 0) {
            int temp = n % 10;
            res += temp * temp;
            n = n / 10;
        }
        return res;
    }
}

```

# 2、Two Pointer

The idea behind this approach is similar to **LeetCode 142. Linked List Cycle II**.

Trade-off: use fast and slow pointers to eliminate the need for a `HashSet`.

The essence of using the **fast and slow pointer technique (Floyd’s Cycle Detection Algorithm)** to solve the Happy Number problem is to treat the number transformation process as an **“implicit singly linked list.”**

After each number is processed by `getNextNumber`, it produces exactly one unique successor number. This is similar to each node having exactly one `next` pointer:

- **Case 1 (Happy Number):** The sequence will eventually reach `1`, and since 1^2 = 1, it forms a self-loop at `1` (`1 -> 1 -> 1...`).
- **Case 2 (Not a Happy Number):** The sequence will eventually fall into a cycle that does not contain `1` (for example, 4 \to 16 \to 37 \dots \to 4).

Therefore, this problem is fundamentally the same as **LeetCode 141. Linked List Cycle**: **as long as there is a cycle, the fast pointer will eventually catch up with the slow pointer inside the cycle.**

### Code Implementation

- The `slow` pointer performs the calculation once each time (moves 1 step): `slow = getNextNumber(slow)`.
- The `fast` pointer performs the calculation twice each time (moves 2 steps): `fast = getNextNumber(getNextNumber(fast))`.

```
/**
 * Time Complexity: O(log n)
 * Space Complexity: O(1) - Only two integer pointer variables are needed
 */
class Solution {
    public boolean isHappy(int n) {
        int slow = n;
        int fast = getNextNumber(n);

        // If fast reaches 1, it means the number is happy, so exit directly
        // If slow == fast, it means the fast and slow pointers meet inside a cycle, so exit
        while (fast != 1 && slow != fast) {
            slow = getNextNumber(slow);
            fast = getNextNumber(getNextNumber(fast));
        }

        // After they meet, check the result: if fast is 1, it is a happy number; otherwise, it has fallen into an infinite cycle
        return fast == 1;
    }

    private int getNextNumber(int n) {
        int res = 0;
        while (n > 0) {
            int temp = n % 10;
            res += temp * temp;
            n /= 10;
        }
        return res;
    }
}
```