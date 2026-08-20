# Modify and Subtract

## 1. Problem

Given an array `A` of size `N`, we can repeatedly perform:

```text
Choose i (1 <= i < N):
A[i]   -= 1
A[i+1] -= 1
```

An array is **good** if it can be reduced to all zeroes.

For every index `i`, we are allowed to replace `A[i]` with **any non-negative integer `X`**.

We need to count how many indices `i` have at least one valid `X` such that the resulting array is good.

### Constraints

- `1 <= T <= 10^4`
- `2 <= N <= 2 * 10^5`
- `0 <= A[i] <= 10^9`
- Sum of `N` over all test cases <= `2 * 10^5`

---

# 2. Understanding the Operation

Define:

```text
x1 = number of operations on (A[0], A[1])
x2 = number of operations on (A[1], A[2])
x3 = number of operations on (A[2], A[3])
...
```

For the array to become all zeroes:

```text
A[0] = x1

A[1] = x1 + x2

A[2] = x2 + x3

A[3] = x3 + x4

...

A[N-1] = x[N-2]
```

Therefore, we can determine the required operation counts from left to right.

---

# 3. Recovering Operations from the Left

For the first element:

```text
x1 = A[0]
```

For the second:

```text
A[1] = x1 + x2

x2 = A[1] - x1
```

For the third:

```text
A[2] = x2 + x3

x3 = A[2] - x2
```

So generally:

```text
x[i] = A[i] - x[i-1]
```

### Important condition

Every `x[i]` must be non-negative.

If:

```text
x[i] < 0
```

then the prefix is impossible.

Why?

Because `x[i]` represents the number of times we perform an operation. We cannot perform an operation a negative number of times.

Also, changing an element **after this point cannot repair the already-invalid prefix**.

---

# 4. Example

Consider:

```text
A = [3, 4, 0, 4, 3]
```

From the left:

```text
x1 = 3

x2 = 4 - 3 = 1

x3 = 0 - 1 = -1   ❌
```

Therefore the original array is not good.

But we are allowed to change one element.

This leads to the main observation.

---

# 5. Key Observation — Remove One Position

Suppose we change:

```text
A[i] -> X
```

Temporarily ignore `A[i]`.

The array splits into two independent parts:

```text
LEFT                  RIGHT

A[0 ... i-1]          A[i+1 ... N-1]
```

The left side tells us how much value position `i` needs from the left.

Call this:

```text
L
```

The right side tells us how much value position `i` needs from the right.

Call this:

```text
R
```

Therefore we can choose:

```text
X = L + R
```

Since both `L` and `R` are non-negative:

```text
X >= 0
```

So for index `i` to be valid:

```text
LEFT must be valid
AND
RIGHT must be valid
```

That is the entire trick.

---

# 6. Why Does X = L + R Work?

Imagine:

```text
             X
           /   \
          /     \
       LEFT     RIGHT
         |         |
         L         R
```

The left side needs `L` units from `X`.

The right side needs `R` units from `X`.

Therefore:

```text
X = L + R
```

For example, if:

```text
L = 1
R = 1
```

then:

```text
X = 2
```

The modified array becomes:

```text
[3, 4, 2, 4, 3]
```

and it is good.

---

# 7. Prefix Array

We precompute the required operation value for every prefix.

Let:

```text
pref[i] = required operation count after processing A[0...i]
```

For:

```text
A = [3, 4, 0, 4, 3]
```

we get:

```text
i = 0:
x1 = 3

i = 1:
x2 = 4 - 3 = 1

i = 2:
x3 = 0 - 1 = -1  -> invalid
```

Therefore:

```text
prefValid = [true, true, false, false, false]
```

The first two prefix portions are valid, but anything containing index `2` is invalid.

---

# 8. Suffix Array

We do exactly the same thing from right to left.

For:

```text
A = [3, 4, 0, 4, 3]
```

from the right:

```text
x4 = 3

x3 = 4 - 3 = 1

x2 = 0 - 1 = -1  ❌
```

Therefore:

```text
suffValid = [false, false, false, true, true]
```

The suffix `[4,3]` is valid, but suffixes containing the `0` before it are not.

---

# 9. Checking Every Index

For a candidate index `i`:

### Left

If `i == 0`, there is no left side:

```text
L = 0
```

Otherwise:

```text
L = pref[i-1]
```

and we require:

```text
prefValid[i-1] == true
```

### Right

If `i == N-1`, there is no right side:

```text
R = 0
```

Otherwise:

```text
R = suff[i+1]
```

and we require:

```text
suffValid[i+1] == true
```

If both are valid:

```text
X = L + R
```

and index `i` is counted.

---

# 10. Complete Example

Consider:

```text
A = [3, 4, 0, 4, 3]
```

### Prefix

```text
pref:
index       0   1   2   3   4
            3   1   -   -   -

valid:      T   T   F   F   F
```

### Suffix

```text
suff:
index       0   1   2   3   4
            -   -   -   1   3

valid:      F   F   F   T   T
```

