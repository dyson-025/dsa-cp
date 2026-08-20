# Mex and Max

## 1. Problem

Given an array `A` of size `N`, count the number of **non-empty subsequences** `B` such that:

```text
|mex(B) - max(B)| <= 1
```

where:

- `mex(B)` = smallest non-negative integer not present in `B`
- `max(B)` = maximum element in `B`

Two subsequences are different if they use different indices, even when their values are the same.

Answer modulo:

```text
998244353
```

### Constraints

- `1 <= T <= 100`
- `2 <= N <= 100`
- `0 <= A[i] <= N`

---

# 2. Understanding MEX

The MEX is the smallest non-negative integer that does not occur.

Examples:

```text
B = [0, 1, 2]
mex = 3

B = [0, 1, 4]
mex = 2

B = [1, 2]
mex = 0

B = [0, 2]
mex = 1
```

The maximum is simply the largest selected value.

For:

```text
B = [0, 2]
```

we have:

```text
mex = 1
max = 2

|1 - 2| = 1
```

so this subsequence is good.

---

# 3. What Are We Trying to Count?

For every element, we have two choices:

```text
Take it
Don't take it
```

Therefore, without any optimization, there are:

```text
2^N
```

possible subsequences.

Since `N <= 100`, we need to avoid explicitly generating all subsequences.

This naturally suggests **DP over the information that matters about the current subsequence**.

---

# 4. What Information Do We Need?

While processing the array from left to right, suppose we have already selected some elements.

At the end, we need:

```text
mex(B)
max(B)
```

So the important state is:

```text
(index, mex, maximum)
```

Define:

```text
dp[idx][mex][maxi]
```

as the number of ways to process the suffix starting at `idx`, when the currently selected elements have:

```text
mex = mex
max = maxi
```

---

# 5. Why Do We Sort the Array?

The code does:

```cpp
sort(v.begin(), v.end());
```

This is important for the MEX transition.

Suppose the current MEX is:

```text
mex = 2
```

If we select a value:

```text
v[idx] = 2
```

then the MEX becomes:

```text
3
```

because `2` was the missing value.

But if:

```text
v[idx] < mex
```

selecting it cannot change the MEX.

If:

```text
v[idx] > mex
```

it also cannot immediately change the MEX.

Because the array is sorted, once we move forward, we never encounter a smaller value later.

Therefore we can update MEX with the simple rule:

```cpp
if (v[idx] == mex)
    newMex++;
```

---

# 6. DP State

Our recursive function is:

```cpp
solve(idx, mex, maxi)
```

Meaning:

> We are currently at index `idx`, and the subsequence selected so far has MEX `mex` and maximum `maxi`.

The DP table is:

```cpp
dp[idx][mex][maxi]
```

---

# 7. Empty Subsequence Handling

For the empty subsequence:

```text
mex(empty) = 0
```

but:

```text
max(empty)
```

does not exist.

Therefore we use:

```cpp
maxi = -1
```

as a sentinel.

We start with:

```cpp
solve(0, 0, -1, v);
```

Meaning:

```text
idx  = 0
mex  = 0
maxi = -1 → nothing selected yet
```

This is important because the problem asks for **non-empty subsequences**.

---

# 8. Base Case

When:

```cpp
idx == n
```

we have processed the entire array.

Now we check:

```cpp
return (maxi != -1 && abs(mex - maxi) <= 1);
```

There are two conditions.

### Condition 1: Non-empty

```cpp
maxi != -1
```

If `maxi == -1`, nothing was selected.

So we return `0`.

### Condition 2: Good subsequence

```cpp
abs(mex - maxi) <= 1
```

If true:

```text
return 1
```

otherwise:

```text
return 0
```

---

# 9. Two Choices at Every Element

At every index we have exactly two choices.

## Choice 1 — Don't Take

The current MEX and maximum remain unchanged:

```cpp
int nottake = solve(
    idx + 1,
    mex,
    maxi,
    v
);
```

---

## Choice 2 — Take

We create new states:

```cpp
int newMex = mex;
int newMaxi = maxi;
```

### Update MEX

If the selected value is exactly the current MEX:

```cpp
if (v[idx] == newMex)
    newMex++;
```

Because we just added the missing value.

### Update Maximum

```cpp
newMaxi = max(newMaxi, v[idx]);
```

