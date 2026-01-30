# Minimum Window Substring

**LeetCode 76** | **Difficulty:** Hard | **Category:** Sliding Window, Hash Map, Two Pointers

---

## Problem Statement

Given two strings `s` and `t` of lengths `m` and `n` respectively, return the **minimum window substring** of `s` such that every character in `t` (including duplicates) is included in the window. If there is no such substring, return the empty string `""`.

The testcases will be generated such that the answer is **unique**.

---

## Examples

### Example 1:
```
Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"

Explanation: 
The minimum window substring "BANC" includes 'A', 'B', and 'C' from string t.
```

### Example 2:
```
Input: s = "a", t = "a"
Output: "a"

Explanation: 
The entire string s is the minimum window.
```

### Example 3:
```
Input: s = "a", t = "aa"
Output: ""

Explanation: 
Both 'a's from t must be included in the window.
Since the largest window of s only has one 'a', return empty string.
```

---

## Strategy

### Two-Pass Hash Map with "Have vs. Need" Optimization

To solve this efficiently, we need a way to track if our current window is "sufficient."

**Key Insights:**

1. **Requirement Map:** Create a frequency map for string `t` (what we need)
2. **Window Map:** Track characters in our current window of `s` (what we have)
3. **"Have vs. Need" Variable:** Instead of comparing maps repeatedly (which is $O(26)$), use a single integer `have` to track how many unique characters have met their required frequency

**The Algorithm:**

1. **Expand:** Move `right` pointer to include more characters
   - Update window counts
   - Increment `have` when a character meets its requirement

2. **Contract:** Once `have == need` (valid window), shrink from left
   - Try to minimize window size
   - Update result if smaller window found
   - Decrement `have` when a character falls below requirement

3. **Track Result:** Keep the smallest valid window

**Why This Works:**
- **Two Pointers:** Efficient O(n) traversal without nested loops
- **Have/Need Optimization:** O(1) validity check instead of O(k) map comparison
- **Greedy Shrinking:** Once valid, immediately minimize

**Complexity:**
- **Time:** $O(m + n)$ where m = len(s), n = len(t)
  - Build target map: O(n)
  - Each character in s visited at most twice: O(m)
- **Space:** $O(m + n)$ for hash maps

---

## Solution

```python
from collections import Counter

class Solution:
    def minWindow(self, s: str, t: str) -> str:
        if not t or not s:
            return ''
        
        # Frequency map of target string
        target_counts = Counter(t)
        
        # Window frequency map
        window_counts = {}
        
        # 'need' is the number of unique characters in t
        # 'have' is how many of those unique characters meet the requirement in the window
        have, need = 0, len(target_counts)
        
        # Result: [left, right] indices and length
        res, res_len = [-1, -1], float('inf')
        left = 0
        
        for right in range(len(s)):
            # Expand window: add character at right
            char = s[right]
            window_counts[char] = window_counts.get(char, 0) + 1
            
            # If this character is needed and its count matches the target
            if char in target_counts and window_counts[char] == target_counts[char]:
                have += 1
            
            # While the current window is valid, try to shrink it
            while have == need:
                # Update result if this window is smaller than previous best
                if (right - left + 1) < res_len:
                    res = [left, right]
                    res_len = right - left + 1
                
                # Contract window: remove character from left
                left_char = s[left]
                window_counts[left_char] -= 1
                
                # If removing this char makes the window invalid
                if left_char in target_counts and window_counts[left_char] < target_counts[left_char]:
                    have -= 1
                
                left += 1
        
        l, r = res
        return s[l:r+1] if res_len != float('inf') else ""
```

---

## Detailed Walkthrough

### Example: `s = "ADOBECODEBANC"`, `t = "ABC"`

**Setup:**
- `target_counts = {'A': 1, 'B': 1, 'C': 1}`
- `need = 3` (three unique characters)

**Step-by-Step Execution:**

