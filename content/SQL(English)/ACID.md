# 💾 Database Transactions

## 1. Four Properties of Transactions: ACID

### ① Atomicity

A transaction is treated as a single unit of work.

All operations inside the transaction must either:

- **all succeed**, or
    
- **all fail and roll back**
    

There is no partially completed transaction.

#### Example

Suppose A transfers `$100` to B:

```
A: -$100
B: +$100
```

These two operations must succeed or fail together.

If money is deducted from A but cannot be added to B, the entire transaction must be rolled back.

> **Atomicity = All or Nothing**

---

### ② Consistency

A transaction must move the database from one **valid state** to another valid state.

Database rules and integrity constraints must remain satisfied before and after the transaction.

#### Example

Before the transfer:

```
A = $500
B = $500

Total = $1000
```

After A transfers `$100` to B:

```
A = $400
B = $600

Total = $1000
```

The total amount remains unchanged.

> **Consistency = Data rules remain valid**

---

### ③ Isolation

Multiple transactions may execute concurrently, but they should not improperly interfere with each other.

The intermediate state of one transaction should not unexpectedly affect another transaction.

#### Example

Suppose two transactions modify the same bank account at the same time.

The database must use mechanisms such as:

- transaction isolation levels
    
- locks
    
- MVCC
    

to ensure that concurrent operations do not produce incorrect results.

> **Isolation = Concurrent transactions should behave as if they are properly separated**

---

### ④ Durability

Once a transaction is successfully committed, its changes must be permanently stored.

Even if the system crashes afterward, the committed data should not be lost.

#### Example

```
COMMIT;
```

After the transaction is committed, the changes must survive:

```
Application crash
Database restart
System reboot
```

> **Durability = Committed data survives**

---

# 2. Transaction Concurrency Problems

When multiple transactions execute concurrently, several problems may occur.

---

## ① Dirty Read

A **dirty read** happens when one transaction reads data modified by another transaction **before that transaction commits**.

### Example

Transaction B:

```
Balance = 1000
↓
Update Balance = 500
↓
Not committed yet
```

Transaction A reads:

```
Balance = 500
```

Then transaction B performs:

```
ROLLBACK
```

The real balance returns to:

```
1000
```

But transaction A already read `500`.

Therefore, A read **dirty data**.

### Root Cause

```
Transaction A reads
        ↓
uncommitted data
        ↓
from Transaction B
```

> **Dirty Read = Read uncommitted data**

---

## ② Non-repeatable Read

A **non-repeatable read** occurs when the same transaction reads the **same row multiple times**, but gets different values.

This happens because another transaction modifies and commits that row between the two reads.

### Example

Transaction A:

```
SELECT balance FROM account WHERE id = 1;
```

Result:

```
1000
```

Transaction B executes:

```
UPDATE account
SET balance = 500
WHERE id = 1;

COMMIT;
```

Transaction A executes the same query again:

```
SELECT balance FROM account WHERE id = 1;
```

Result:

```
500
```

Within the same transaction:

```
First read  → 1000
Second read → 500
```

### Focus

Non-repeatable reads mainly involve:

```
UPDATE
```

The **same row** changes between two reads.

> **Non-repeatable Read = Same row, different values**

---

## ③ Phantom Read

A **phantom read** occurs when the same transaction performs the same range query multiple times, but the **set of rows changes**.

Another transaction may insert or delete rows that satisfy the query condition.

### Example

Transaction A:

```
SELECT *
FROM users
WHERE age > 20;
```

Result:

```
10 rows
```

Transaction B inserts:

```
INSERT INTO users(...)
VALUES (...);

COMMIT;
```

The new row also satisfies:

```
age > 20
```

Transaction A executes the same query again:

```
SELECT *
FROM users
WHERE age > 20;
```

Now:

```
11 rows
```

A new row appears as if it were a **phantom**.

### Focus

Phantom reads mainly involve:

```
INSERT
DELETE
```

and **range queries**.

> **Phantom Read = Same query condition, different set of rows**

---

# 3. Quick Comparison

|Problem|What Changes?|Typical Cause|
|---|---|---|
|**Dirty Read**|Value may not even be committed|Reading uncommitted data|
|**Non-repeatable Read**|Existing row changes|`UPDATE`|
|**Phantom Read**|Number/set of matching rows changes|`INSERT` / `DELETE`|

### Easy Way to Remember

```
Dirty Read
→ Read something that may disappear after rollback

Non-repeatable Read
→ Same row, different value

Phantom Read
→ Same condition, different rows
```

---

# 4. Transaction Isolation Levels

SQL defines four common transaction isolation levels.

From weakest to strongest:

```
Read Uncommitted
        ↓
Read Committed
        ↓
Repeatable Read
        ↓
Serializable
```

As isolation becomes stronger:

```
Data consistency ↑
Concurrency      ↓
Performance cost ↑
```

---

## Isolation Level Comparison

