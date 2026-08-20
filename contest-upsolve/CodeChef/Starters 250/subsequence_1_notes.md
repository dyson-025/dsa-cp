# Subsequence 1

## 1. Problem

For an array `A`, define:

```text
f(A) = largest L such that [1, 2, ..., L]
       is a subsequence of A
```

For example:

```text
A = [4, 1, 2, 1, 3]
```

We can find:

```text
[1, 2, 3]
```

as a subsequence, so:

```text
f(A) = 3
```

because `[1,2,3,4]` is not a subsequence.

---

We are given an array `A`.

We can split it into any number of contiguous subarrays:

```text
A = A1 + A2 + ... + AK
```

where `+` means concatenation.

We need to maximize:

```text
f(A1) + f(A2) + ... + f(AK)
```

We are free to choose `K`.

---

# 2. Understanding f(A)

The function `f(A)` asks:

> How far can I build the sequence `1,2,3,...` as a subsequence?

For example:

```text
A = [2, 1, 1, 3]
```

We can select:

```text
1 → 3
```

but cannot find `1 → 2 → 3` in the correct order.

So:

```text
f(A) = 1
```

Another example:

```text
A = [1, 2, 4, 1, 3]
```

We can choose:

```text
1 → 2 → 3
```

so:

```text
f(A) = 3
```

---

# 3. Important Observation

Suppose we are processing the original array from left to right.

Whenever we see:

```text
1
```

we can start a new subsequence:

```text
[1]
```

Whenever we see:

```text
2
```

we can extend a segment that currently has:

```text
f = 1
```

to:

```text
[1,2]
```

Whenever we see:

```text
3
```

we can extend a segment having:

```text
f = 2
```

to:

```text
[1,2,3]
```

and so on.

This suggests maintaining the best answer for segments ending at each possible value.

---

# 4. DP State

Let:

```text
dp[x]
```

represent:

> The maximum total value obtained so far if the current/last active segment has `f = x`.

We use:

```text
dp[0] = 0
```

initially.

Why?

Before starting any segment, our current progress is effectively `0`.

---

# 5. Processing a `1`

Suppose the current element is:

```text
x = 1
```

A `1` can always start a new segment:

```text
[1]
```

whose:

```text
f = 1
```

So:

```cpp
dp[1] = max(dp[1], best + 1);
```

where:

```text
best = maximum dp[x] seen so far
```

### Why `best + 1`?

Suppose the previous best total is:

```text
best = 5
```

We can cut before this `1` and start a new segment:

```text
... | [1]
```

The new segment contributes:

```text
f([1]) = 1
```

so the total becomes:

```text
5 + 1 = 6
```

---

# 6. Processing `x > 1`

Suppose:

```text
x = 3
```

To make `f = 3`, the current segment must already have:

```text
[1,2]
```

and then this `3` extends it to:

```text
[1,2,3]
```

Therefore, we need a state with:

```text
dp[2]
```

and transition:

```cpp
dp[3] = max(dp[3], dp[2] + 1);
```

More generally:

```cpp
dp[x] = max(dp[x], dp[x-1] + 1);
```

if `dp[x-1]` is reachable.

---

# 7. Why `dp[x-1] + 1`?

Suppose:

```text
dp[2] = 4
```

This means we have a way to partition the processed prefix such that the current segment has:

```text
f = 2
```

Now we encounter:

```text
x = 3
```

We can append `3` to that current segment.

Its value changes:

```text
f = 2 → f = 3
```

Therefore the total answer increases by `1`:

```text
dp[3] = dp[2] + 1
```

---

# 8. What About Values That Cannot Extend?

Suppose:

```text
x = 4
```

but there is no valid state with:

```text
dp[3]
```

Then this `4` cannot extend the current segment into:

```text
[1,2,3,4]
```

So we simply ignore it.

```cpp
if (dp[x-1] != INT_MIN)
    dp[x] = max(dp[x], dp[x-1] + 1);
```

---

# 9. Why Do We Need `best`?

