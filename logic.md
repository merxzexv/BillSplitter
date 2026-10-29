# Bill Splitter Logic

Bill splitting algorithm, with a worked example, and its implementation in python code.


---

## 1. Core Logic

### Overview

Bill Splitter settles group debts using a **Two-Pointer Approach**:

1. **Balance Calculation:** Calculate each member’s net balance from expenses paid and shares owed.
2. **Partitioning:** Arrange debtors (negative balances) on the left and creditors (positive balances) on the right.
3. **Debt Settlement:** Match debtors and creditors, transferring the minimum outstanding amount until all debts are settled.


---

## 2. Worked Example (5 People, 3 Expenses)

### Group Members

`["Alice", "Bob", "Charlie", "David", "Emma"]`

### Recorded Expenses

1. **Dinner ($100.00):** Alice pays; all 5 share equally ($20 each).
2. **Groceries ($60.00):** Bob pays; Bob, Charlie, and David share equally ($20 each).
3. **Cab ($50.00):** Charlie pays; David owes $30 and Emma owes $20.

### Net Balances Table

| Member | Total Paid | Shares Owed | Net Balance | Role |
|---|---:|---:|---:|---|
| **Alice** | $100.00 | $20.00 | **+$80.00** | Creditor |
| **Bob** | $60.00 | $40.00 | **+$20.00** | Creditor |
| **Charlie** | $50.00 | $40.00 | **+$10.00** | Creditor |
| **David** | $0.00 | $70.00 | **−$70.00** | Debtor |
| **Emma** | $0.00 | $40.00 | **−$40.00** | Debtor |

Initial unsettled list:

```python
b = [
    ["Alice", 80.00],
    ["Bob", 20.00],
    ["Charlie", 10.00],
    ["David", -70.00],
    ["Emma", -40.00]
]
```

### Step-by-Step Execution

#### Partitioning (`i = 0, j = 4`)

1. Swap Alice (+80) with Emma (−40); `j = 3`.
2. Emma is negative; increment `i = 1`.
3. Swap Bob (+20) with David (−70); `j = 2`.
4. David is negative; increment `i = 2`.
5. `i == j`; partitioning ends.

**Partitioned Array:**
- **Debtors:** `Emma (−40.00)`, `David (−70.00)`
- **Creditors:** `Charlie (+10.00)`, `Bob (+20.00)`, `Alice (+80.00)`

#### Debt Settlement (`i = 0, j = 4`)

1. Emma pays Alice **$40.00**. Emma is cleared; Alice has $40 remaining.
2. David pays Alice **$40.00**. Alice is cleared; David owes $30.
3. David pays Bob **$20.00**. Bob is cleared; David owes $10.
4. David pays Charlie **$10.00**. Both are cleared.

The pointers cross (`i >= j`), settling all debts in **4 transactions**.

---

## 3. Python Implementation

```python
members = ["Alice", "Bob", "Charlie", "David", "Emma"]
expenses = [
    ("Alice", 100, members),
    ("Bob", 60, ["Bob", "Charlie", "David"]),
    ("Charlie", 50, {"David": 30, "Emma": 20})
]

bal = dict.fromkeys(members, 0)
for p, a, s in expenses:
    bal[p] += a
    for m, v in ({m: a / len(s) for m in s} if isinstance(s, list) else s).items():
        bal[m] -= v

b = [[m, round(v, 2)] for m, v in bal.items() if abs(v) > 0.01]

# partition
i, j = 0, len(b) - 1
while i < j:
    if b[i][1] > 0:
        b[i], b[j] = b[j], b[i]
        j -= 1
    else:
        i += 1

# settlement
i, j = 0, len(b) - 1
while i < j:
    a = min(-b[i][1], b[j][1])
    print(f"{b[i][0]} pays ${a:.2f} to {b[j][0]}")
    b[i][1] += a
    b[j][1] -= a
    if abs(b[i][1]) < 0.01:
        i += 1
    if abs(b[j][1]) < 0.01:
        j -= 1
```

---
## 4. Complexity Analysis

### Time Complexity: $O(N + E \cdot M)$
- **Balance Calculation:** $O(E \cdot M)$ for $E$ expenses and $M$ average participants per split.
- **Filtering, Partitioning & Settlement:** $O(N)$ each, where $N$ is the number of members.
- **Overall:** $O(N + E \cdot M)$

### Space Complexity: $O(N)$
- **Balance Map & Unsettled Array:** $O(N)$ auxiliary space.
- **Overall:** $O(N)$