Then:

```cpp
int take = solve(
    idx + 1,
    newMex,
    newMaxi,
    v
);
```

---

# 10. Combine Both Choices

Every valid subsequence from the current state either:

```text
takes v[idx]
```

or:

```text
doesn't take v[idx]
```

Therefore:

```cpp
ans = take + nottake
```

Modulo `998244353`:

```cpp
int ans = (take + nottake) % MOD;
```

Then store it:

```cpp
dp[idx][mex][maxi] = ans;
```

---

# 11. Important DP Memoization Detail

The current state is:

```text
(idx, mex, maxi)
```

So we must store the answer using the **original** state:

```cpp
dp[idx][mex][maxi]
```

Do not mutate `mex` and `maxi` and then use the modified values as the current DP key.

That's why the code uses:

```cpp
int newMex = mex;
int newMaxi = maxi;
```

and modifies those instead.

Correct:

```text
current state:
(idx, mex, maxi)

take transition:
(idx+1, newMex, newMaxi)

store answer:
dp[idx][mex][maxi]
```

---

# 12. Complete Transition

The entire DP transition is:

```text
                     solve(idx, mex, maxi)
                       /              \
                      /                \
                DON'T TAKE            TAKE
                    |                    |
                    |              newMex = mex
                    |              newMaxi = maxi
                    |                    |
                    |          if v[idx] == mex:
                    |              newMex++
                    |                    |
                    |          newMaxi=max(maxi,v[idx])
                    |                    |
                    ↓                    ↓
              solve(idx+1,       solve(idx+1,
                  mex,               newMex,
                  maxi)              newMaxi)
                    \                  /
                     \                /
                      ------ + -------
                            |
                           ans
```

---

# 13. Dry Run — `[0, 1, 2]`

Consider:

```text
A = [0, 1, 2]
```

After sorting:

```text
[0, 1, 2]
```

Start:

```text
solve(0, 0, -1)
```

Consider `0`.

### Don't take 0

State:

```text
mex = 0
max = -1
```

Eventually this represents an empty subsequence, which is rejected.

### Take 0

Since:

```text
v[idx] == mex
0 == 0
```

MEX becomes:

```text
mex = 1
```

and:

```text
max = 0
```

Now we process `1`.

Taking it:

```text
mex = 2
max = 1
```

Taking `2`:

```text
mex = 3
max = 2
```

At the end:

```text
|3 - 2| = 1
```

so:

```text
[0,1,2]
```

is good.

Other good subsequences are:

```text
[0]
[0,1]
[0,2]
[1]
```

giving total:

```text
5
```

which matches the sample.

---

# 14. Why `[1]` Is Good

This is an important MEX example.

For:

```text
B = [1]
```

we have:

```text
mex = 0
max = 1
```

Therefore:

```text
|0 - 1| = 1
```

so it is valid.

This is why we cannot assume that a good subsequence must contain `0`.

---

# 15. Why `[0,2]` Is Good

For:

```text
B = [0,2]
```

we have:

```text
mex = 1
max = 2
```

Therefore:

```text
|1 - 2| = 1
```

so it is valid.

---

# 16. Why `[3,3,3]` Gives 0

For any non-empty subsequence consisting only of `3`s:

```text
mex = 0
max = 3
```

Therefore:

```text
|0 - 3| = 3
```

which is greater than `1`.

So:

```text
answer = 0
```

---

# 17. C++ Code

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MOD = 998244353;

int dp[105][105][105];