We need `best` when we see a `1`.

A `1` can start a completely new segment after **any optimal partition of the previous prefix**.

So:

```text
best = max(dp[0], dp[1], dp[2], ...)
```

represents the best answer we have obtained so far.

Then:

```cpp
dp[1] = max(dp[1], best + 1);
```

means:

> Take the best partition of everything before this `1`, cut before the `1`, and start a new segment `[1]`.

---

# 10. The Crucial Idea

The problem allows us to choose **where to split** the array.

For every `1`, we have the option:

```text
previous segments | [1]
```

For every `x > 1`, we have the option:

```text
... [1,2,...,x-1] + x
```

So there are only two meaningful actions:

```text
x = 1
    → start a new segment

x > 1
    → extend a segment whose current f is x-1
```

This reduces the partition problem to a small DP.

---

# 11. Dry Run

Consider:

```text
A = [1, 2, 1]
```

Expected answer:

```text
3
```

The optimal partition is:

```text
[1,2] + [1]
```

because:

```text
f([1,2]) = 2
f([1])   = 1

total = 3
```

---

## Start

```text
dp = [0, -INF, -INF, ...]
best = 0
```

---

## Process `1`

```text
x = 1
```

Start a new segment:

```text
dp[1] = best + 1
      = 0 + 1
      = 1
```

Update:

```text
best = 1
```

Now:

```text
dp[1] = 1
```

---

## Process `2`

We need `dp[1]`.

```text
dp[2] = dp[1] + 1
      = 1 + 1
      = 2
```

Update:

```text
best = 2
```

Now the current segment effectively represents:

```text
[1,2]
```

and contributes `2`.

---

## Process `1`

A `1` starts another segment.

```text
dp[1] = max(dp[1], best + 1)
      = max(1, 2 + 1)
      = 3
```

Update:

```text
best = 3
```

So:

```text
answer = 3
```

corresponding to:

```text
[1,2] + [1]
```

---

# 12. Another Dry Run

Consider:

```text
A = [2, 1, 1, 2, 1, 3, 4]
```

The optimal split is:

```text
[2,1] + [1,2,1,3,4]
```

Let's understand why the DP finds the answer.

Initially:

```text
best = 0
```

### `2`

There is no `dp[1]`.

So `2` cannot start a useful sequence.

Ignore it.

```text
best = 0
```

### First `1`

```text
dp[1] = best + 1
      = 1
```

So:

```text
best = 1
```

### Second `1`

Start another segment:

```text
dp[1] = max(1, 1 + 1)
      = 2
```

So:

```text
best = 2
```

This corresponds to splitting:

```text
[2,1] | [1]
```

where the first segment contributes `1`.

### `2`

Extend a segment with `f=1`:

```text
dp[2] = dp[1] + 1
      = 2 + 1
      = 3
```

### Next `1`

Start a new segment after the best previous partition:

```text
dp[1] = best + 1
      = 4
```

### `3`

Extend `dp[2]`:

```text
dp[3] = dp[2] + 1
```

### `4`

Extend `dp[3]`:

```text
dp[4] = dp[3] + 1
```

Eventually:

```text
best = 5
```

which corresponds to:

```text
[2,1] + [1,2,1,3,4]
```

with:

```text
f([2,1]) = 1
f([1,2,1,3,4]) = 4

total = 5
```

---

# 13. Why We Don't Need to Store the Whole Segment

A segment can contain many values:

```text
[1, 2, 1, 3, 5, 2, 4, ...]
```

But to extend its `f`, we only care about:

```text
current f
```

If its current:

```text
f = x-1
```

and we encounter:

```text
x
```

we can increase:

```text
f → x
```

Therefore `dp[x]` is enough.

We don't need to remember the actual elements of the segment.

---

# 14. Meaning of `dp[x]`

This is subtle and important.

`dp[x]` is **not**:

> Number of ways.

It is:

> Maximum total score achievable so far when the currently active segment has `f = x`.