| Step | Right | Char | Window Counts | Have | Left | Window | Valid? | Result |
|------|-------|------|---------------|------|------|--------|--------|--------|
| 0 | 0 | 'A' | {'A':1} | 1 | 0 | "A" | ❌ | - |
| 1 | 1 | 'D' | {'A':1, 'D':1} | 1 | 0 | "AD" | ❌ | - |
| 2 | 2 | 'O' | {'A':1, 'D':1, 'O':1} | 1 | 0 | "ADO" | ❌ | - |
| 3 | 3 | 'B' | {'A':1, 'D':1, 'O':1, 'B':1} | 2 | 0 | "ADOB" | ❌ | - |
| 4 | 4 | 'E' | {'A':1, 'D':1, 'O':1, 'B':1, 'E':1} | 2 | 0 | "ADOBE" | ❌ | - |
| 5 | 5 | 'C' | {'A':1, 'D':1, 'O':1, 'B':1, 'E':1, 'C':1} | 3 | 0 | "ADOBEC" | ✅ | [0,5] len=6 |
| 5a | 5 | - | {'D':1, 'O':1, 'B':1, 'E':1, 'C':1} (removed 'A') | 2 | 1 | "DOBEC" | ❌ | [0,5] |
| 6 | 6 | 'O' | {'D':1, 'O':2, 'B':1, 'E':1, 'C':1} | 2 | 1 | "DOBECO" | ❌ | [0,5] |
| 7 | 7 | 'D' | {'D':2, 'O':2, 'B':1, 'E':1, 'C':1} | 2 | 1 | "DOBECOD" | ❌ | [0,5] |
| 8 | 8 | 'E' | {'D':2, 'O':2, 'B':1, 'E':2, 'C':1} | 2 | 1 | "DOBECODE" | ❌ | [0,5] |
| 9 | 9 | 'B' | {'D':2, 'O':2, 'B':2, 'E':2, 'C':1} | 2 | 1 | "DOBECODEB" | ❌ | [0,5] |
| 10 | 10 | 'A' | {'D':2, 'O':2, 'B':2, 'E':2, 'C':1, 'A':1} | 3 | 1 | "DOBECODEBA" | ✅ | [0,5] |
| 10a | 10 | - | ... (shrinking) | - | 5 | "CODEBA" | ✅ | [0,5] |
| 10b | 10 | - | {'O':1, 'B':1, 'E':1, 'A':1} (removed 'D','O','D','E','C') | 2 | 6 | "ODEBA" | ❌ | [0,5] |
| 11 | 11 | 'N' | {'O':1, 'B':1, 'E':1, 'A':1, 'N':1} | 2 | 6 | "ODEBAN" | ❌ | [0,5] |
| 12 | 12 | 'C' | {'O':1, 'B':1, 'E':1, 'A':1, 'N':1, 'C':1} | 3 | 6 | "ODEBANC" | ✅ | [0,5] |
| 12a | 12 | - | ... (shrinking) | - | 9 | "BANC" | ✅ | **[9,12] len=4** |
| 12b | 12 | - | {'A':1, 'N':1, 'C':1} (removed 'O','D','E','B') | 2 | 10 | "ANC" | ❌ | [9,12] |

**Final Answer:** `s[9:13] = "BANC"` with length 4

**Key Observations:**
1. At step 5, we find the first valid window "ADOBEC" (length 6)
2. We immediately shrink until invalid
3. At step 12, we find "ODEBANC" (length 7)
4. We shrink to "BANC" (length 4) - the optimal answer!

---

## Critical Insight: The "Have/Need" Optimization

### Why Not Compare Hash Maps Directly?

**Naive Approach (SLOW):**
```python
# Wrong! This is O(k) where k = unique chars in t
while all(window_counts.get(char, 0) >= target_counts[char] for char in target_counts):
    # Shrink window...
```

**Problem:** 
- If `t` contains many unique characters (up to 26 for lowercase, 52 for mixed case)
- Each comparison takes O(k) time
- With n iterations, total becomes O(n × k)

**Optimized Approach (FAST):**
```python
# Correct! This is O(1)
while have == need:
    # Shrink window...
```

**How It Works:**

1. **`need`** = `len(Counter(t))` (number of unique characters in t)
   - Example: t = "AABC" → need = 3 (A, B, C)

2. **`have`** tracks how many unique characters in the window meet their requirement
   - Increment when: `window_counts[char] == target_counts[char]`
   - Decrement when: `window_counts[char] < target_counts[char]`