int solve(int idx, int mex, int maxi, vector<int>& v) {

    // All elements processed.
    if (idx == v.size()) {

        // Empty subsequence is not allowed.
        if (maxi == -1)
            return 0;

        // Check |mex - max| <= 1.
        return abs(mex - maxi) <= 1;
    }

    // Current DP state:
    // (idx, mex, maxi)
    //
    // maxi = -1 represents the empty subsequence.
    if (maxi != -1 && dp[idx][mex][maxi] != -1)
        return dp[idx][mex][maxi];

    // --------------------------------
    // 1. Don't take v[idx]
    // --------------------------------
    int nottake = solve(
        idx + 1,
        mex,
        maxi,
        v
    );

    // --------------------------------
    // 2. Take v[idx]
    // --------------------------------
    int newMex = mex;
    int newMaxi = maxi;

    // If we add the current missing number,
    // MEX increases by 1.
    if (v[idx] == newMex)
        newMex++;

    // Update maximum.
    newMaxi = max(newMaxi, v[idx]);

    int take = solve(
        idx + 1,
        newMex,
        newMaxi,
        v
    );

    // Combine both choices.
    int ans = (take + nottake) % MOD;

    // Store using the ORIGINAL state.
    if (maxi != -1)
        dp[idx][mex][maxi] = ans;

    return ans;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int T;
    cin >> T;

    while (T--) {

        int n;
        cin >> n;

        vector<int> v(n);

        for (int &x : v)
            cin >> x;

        // Sorting makes the MEX transition simple.
        sort(v.begin(), v.end());

        memset(dp, -1, sizeof(dp));

        // Initially:
        // idx = 0
        // mex = 0
        // maxi = -1 (empty)
        cout << solve(0, 0, -1, v) << '\n';
    }

    return 0;
}
```

---

# 18. Complexity

The DP state contains:

```text
idx  -> O(N)
mex  -> O(N)
maxi -> O(N)
```

Therefore there are at most:

```text
O(N^3)
```

states.

Each state has two transitions:

```text
take
not take
```

so:

```text
Time Complexity: O(N^3)
Space Complexity: O(N^3)
```

With:

```text
N <= 100
```

this is feasible.

The recursion stack adds:

```text
O(N)
```

but the DP table dominates the space.

---

# 19. Why Sorting Helps

Without sorting, suppose:

```text
mex = 0
```

and we select:

```text
5
```

The MEX remains `0`.

Later we might encounter:

```text
0
```

and MEX changes to `1`.

With sorting, values arrive in increasing order.

Therefore, when:

```cpp
v[idx] == mex
```

we know that selecting this value advances the MEX.

Values smaller than the current MEX have already been processed, and values larger than the current MEX cannot fill the current missing value.

---

# 20. Important Caveat About the MEX Transition

The code uses:

```cpp
if (v[idx] == newMex)
    newMex++;
```

This works because the array is sorted.

However, conceptually remember:

> Adding a number increases MEX only when that number is exactly the current MEX.

For example:

```text
current mex = 2

take 0 → mex stays 2
take 1 → mex stays 2
take 2 → mex becomes 3
take 5 → mex stays 2
```

---

# 21. Interview / Contest Explanation

A concise explanation:

> We process the sorted array using DP. For every state we only need the current index, MEX of the selected subsequence, and its maximum. At each index we either skip the element or take it. If we take the current element and it equals the current MEX, the MEX increases by one; the maximum is updated normally. At the end, a state contributes one if the subsequence is non-empty and `|mex - max| <= 1`. Memoization avoids recomputing identical states.

---

# 22. Pattern to Remember

## Subsequence + Small State → DP

When:

```text
Each element → Take / Don't Take
```

and the final condition depends only on a few properties of the chosen elements, ask:

> Can I represent the state using those properties?

Here:

```text
index
  +
mex
  +
maximum
```

is enough.

So:

```text
dp[index][mex][maximum]
```

captures everything we need.

---

# 23. DP Thinking

The most important thought process is:

```text
What information about the selected subsequence
will matter in the future?
```

For this problem:

```text
"MEX" matters
"MAX" matters
"Which index am I processing?" matters
```

Nothing else about the actual selected elements is needed.

Therefore:

```text
STATE = (idx, mex, max)
```

This is the core DP insight.

---

# 24. Key Takeaways 🧠

### 1. Empty subsequence

Use:

```cpp
maxi = -1
```

to represent:

```text
nothing selected yet
```

and reject it at the base case.

### 2. Take / Don't Take

Every element creates two transitions:

```text
not take → state unchanged
take     → update mex and max
```

### 3. MEX

If:

```cpp
v[idx] == mex
```

then:

```cpp
mex++
```

after sorting.

### 4. Maximum

Always:

```cpp
newMaxi = max(maxi, v[idx])
```

### 5. Memoization

Store using the **original state**:

```cpp
dp[idx][mex][maxi]
```

not the modified `newMex/newMaxi`.

### One-line memory trick

> **For every subsequence state, remember only `(index, MEX, MAX)`; take/skip each element, update MEX if it fills the current missing value, update MAX normally, and accept the state if `|MEX-MAX| <= 1`.**