|Isolation Level|Description|Dirty Read|Non-repeatable Read|Phantom Read|
|---|---|---|---|---|
|**Read Uncommitted**|Can read uncommitted changes from other transactions|❌ Possible|❌ Possible|❌ Possible|
|**Read Committed**|Only committed data can be read|✅ Prevented|❌ Possible|❌ Possible|
|**Repeatable Read**|Repeated reads of the same row within a transaction remain consistent|✅ Prevented|✅ Prevented|⚠️ DB-dependent|
|**Serializable**|Transactions behave as if they execute sequentially|✅ Prevented|✅ Prevented|✅ Prevented|

---

# 5. Read Uncommitted

This is the **lowest isolation level**.

One transaction may read data modified by another transaction even if the modification has not been committed.

Example:

```
Transaction B
UPDATE balance = 500
        ↓
Not committed

Transaction A
        ↓
Can already read 500
```

Possible problems:

```
Dirty Read
Non-repeatable Read
Phantom Read
```

> **Read Uncommitted = Almost no read isolation**

---

# 6. Read Committed

A transaction can only read data that has already been committed.

Therefore:

```
Dirty Read ✅ Prevented
```

However, another transaction may modify and commit the same row between two reads.

Therefore:

```
Non-repeatable Read ❌ Still possible
Phantom Read        ❌ Still possible
```

### Example

```
A reads value = 100

B updates value = 200
B commits

A reads again = 200
```

This is the default isolation level for some databases, such as Oracle and PostgreSQL.

> **Read Committed = Only read committed data**

---

# 7. Repeatable Read ⭐

`Repeatable Read` ensures that repeated reads of the same data within one transaction remain consistent.

In standard SQL:

```
Dirty Read           ✅ Prevented
Non-repeatable Read  ✅ Prevented
Phantom Read         ⚠️ May still occur
```

---

## MySQL InnoDB

For MySQL InnoDB:

```
Default isolation level = Repeatable Read
```

InnoDB combines:

```
MVCC
+
Record Locks
+
Gap Locks
+
Next-Key Locks
```

to provide stronger behavior than basic SQL `Repeatable Read`.

---

## MVCC

**MVCC = Multi-Version Concurrency Control**

For ordinary snapshot reads:

```
SELECT ...
```

InnoDB can let a transaction see a consistent snapshot of the database.

Conceptually:

```
Transaction A starts
        ↓
Sees snapshot V1

Transaction B changes data
        ↓
Commits V2

Transaction A reads again
        ↓
Still sees V1
```

Therefore repeated snapshot reads remain consistent.

---

## Next-Key Lock

For locking reads such as:

```
SELECT ...
FOR UPDATE;
```

InnoDB may use a **Next-Key Lock**.

A Next-Key Lock is approximately:

```
Record Lock
+
Gap Lock
```

It locks:

```
existing index record
+
gap around/before the record
```

This can prevent another transaction from inserting new records into the locked range.

### Example

Suppose an index contains:

```
10     20     30
```

A range lock may conceptually protect:

```
(10, 20]
```

Another transaction cannot freely insert a value such as:

```
15
```

into that locked range.

This helps prevent phantom rows during locking reads.

> **Next-Key Lock = Record Lock + Gap Lock**

---

# 8. Serializable

`Serializable` is the strongest standard isolation level.

Transactions behave as though they execute one after another.

Conceptually:

```
Transaction A
    ↓
finish
    ↓
Transaction B
    ↓
finish
```

It prevents:

```
Dirty Read
Non-repeatable Read
Phantom Read
```

However, stronger isolation usually means:

```
Concurrency ↓
Lock contention ↑
Performance cost ↑
```

> **Serializable = Highest isolation, lowest concurrency**

---

# 9. Interview Memory Table

|Level|Dirty Read|Non-repeatable Read|Phantom Read|
|---|---|---|---|
|**Read Uncommitted**|❌|❌|❌|
|**Read Committed**|✅|❌|❌|
|**Repeatable Read**|✅|✅|⚠️|
|**Serializable**|✅|✅|✅|

Where:

```
✅ = Prevented
❌ = Possible
```

For **MySQL InnoDB Repeatable Read**, remember:

> **MVCC handles consistent snapshot reads, while Next-Key Locks help prevent phantom rows in locking reads.**

---

# 10. Spring Transaction Configuration

In Spring, the transaction isolation level can be configured using `@Transactional`.

Example:

```
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void updateAccount() {
    // business logic
}
```

Common options include:

```
Isolation.READ_UNCOMMITTED
Isolation.READ_COMMITTED
Isolation.REPEATABLE_READ
Isolation.SERIALIZABLE
```

You can also use:

```
Isolation.DEFAULT
```

to use the database's default transaction isolation level.

---

# 🧠 Final Memory Trick

### ACID

```
A → Atomicity
    All or nothing

C → Consistency
    Valid state → Valid state

I → Isolation
    Concurrent transactions do not improperly interfere

D → Durability
    Commit means permanent
```

### Concurrency Problems

```
Dirty Read
→ Read uncommitted data

Non-repeatable Read
→ Same row, different value

Phantom Read
→ Same condition, different rows
```

### Isolation Levels

```
RU → Dirty Read possible

RC → No Dirty Read

RR → No Dirty Read
     No Non-repeatable Read

Serializable
→ Prevent all three
```

### MySQL Interview Keyword

```
Repeatable Read
        ↓
MVCC
+
Next-Key Lock
        ↓
Record Lock + Gap Lock
```

 