**Example:**
```
t = "AAB"
target_counts = {'A': 2, 'B': 1}
need = 2

Window: "AAAB"
window_counts = {'A': 3, 'B': 1}
- 'A': 3 >= 2 ✓ (have++)
- 'B': 1 >= 1 ✓ (have++)
have = 2 == need → Valid!

Remove one 'A':
window_counts = {'A': 2, 'B': 1}
- 'A': 2 >= 2 ✓ (still valid)
- 'B': 1 >= 1 ✓
have = 2 == need → Still valid!

Remove another 'A':
window_counts = {'A': 1, 'B': 1}
- 'A': 1 < 2 ✗ (have--)
have = 1 != need → Invalid!
```

**Key Rules:**
- `have` only increments when count **exactly matches** target (prevents counting twice)
- `have` only decrements when count **falls below** target
- Comparison is O(1) instead of O(k)

---

## Edge Cases

### 1. Target Longer Than Source
```python
s = "a", t = "aa"
# Output: ""
# Impossible to satisfy
```

### 2. Single Character Match
```python
s = "a", t = "a"
# Output: "a"
```

### 3. No Common Characters
```python
s = "abc", t = "xyz"
# Output: ""
```

### 4. Target Has Duplicates
```python
s = "aabbcc", t = "abc"
# Output: "abc" or "bca" or "cab" (any valid substring)
```

### 5. Entire String is Answer
```python
s = "abc", t = "abc"
# Output: "abc"
```

### 6. Target with High Frequency
```python
s = "aaaaaaaaab", t = "aaab"
# Output: "aaaab" (need 3 'a's and 1 'b')
```

---

## Common Mistakes

### ❌ Mistake 1: Comparing Hash Maps in While Loop
```python
# Wrong! O(k) comparison
while all(window_counts.get(c, 0) >= target_counts[c] for c in target_counts):
    # This is slow!
```

**Fix:** Use `have == need` for O(1) comparison.

### ❌ Mistake 2: Incrementing Have Incorrectly
```python
# Wrong! Increments multiple times for same character
if char in target_counts and window_counts[char] <= target_counts[char]:
    have += 1
```

**Fix:** Only increment when **exactly equal**:
```python
if char in target_counts and window_counts[char] == target_counts[char]:
    have += 1
```

### ❌ Mistake 3: Not Checking Bounds
```python
# Wrong! Accessing without checking if res was updated
return s[res[0]:res[1]+1]  # Might return s[-1:0] if no solution!
```

**Fix:**
```python
return s[l:r+1] if res_len != float('inf') else ""
```

### ❌ Mistake 4: Wrong Window Length Calculation
```python
# Wrong! Off-by-one error
if (right - left) < res_len:
    res = [left, right]
    res_len = right - left
```

**Fix:**
```python
if (right - left + 1) < res_len:
    res = [left, right]
    res_len = right - left + 1
```

---

## Interview Insights

**Question Recognition:**
- "Minimum window" + "contains all characters" → Sliding window with hash maps
- "Substring" + "frequency requirement" → Two pointers with character counting

**Key Points to Mention:**

1. **Why sliding window?**
   - "We can use two pointers to avoid checking all O(n²) substrings. The window expands to find valid solutions and contracts to minimize."

2. **The have/need optimization:**
   - "Instead of comparing two hash maps in O(k) time, we use integers to track validity in O(1)."

3. **Time complexity:**
   - "Each character is visited at most twice (once by right, once by left), giving us O(m) where m is the length of s."

**Follow-up Questions:**

1. **Q:** "What if we want the longest window instead?"
   - **A:** Only update result when shrinking fails (window becomes invalid).

2. **Q:** "What if t can have characters not in s?"
   - **A:** Algorithm handles it—if a character in t doesn't exist in s, `have` never equals `need`.

3. **Q:** "Can you optimize space?"
   - **A:** If character set is fixed (e.g., lowercase letters), use arrays of size 26 instead of hash maps.

4. **Q:** "What if we need to find all minimum windows?"
   - **A:** Store all windows with the minimum length instead of just one.

**Optimization Discussion:**
- Already optimal at O(m + n) time
- Can reduce space to O(1) if character set is bounded (e.g., only lowercase English letters)

---

## Variations

