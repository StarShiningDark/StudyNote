# HashMap

`HashMap` 没有单独的 `contains()` 方法，而是被明确拆分成了 **`containsKey()`** 和 **`containsValue()`**。

因为数据结构的设计目的不同：

- **`Set`** 里面存的只有单个元素，所以直接用 `set.contains(element)` 判断元素在不在即可。
    
- **`Map`（如 `HashMap`）** 存储的是**键值对（Key-Value）**。如果只有一个 `contains`，系统无法确定你想查的是 Key 还是 Value，所以 Java 设计成了两个具体的方法：
    
    - **`map.containsKey(key)`**：检查某个 **Key** 是否存在（时间复杂度为 $O(1)$，最常用）。
        
    - **`map.containsValue(value)`**：检查某个 **Value** 是否存在（需要遍历整个 Map，时间复杂度为 $O(n)$）。
        

_(另外注意方法名末尾都有 `s`，是 `contains` 而不是 `contain`。)_

### 这道题为什么用 `Set` 而不用 `Map`？

在“快乐数”这道题里，核心需求仅仅是“记录之前算出来的数字是否再次出现（检测死循环）”。

- 这里每个数字只需要当成一个独立的集合元素存储，不需要给它绑定任何附加的 `Value`。
    
- 如果硬要用 `HashMap`，就必须写成 `Map<Integer, Boolean> map = new HashMap<>()`，每次存入 `map.put(n, true)`，判断时写 `map.containsKey(n)`。这不仅代码冗余，而且额外浪费了存储 Value 的内存空间。
    
- 因此，仅记录“存在性”的场景，**`HashSet` 是最合适且最省内存的数据结构**。
    

### 一个写代码的小技巧（把判断和添加合二为一）

在 Java 中，`Set.add()` 本身会返回一个 `boolean` 值：

- 如果集合里**没有**该元素，成功插入并返回 **`true`**。
    
- 如果集合里**已经存在**该元素，插入失败并返回 **`false`**。
    

利用这个特性，循环条件中的 `!record.contains(n)` 和内部的 `record.add(n)` 可以直接合并：

Java

```
class Solution {
    public boolean isHappy(int n) {
        Set<Integer> record = new HashSet<>();
        // add(n) 如果返回 false，说明出现环了，直接跳出循环
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

# Two Pointer

这个方法的思路和142 环形链表 二 那个题。

trade-off：用快慢指针省掉HashSet

**快慢指针（Floyd 判圈算法）**解决快乐数的本质，就是把数字的变换过程看成一条**“隐式单向链表”**。

每个数字经过 `getNextNumber` 计算后，只会产生唯一的一个后继数字。这就像每个节点只有一个 `next` 指针：

- **情况 1（快乐数）：** 最终一定会走到 `1`，而 $1^2 = 1$，相当于在 `1` 这里自环（`1 -> 1 -> 1...`）。
    
- **情况 2（不是快乐数）：** 最终一定会掉进一个不包含 `1` 的循环圈里（例如 $4 \to 16 \to 37 \dots \to 4$）。
    

因此，这道题和**LeetCode 141. 环形链表**在底层完全一模一样：**只要有环，快指针一定会在环内套圈追上慢指针。**

### 代码实现

- `slow` 指针每次只算一次（走 1 步）：`slow = getNextNumber(slow)`。
    
- `fast` 指针每次连续算两次（走 2 步）：`fast = getNextNumber(getNextNumber(fast))`。
    

Java

```
/**
 * Time Complexity: O(log n)
 * Space Complexity: O(1) - 仅需两个整型变量指针
 */
class Solution {
    public boolean isHappy(int n) {
        int slow = n;
        int fast = getNextNumber(n);

        // 如果 fast 到了 1，说明是快乐数，直接退出
        // 如果 slow == fast，说明快慢指针相遇入环，退出
        while (fast != 1 && slow != fast) {
            slow = getNextNumber(slow);
            fast = getNextNumber(getNextNumber(fast));
        }

        // 相遇后检查：如果是在 1 相遇说明是快乐数，否则说明掉进了死循环
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