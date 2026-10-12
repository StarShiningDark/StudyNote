# Recursive Process

> **递的时候负责走到底，归的时候负责删节点和重新接线。**

> **Recursion goes all the way down; returning removes nodes and reconnects the list.**

## Remove LinkedList Elements


假设链表是：

```
1 -> 6 -> 3 -> null
```

目标值：

```
val = 6
```

**Assume the linked list is:**

```
1 -> 6 -> 3 -> null
```

**Target value:**

```
val = 6
```

---

## ① 第一阶段：递 —— 一路向下，压栈

### Stage 1: Recursion — Go Down and Push onto the Stack

### 节点 `1`

访问节点 `1`，先去处理后面的：

```
6 -> 3 -> null
```

等待递归结果。

> Visit node `1`: recursively process `6 -> 3 -> null` and wait for the result.

---

### 节点 `6`

访问节点 `6`，继续处理后面的：

```
3 -> null
```

等待递归结果。

> Visit node `6`: recursively process `3 -> null` and wait for the result.

---

### 节点 `3`

访问节点 `3`，继续处理：

```
null
```

等待递归结果。

> Visit node `3`: recursively process `null` and wait for the result.

---

### 遇到 `null`

触发递归终止条件：

```
if (head == null) {
    return null;
}
```

返回：

```
null
```

> Encounter `null`: trigger the base case and return `null`.

---

## ② 第二阶段：归 —— 从后往前重新连接，弹栈

### Stage 2: Returning — Reconnect from Back to Front and Pop the Stack

现在递归开始一层一层返回：

```
null
 ↑
 3
 ↑
 6
 ↑
 1
```

---

### 回到节点 `3`

执行：

```
head.next = null;
```

此时链表：

```
3 -> null
```

检查：

```
head.val == val
3 == 6   // false
```

节点 `3` 不需要删除，因此：

```
return head;
```

也就是返回：

```
3 -> null
```

> `head.val (3) != 6`, so keep node `3` and return it.

---

### 回到节点 `6`

上一层返回的是节点 `3`，所以：

```
head.next = 3;
```

暂时得到：

```
6 -> 3 -> null
```

接着检查：

```
head.val == val
6 == 6   // true
```

节点 `6` 需要被删除。

因此执行：

```
return head.next;
```

这里返回的不是 `6`，而是：

```
3 -> null
```

于是节点 `6` 被直接跳过。

> `head.val (6) == 6`, so node `6` should be removed.  
> Return `head.next`, which is node `3`, so node `6` is skipped.

---

### 回到节点 `1`

节点 `6` 那一层返回的是：

```
3 -> null
```

所以：

```
head.next = 3;
```

原本：

```
1 -> 6 -> 3
```

现在直接变成：

```
1 -> 3
```

节点 `6` 被完美跳过。

接着检查：

```
head.val == val
1 == 6   // false
```

因此保留节点 `1`：

```
return head;
```

> `head.val (1) != 6`, so keep node `1` and return it.

---

# ✅ 最终结果

```
1 -> 3 -> null
```

**Final result:**

```
1 -> 3 -> null
```

---

## 🧠 核心理解

这一句最关键：

```
head.next = removeElements(head.next, val);
```

它的意思不是简单地：

> “让 `head.next` 指向下一个节点。”

而是：

> **先把后面的链表处理干净，再把处理后的新链表接回来。**

然后：

```
return head.val == val ? head.next : head;
```

决定当前节点是否留下：

```
当前节点 == val
        ↓
   返回 head.next
   跳过当前节点

当前节点 != val
        ↓
     返回 head
     保留当前节点
```