### Variation 1: Find All Minimum Windows
```python
def minWindowAll(s, t):
    # Similar logic, but store all windows with min length
    target_counts = Counter(t)
    window_counts = {}
    have, need = 0, len(target_counts)
    
    all_windows = []
    min_len = float('inf')
    left = 0
    
    for right in range(len(s)):
        char = s[right]
        window_counts[char] = window_counts.get(char, 0) + 1
        
        if char in target_counts and window_counts[char] == target_counts[char]:
            have += 1
        
        while have == need:
            curr_len = right - left + 1
            
            if curr_len < min_len:
                min_len = curr_len
                all_windows = [s[left:right+1]]
            elif curr_len == min_len:
                all_windows.append(s[left:right+1])
            
            left_char = s[left]
            window_counts[left_char] -= 1
            
            if left_char in target_counts and window_counts[left_char] < target_counts[left_char]:
                have -= 1
            
            left += 1
    
    return all_windows
```

### Variation 2: Window with At Most K Distinct Characters
```python
def minWindowKDistinct(s, k):
    char_count = {}
    left = 0
    max_len = 0
    
    for right in range(len(s)):
        char_count[s[right]] = char_count.get(s[right], 0) + 1
        
        while len(char_count) > k:
            char_count[s[left]] -= 1
            if char_count[s[left]] == 0:
                del char_count[s[left]]
            left += 1
        
        max_len = max(max_len, right - left + 1)
    
    return max_len
```

### Variation 3: Using Array Instead of Hash Map (Fixed Character Set)
```python
def minWindow_array(s, t):
    if not t or not s:
        return ""
    
    # For lowercase + uppercase English letters (52 total)
    target_counts = [0] * 128  # ASCII
    window_counts = [0] * 128
    
    need = 0
    for char in t:
        if target_counts[ord(char)] == 0:
            need += 1
        target_counts[ord(char)] += 1
    
    have = 0
    res, res_len = [-1, -1], float('inf')
    left = 0
    
    for right in range(len(s)):
        char_idx = ord(s[right])
        window_counts[char_idx] += 1
        
        if window_counts[char_idx] == target_counts[char_idx]:
            have += 1
        
        while have == need:
            if (right - left + 1) < res_len:
                res = [left, right]
                res_len = right - left + 1
            
            left_char_idx = ord(s[left])
            window_counts[left_char_idx] -= 1
            
            if target_counts[left_char_idx] > 0 and window_counts[left_char_idx] < target_counts[left_char_idx]:
                have -= 1
            
            left += 1
    
    l, r = res
    return s[l:r+1] if res_len != float('inf') else ""
```

---

## Related Problems

1. **Substring with Concatenation of All Words (LeetCode 30)** - Similar window concept
2. **Find All Anagrams in a String (LeetCode 438)** - Fixed-size window variant
3. **Longest Substring with At Most K Distinct Characters (LeetCode 340)** - Similar two-pointer approach
4. **Permutation in String (LeetCode 567)** - Simpler version with fixed window
5. **Smallest Range Covering Elements from K Lists (LeetCode 632)** - Generalized version

---

## Summary

**Key Takeaways:**

1. **Algorithm:** Sliding window with hash maps
   - Expand right to find valid windows
   - Contract left to minimize window size

2. **Core Optimization:** Have/Need counters
   - `need` = number of unique characters in t
   - `have` = how many unique chars meet their requirement
   - O(1) validity check instead of O(k) map comparison

3. **Character Tracking:**
   - Increment `have` only when `window_counts[char] == target_counts[char]`
   - Decrement `have` only when `window_counts[char] < target_counts[char]`

4. **Complexity:**
   - Time: $O(m + n)$ - Linear in both string lengths
   - Space: $O(m + n)$ - Or O(1) with fixed character set

5. **Mental Model:** Think of a rubber band that stretches to include all required characters, then contracts to find the tightest fit, repeating this process across the entire string.

**Template:**
```python
target_counts = Counter(t)
window_counts = {}
have, need = 0, len(target_counts)
res, res_len = [-1, -1], float('inf')
left = 0

for right in range(len(s)):
    # Expand: add char at right
    char = s[right]
    window_counts[char] = window_counts.get(char, 0) + 1
    if char in target_counts and window_counts[char] == target_counts[char]:
        have += 1
    
    # Contract: shrink from left while valid
    while have == need:
        # Update result
        if (right - left + 1) < res_len:
            res = [left, right]
            res_len = right - left + 1
        
        # Remove char at left
        left_char = s[left]
        window_counts[left_char] -= 1
        if left_char in target_counts and window_counts[left_char] < target_counts[left_char]:
            have -= 1
        left += 1

l, r = res
return s[l:r+1] if res_len != float('inf') else ""
```