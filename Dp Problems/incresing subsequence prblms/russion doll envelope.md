

# **Russian Doll Envelopes**

**LeetCode Link:** [https://leetcode.com/problems/russian-doll-envelopes/](https://leetcode.com/problems/russian-doll-envelopes/)

---

## **Problem Statement**

You are given a 2D array of integers `envelopes` where `envelopes[i] = [w_i, h_i]` represents the width and the height of an envelope.

One envelope can fit into another if and only if both the width and height of one envelope are greater than the other envelope's width and height.

Return the maximum number of envelopes you can Russian doll (i.e., put one inside the other).

**Note:** You cannot rotate an envelope.

**Example:**
```
Input: envelopes = [[5,4],[6,4],[6,7],[2,3]]
Output: 3
Explanation: The maximum number of envelopes you can Russian doll is 3 ([2,3] => [5,4] => [6,7]).

Input: envelopes = [[1,1],[1,1],[1,1]]
Output: 1
```

---

## **Strategy: Sort + LIS**

**Key Insight:** This is a **2D LIS problem**. We need to handle both width and height.

### **Why We Can't Just Sort and Apply Standard LIS**

**Problem:**
```
envelopes = [[6,4], [6,7]]
Both have width 6!
Can we nest them? No! We need BOTH dimensions strictly greater.
```

If we sort by width ascending and apply LIS on heights, we'd incorrectly count envelopes with the same width.

### **The Clever Trick: Sort Width Ascending, Height Descending**

**Sorting Strategy:**
```python
envelopes.sort(key=lambda x: (x[0], -x[1]))
```

**Effect:**
- Same width envelopes are grouped together
- Within same width group, heights are in **descending** order
- This ensures we can only pick **one** envelope from each width group!

**Why This Works:**

When finding LIS on heights:
- Envelopes with same width have descending heights
- In LIS, we look for **strictly increasing** sequences
- Descending values cannot both be in the increasing sequence
- Therefore, at most one envelope per width is selected!

---

## **Solution**

```python
class Solution:
    def maxEnvelopes(self, envelopes: List[List[int]]) -> int:
        # Sort by width ascending, then by height descending
        envelopes.sort(key=lambda x: (x[0], -x[1]))
        
        # tails[i] = smallest height for a chain of length i+1
        tails = []
        
        for _, h in envelopes:
            # Binary search for the position to insert/replace h
            left = 0
            right = len(tails)
            
            while left < right:
                mid = left + (right - left) // 2
                
                if tails[mid] < h:
                    left = mid + 1
                else:
                    right = mid
            
            # If left == len(tails), h is larger than all existing heights
            if left == len(tails):
                tails.append(h)
            else:
                # Replace the height at position left with h
                tails[left] = h
        
        return len(tails)
```

---

## **Example Walkthrough**

**Input:** `envelopes = [[5,4], [6,4], [6,7], [2,3]]`

### **Step 1: Sort**
```python
envelopes.sort(key=lambda x: (x[0], -x[1]))
# Result: [[2,3], [5,4], [6,7], [6,4]]
#          width:2  width:5  width:6(desc heights)
```

**Notice:** Envelopes with width 6 have heights [7, 4] (descending).

### **Step 2: LIS on Heights**

**Extract heights:** `[3, 4, 7, 4]`

| h | Action | tails | Reason |
|---|--------|-------|--------|
| 3 | Append | [3] | First element |
| 4 | Append | [3, 4] | 4 > 3, extend |
| 7 | Append | [3, 4, 7] | 7 > 4, extend |
| 4 | Replace | [3, 4, 4] | Replace 7 with 4 at index 2 |

**Wait, this gives length 3, but let's verify:**

Actual sequence: `[2,3]` → `[5,4]` → `[6,7]`
- Can [2,3] fit in [5,4]? 2 < 5 ✓ and 3 < 4 ✓ → Yes!
- Can [5,4] fit in [6,7]? 5 < 6 ✓ and 4 < 7 ✓ → Yes!

**Note:** The envelope `[6,4]` is **not** included because:
- In sorted order: `... [6,7], [6,4]`
- Heights: `... 7, 4` (descending)
- LIS algorithm cannot use both 7 and 4 (not increasing)
- This correctly prevents using two envelopes with the same width!

**Result:** `3`

---

## **Why Sort Height Descending for Same Width?**

**Scenario:** `envelopes = [[6,4], [6,7]]`

**Wrong Approach (both ascending):**
```
Sort: [[6,4], [6,7]]
Heights: [4, 7]
LIS on [4, 7] = 2 ❌ Wrong! Can't nest envelopes with same width
```

**Correct Approach (width asc, height desc):**
```
Sort: [[6,7], [6,4]]
Heights: [7, 4]
LIS on [7, 4] = 1 ✓ Correct! Can only pick one
```

The descending heights **prevent** the LIS algorithm from selecting multiple envelopes with the same width.

---

The descending heights **prevent** the LIS algorithm from selecting multiple envelopes with the same width.

---

## **Binary Search Logic Explained**

### **The Two Pointers Strategy**

We use `left` and `right` pointers:

- **If `tails[mid] < h`:** The height at mid is too small to be our replacement target. We move `left = mid + 1` (search right side).
- **If `tails[mid] >= h`:** Mid could be the replacement spot, but there might be an even better (smaller) index to the left. We move `right = mid` (search left side).

### **Why This Specific Template?**

This is the standard template for finding the **First Element Greater Than or Equal To** a target.

| Aspect | Explanation |
|--------|-------------|
| **Initial `right`** | Set to `len(tails)` because if h is larger than everything in the list, the insertion point is at the very end |
| **Loop Condition** | `left < right`: When `left == right`, the search space has collapsed to a single point (our answer) |
| **Mid Calculation** | `mid = left + (right - left) // 2` prevents potential integer overflow (though Python handles large integers automatically) |

---

## **Key Insight**

By thinking of it this way:

- We are trying to find a **"home"** for the current height
- If we find a home already occupied by someone **larger or equal** (`tails[mid] >= h`), we **replace them** to make the sequence **"tighter"**
- If no one can accommodate us, we **start a new "home"** at the end of the list

This greedy approach ensures we always maintain the longest possible chain!

---

## **Comparison: Naive O(n²) vs Optimized O(n log n)**

### **Naive Approach**
```python
envelopes.sort()
dp = [1] * n
for i in range(n):
    for j in range(i):
        if envelopes[j][0] < envelopes[i][0] and envelopes[j][1] < envelopes[i][1]:
            dp[i] = max(dp[i], dp[j] + 1)
return max(dp)
```
**Time:** $O(n^2)$ - Too slow for large inputs!

### **Optimized Approach (This Solution)**
- Sort cleverly: $O(n \log n)$
- LIS with binary search: $O(n \log n)$
- **Total:** $O(n \log n)$ ✓

---

## **Edge Cases**

### **1. All Same Width**
```
Input: [[5,4], [5,7], [5,2]]
After sorting: [[5,7], [5,4], [5,2]]
Heights: [7, 4, 2] (strictly decreasing)
LIS = 1 (can only pick one)
```

### **2. All Same Height**
```
Input: [[2,5], [4,5], [6,5]]
After sorting: [[2,5], [4,5], [6,5]]
Heights: [5, 5, 5]
LIS = 1 (can only pick one because heights must be strictly increasing)
```

### **3. Already Nested**
```
Input: [[1,2], [2,3], [3,4]]
After sorting: [[1,2], [2,3], [3,4]]
Heights: [2, 3, 4]
LIS = 3 (all can be nested)
```

---

## **Why Use Binary Search Instead of `bisect_left`?**

**You could use `bisect_left`:**
```python
import bisect
for _, h in envelopes:
    idx = bisect.bisect_left(tails, h)
    if idx == len(tails):
        tails.append(h)
    else:
        tails[idx] = h
```

**Both are equivalent!** The manual binary search implementation shown in the solution is educational and shows the underlying logic.

---

## **Complexity Analysis**

**Time Complexity:** $O(n \log n)$  
- Sorting: $O(n \log n)$
- For each envelope, binary search on `tails`: $O(\log n)$
- Total: $O(n \log n) + O(n \log n) = O(n \log n)$

**Space Complexity:** $O(n)$  
The `tails` array can grow up to size $n$ in the worst case (when all envelopes can be nested).