So this is an **optimization DP**, not a counting DP.

---

# 15. Why `INT_MIN`?

Initially, most states are impossible.

For example:

```text
dp[3]
```

is impossible if we haven't managed to build a segment with:

```text
[1,2,3]
```

So we use:

```cpp
INT_MIN
```

to represent:

```text
unreachable state
```

Then:

```cpp
if (dp[x-1] != INT_MIN)
```

prevents invalid transitions.

---

# 16. Complete Algorithm

### Step 1

Find:

```text
mx = maximum value in A
```

We only need states from:

```text
0 ... mx
```

### Step 2

Initialize:

```text
dp[0] = 0
all other states = -INF
best = 0
```

### Step 3

Process each `x`:

### If `x == 1`

Start a new segment:

```cpp
dp[1] = max(dp[1], best + 1);
```

### If `x > 1`

Extend a segment with `f = x-1`:

```cpp
if (dp[x-1] != INT_MIN)
    dp[x] = max(dp[x], dp[x-1] + 1);
```

### Step 4

Update:

```cpp
best = max(best, dp[x]);
```

### Step 5

At the end:

```text
answer = best
```

---

# 17. C++ Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {

    int T;
    cin >> T;

    while (T--) {

        int n;
        cin >> n;

        vector<int> a(n);

        for (int &x : a)
            cin >> x;

        int mx = *max_element(a.begin(), a.end());

        // dp[x] =
        // maximum total score when the current
        // active segment has f = x.
        vector<int> dp(mx + 1, INT_MIN);

        // No active segment yet.
        dp[0] = 0;

        int best = 0;

        for (int x : a) {

            // x = 1 can always start a new segment.
            if (x == 1) {

                dp[1] = max(dp[1], best + 1);

            }
            else {

                // x can extend a segment whose
                // current f is x - 1.
                if (dp[x - 1] != INT_MIN) {

                    dp[x] = max(
                        dp[x],
                        dp[x - 1] + 1
                    );
                }
            }

            // Best answer over all possible
            // current segment states.
            best = max(best, dp[x]);
        }

        cout << best << '\n';
    }
}
```

---

# 18. Complexity

Let:

```text
M = max(A[i])
```

We maintain:

```text
dp[0 ... M]
```

For every element, we perform O(1) work.

Therefore:

```text
Time Complexity:  O(N)
Space Complexity: O(M)
```

Since:

```text
A[i] <= N
```

we have:

```text
M <= N
```

so:

```text
Time:  O(N)
Space: O(N)
```

Across all test cases:

```text
O(sum N)
```

with:

```text
sum N <= 2 * 10^5
```

---

# 19. Contest Explanation

A concise explanation:

> We process the array from left to right and maintain `dp[x]`, the maximum score obtained so far when the currently active segment has `f = x`. If the current value is `1`, it can start a new segment, giving `best + 1`, where `best` is the best score for the previous prefix. If the current value is `x > 1`, it can extend a segment whose current value of `f` is `x-1`, changing that segment's contribution from `x-1` to `x` and therefore increasing the total by `1`. We keep the best value across all states.

---

# 20. Pattern to Remember

## Partition + Local State DP

When a problem says:

```text
Split the array into any number of contiguous parts
and maximize the sum of some function of each part
```

don't immediately try all partitions.

Ask:

> What information about the current segment is sufficient to extend it?

Here:

```text
f(segment)
```

is all we need.

Therefore:

```text
dp[f]
```

is sufficient.

---

# 21. The Key Transition

The most important observation is:

```text
x = 1
    ↓
start a new segment
    ↓
dp[1] = best + 1
```

and:

```text
x > 1
    ↓
need current segment with f = x-1
    ↓
append x
    ↓
f increases by 1
    ↓
dp[x] = dp[x-1] + 1
```

---

# 22. One-Line Mental Model

> **A `1` starts a new chain, while `x > 1` extends a chain currently at `x-1`; maintain the best total score for every current chain length.**

That is the entire DP idea behind the problem.
