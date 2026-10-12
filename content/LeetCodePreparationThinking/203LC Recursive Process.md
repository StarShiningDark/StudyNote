# Recursive Process

> **Recursion is responsible for going all the way down, while returning is responsible for removing nodes and reconnecting the list.**

> **Recursion goes all the way down; returning removes nodes and reconnects the list.**

## Remove LinkedList Elements

Assume the linked list is:

```
1 -> 6 -> 3 -> null
```

Target value:

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

## ① Stage 1: Recursion — Go All the Way Down and Push onto the Stack

### Stage 1: Recursion — Go Down and Push onto the Stack

### Node `1`

Visit node `1` and first process the remaining list:

```
6 -> 3 -> null
```

Wait for the recursive result.

> Visit node `1`: recursively process `6 -> 3 -> null` and wait for the result.

---

### Node `6`

Visit node `6` and continue processing the remaining list:

```
3 -> null
```

Wait for the recursive result.

> Visit node `6`: recursively process `3 -> null` and wait for the result.

---

### Node `3`

Visit node `3` and continue processing:

```
null
```

Wait for the recursive result.

> Visit node `3`: recursively process `null` and wait for the result.

---

### Encounter `null`

Trigger the recursive base case:

```
if (head == null) {
    return null;
}
```

Return:

```
null
```

> Encounter `null`: trigger the base case and return `null`.

---

## ② Stage 2: Returning — Reconnect from Back to Front and Pop the Stack

### Stage 2: Returning — Reconnect from Back to Front and Pop the Stack

Now the recursion starts returning layer by layer:

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

### Return to Node `3`

Execute:

```
head.next = null;
```

The linked list is now:

```
3 -> null
```

Check:

```
head.val == val
3 == 6   // false
```

Node `3` does not need to be removed, so:

```
return head;
```

That is, return:

```
3 -> null
```

> `head.val (3) != 6`, so keep node `3` and return it.

---

### Return to Node `6`

The previous recursive layer returned node `3`, so:

```
head.next = 3;
```

Temporarily, we get:

```
6 -> 3 -> null
```

Then check:

```
head.val == val
6 == 6   // true
```

Node `6` needs to be removed.

Therefore, execute:

```
return head.next;
```

What gets returned here is not `6`, but:

```
3 -> null
```

So node `6` is skipped directly.

> `head.val (6) == 6`, so node `6` should be removed.  
> Return `head.next`, which is node `3`, so node `6` is skipped.

---

### Return to Node `1`

The layer for node `6` returned:

```
3 -> null
```

So:

```
head.next = 3;
```

Originally:

```
1 -> 6 -> 3
```

Now it directly becomes:

```
1 -> 3
```

Node `6` is perfectly skipped.

Then check:

```
head.val == val
1 == 6   // false
```

Therefore, keep node `1`:

```
return head;
```

> `head.val (1) != 6`, so keep node `1` and return it.

---

# ✅ Final Result

```
1 -> 3 -> null
```

**Final result:**

```
1 -> 3 -> null
```

---

## 🧠 Core Understanding

This line is the key:

```
head.next = removeElements(head.next, val);
```

It does not simply mean:

> “Make `head.next` point to the next node.”

Instead, it means:

> **First process and clean up the remaining linked list, then reconnect the processed new list back to the current node.**

Then:

```
return head.val == val ? head.next : head;
```

determines whether the current node should be kept:

```
current node == val
        ↓
   return head.next
   skip current node

current node != val
        ↓
     return head
     keep current node
```