

# **Russian Doll Envelopes**

**LeetCode Link:** [https://leetcode.com/problems/russian-doll-envelopes/](https://leetcode.com/problems/russian-doll-envelopes/)

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

## **Key Insight**

By thinking of it this way:

- We are trying to find a **"home"** for the current height
- If we find a home already occupied by someone **larger or equal** (`tails[mid] >= h`), we **replace them** to make the sequence **"tighter"**
- If no one can accommodate us, we **start a new "home"** at the end of the list

This greedy approach ensures we always maintain the longest possible chain!