Now check each index:

| i | Left | Right | Result |
|---|---|---|---|
| 0 | Empty | Invalid | ❌ |
| 1 | Valid | Invalid | ❌ |
| 2 | Valid, `L=1` | Valid, `R=1` | ✅ `X=2` |
| 3 | Invalid | Valid | ❌ |
| 4 | Invalid | Empty | ❌ |

Therefore:

```text
Answer = 1
```

---

# 11. Optimized C++ Implementation

We do **not** need to store the actual prefix/suffix requirements.

For each index, we only need to know:

```text
Is the prefix valid?
Is the suffix valid?
```

Why?

If both are valid, their requirements `L` and `R` are already non-negative, so:

```text
X = L + R >= 0
```

Therefore, the actual value of `X` does not need to be calculated.

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int t;
    cin >> t;

    while (t--) {
        int n;
        cin >> n;

        vector<int> v(n);

        for (auto& it : v)
            cin >> it;

        // --------------------------------
        // Prefix validity
        // --------------------------------
        vector<bool> pf(n, false);

        pf[0] = true;
        int pfsum = v[0];

        for (int i = 1; i < n; i++) {

            int req = v[i] - pfsum;

            if (req < 0)
                break;

            pf[i] = true;

            // Carry the required operation count forward.
            pfsum = req;
        }

        // --------------------------------
        // Suffix validity
        // --------------------------------
        vector<bool> sf(n, false);

        sf[n - 1] = true;
        int sfsum = v[n - 1];

        for (int i = n - 2; i >= 0; i--) {

            int req = v[i] - sfsum;

            if (req < 0)
                break;

            sf[i] = true;

            // Carry the required operation count backward.
            sfsum = req;
        }

        // --------------------------------
        // Check every possible modified index
        // --------------------------------
        int ans = 0;

        for (int i = 0; i < n; i++) {

            bool leftok =
                (i == 0 || pf[i - 1]);

            bool rightok =
                (i == n - 1 || sf[i + 1]);

            if (leftok && rightok)
                ans++;
        }

        cout << ans << '\n';
    }
}
```

### Important implementation detail

After calculating:

```cpp
int req = v[i] - pfsum;
```

we **must** do:

```cpp
pfsum = req;
```

Similarly:

```cpp
sfsum = req;
```

Otherwise, every next calculation would use the old requirement instead of the newly calculated one.

For example:

```text
A = [3, 4, 2, 4, 3]

x1 = 3
x2 = 4 - 3 = 1
x3 = 2 - 1 = 1
x4 = 4 - 1 = 3
```

So the running value must change:

```text
3 → 1 → 1 → 3
```

---

# 12. Complexity

For every test case:

```text
Build prefix validity  → O(N)
Build suffix validity  → O(N)
Check every index      → O(N)
```

Therefore:

```text
Time Complexity:  O(N)
Space Complexity: O(N)
```

Across all test cases:

```text
Sum of N <= 2 * 10^5
```

so the total complexity is:

```text
O(sum N)
```

The optimized version uses only two boolean arrays:

```text
pf[] → prefix validity
sf[] → suffix validity
```

We don't store `pref[]` and `suff[]` because their actual values are unnecessary for the final counting step.

---

# 13. How to Explain in a Contest

A concise explanation:

> Let `x[i]` be the number of times we perform the operation on the pair `(i, i+1)`. For a fixed array, these values are uniquely determined from left to right by `x[i] = A[i] - x[i-1]`, and every `x[i]` must be non-negative. We precompute these requirements for every prefix and similarly from the right for every suffix. If we change `A[i]`, the elements before and after `i` become independent. The left side requires some value `L` from `A[i]`, while the right side requires `R`. Therefore we can choose `X = L + R`. Hence an index is valid exactly when both its prefix and suffix are valid.

---

# 14. Core Pattern

## Remove One Element → Independent Prefix + Suffix

Whenever a problem says:

```text
"You may change A[i] to ANY value."
```

ask:

> Can I temporarily remove `A[i]` and make the left and right sides independent?

If yes, try:

```text
prefix information
       +
suffix information
       ↓
answer for index i
```

For this problem:

```text
Left requirement  = pref[i-1]
Right requirement = suff[i+1]

X = Left + Right
```

---

# 15. Key Takeaway 🧠

The most important thing to remember is **not the code**.

Remember the operation chain:

```text
x1 = A[0]

x2 = A[1] - x1

x3 = A[2] - x2

...
```

If any required `x` becomes negative:

```text
prefix is impossible
```

Do the same from the right.

Then for every position:

```text
left valid + right valid
          ↓
       X = L + R
          ↓
      X is automatically >= 0
          ↓
      index is valid
```

We don't need to calculate `X` in the implementation. We only need to check whether both sides are valid.

### One-line memory trick

> **Compute required operations from both directions; if removing an index leaves a valid prefix and valid suffix, combine their requirements and that index works.**
