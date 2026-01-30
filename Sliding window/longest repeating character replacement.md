# Longest Repeating Character Replacement

**LeetCode 424** | **Difficulty:** Medium | **Category:** Sliding Window, Hash Map

---

## Problem Statement

You are given a string `s` and an integer `k`. You can choose any character of the string and change it to any other uppercase English character. You can perform this operation **at most** `k` times.

Return the **length of the longest substring** containing the same letter you can get after performing the above operations.

---

## Examples

### Example 1:
```
Input: s = "ABAB", k = 2
Output: 4

Explanation: 
Replace the two 'A's with two 'B's or vice versa.
Result: "AAAA" or "BBBB"
```

### Example 2:
```
Input: s = "AABABBA", k = 1
Output: 4

Explanation: 
Replace the one 'A' in the middle to 'B' and form "AABBBBA".
The substring "BBBB" has the longest repeating letters, which is 4.
```

### Example 3:
```
Input: s = "ABBB", k = 2
Output: 4

Explanation:
Change 'A' to 'B' to get "BBBB".
```

---

## Strategy

### Window Validity with Maximum Frequency Optimization

**Key Insight:** In any window, if you want to turn all characters into the same one, the most logical choice is to **change everything into the most frequent character** already in that window.

### The Golden Rule

A window is **"valid"** if:

$$\text{(Window Length)} - \text{(Max Frequency)} \le k$$

Where:
- **Window Length:** `right - left + 1`
- **Max Frequency:** The count of the most frequent character in the window
- **Difference:** These are the "other" characters we must replace

**If this count is ≤ k, we can make the whole window uniform.**

**Example:**
```
Window: "AABBA", k = 1
- Window Length: 5
- Most frequent character: 'A' or 'B' (both appear 3 times, so max_freq = 3)
- Characters to replace: 5 - 3 = 2
- Is valid? 2 > 1 (k), so NO, invalid window
```

**Algorithm:**

1. **Expand:** Move `right` pointer and add character to window
2. **Update Max Frequency:** Track the highest character frequency seen in the current window
3. **Check Validity:** If `window_length - max_freq > k`, shrink from left
4. **Track Result:** Record maximum valid window length

**Why This Works:**
- We only need to replace the **least frequent** characters
- The most frequent character should remain unchanged
- If we can replace all other characters with at most k operations, the window is valid

**Complexity:**
- **Time:** $O(n)$ - Each character visited at most twice
- **Space:** $O(26) = O(1)$ - At most 26 uppercase letters

---

## Solution

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        count = {}
        max_freq = 0
        left = 0
        max_length = 0
        
        for right in range(len(s)):
            # 1. Expand the window by adding character at right
            char = s[right]
            count[char] = count.get(char, 0) + 1
            
            # 2. Update the frequency of the most common character in the window
            max_freq = max(max_freq, count[char])
            
            # 3. Check if the window is valid
            # If (window length - max_freq) > k, it's invalid
            while (right - left + 1) - max_freq > k:
                # Shrink from the left
                count[s[left]] -= 1
                left += 1
                # Note: We don't need to update max_freq here!
                # (See optimization explanation below)
            
            # 4. Update result with current valid window
            max_length = max(max_length, right - left + 1)
        
        return max_length
```

---

## Detailed Walkthrough

### Example: `s = "AABABBA"`, `k = 1`

**Step-by-Step Execution:**

| Step | Right | Char | Count | Max Freq | Window | Length | Replacements Needed | Valid? | Max Length |
|------|-------|------|-------|----------|--------|--------|---------------------|--------|------------|
| 0 | 0 | 'A' | {'A':1} | 1 | "A" | 1 | 1-1=0 | ✅ | 1 |
| 1 | 1 | 'A' | {'A':2} | 2 | "AA" | 2 | 2-2=0 | ✅ | 2 |
| 2 | 2 | 'B' | {'A':2,'B':1} | 2 | "AAB" | 3 | 3-2=1 | ✅ | 3 |
| 3 | 3 | 'A' | {'A':3,'B':1} | 3 | "AABA" | 4 | 4-3=1 | ✅ | 4 |
| 4 | 4 | 'B' | {'A':3,'B':2} | 3 | "AABAB" | 5 | 5-3=2 | ❌ | 4 |
| 4a | 4 | - | {'A':2,'B':2} | 3 | "ABAB" | 4 | 4-3=1 | ✅ | 4 |
| 5 | 5 | 'B' | {'A':2,'B':3} | 3 | "ABABB" | 5 | 5-3=2 | ❌ | 4 |
| 5a | 5 | - | {'A':1,'B':3} | 3 | "BABB" | 4 | 4-3=1 | ✅ | 4 |
| 6 | 6 | 'A' | {'A':2,'B':3} | 3 | "BABBA" | 5 | 5-3=2 | ❌ | 4 |
| 6a | 6 | - | {'A':2,'B':2} | 3 | "ABBA" | 4 | 4-3=1 | ✅ | 4 |

**Final Answer:** `4` (substring "AABA" or "ABBA" or "BABB")

**Detailed Explanation:**

**Step 3:** Window "AABA"
- Count: {'A': 3, 'B': 1}
- Max frequency: 3 (character 'A')
- Replacements needed: 4 - 3 = 1
- Can we do this with k=1? Yes! Replace 'B' with 'A' → "AAAA"

**Step 4:** Window "AABAB"
- Count: {'A': 3, 'B': 2}
- Max frequency: 3
- Replacements needed: 5 - 3 = 2
- Can we do this with k=1? No! Need to shrink

**Step 4a:** Shrink to "ABAB"
- Remove left character 'A'
- Count: {'A': 2, 'B': 2}
- Replacements needed: 4 - 3 = 1 (note: max_freq stays 3)
- Valid again!

---

## Critical Understanding: The "Max Freq" Optimization

### Why Don't We Recalculate `max_freq` When Shrinking?

**You might notice:**
When we shrink the window (`left += 1`), we **don't re-calculate `max_freq`** by scanning the whole hash map.

**Why?**

Because `max_length` **only increases** when we find a window with a **higher `max_freq`** than we've ever seen before. Even if the current window's max frequency drops, it won't help us find a new record-breaking `max_length`. We only care about `max_freq` when it's pushing our boundary further.

**Example to Illustrate:**

```
Window: "AABBB", k = 1
max_freq = 3 (from 'B')
max_length = 5 (valid: 5 - 3 = 2 > 1, so we need to shrink)

After shrinking to "ABBB":
Actual max freq in current window = 3 ('B')
But max_freq variable = 3 (unchanged)

Does it matter? NO!
- If we kept max_freq = 3, we check: 4 - 3 = 1 ≤ 1 → Valid
- If we updated to actual = 3, we check: 4 - 3 = 1 ≤ 1 → Valid

Both give same answer!
```

**Key Insight:**

The `max_freq` variable represents the **highest frequency we've ever seen** in any window configuration. When we shrink:
- If the actual max frequency in the current window is less than our recorded `max_freq`, it doesn't matter because we're looking for a **larger** window
- We can only get a larger window if we find a character with frequency **higher** than our current `max_freq`
- So keeping the "stale" `max_freq` is actually an optimization!

**Mathematical Proof:**

For a window to be **better** than our current `max_length`:
```
new_window_length > max_length

For this to happen with the same max_freq:
(right - left + 1) > max_length
But we only increment max_length when we find a valid window
So we need: (right - left + 1) - max_freq ≤ k

If max_freq stays the same, we can only get a longer window by expanding right
This will eventually make (right - left + 1) - max_freq > k
Which forces us to shrink

So the ONLY way to beat our record is to find a HIGHER max_freq!
```

---

## Alternative: Strict Max Freq Tracking

If you want to be more "correct" and actually track the true max frequency:

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        count = {}
        left = 0
        max_length = 0
        
        for right in range(len(s)):
            char = s[right]
            count[char] = count.get(char, 0) + 1
            
            # Calculate actual max frequency in current window
            max_freq = max(count.values())
            
            while (right - left + 1) - max_freq > k:
                count[s[left]] -= 1
                if count[s[left]] == 0:
                    del count[s[left]]
                left += 1
                # Recalculate max_freq after shrinking
                max_freq = max(count.values()) if count else 0
            
            max_length = max(max_length, right - left + 1)
        
        return max_length
```

**Complexity:** $O(n \times 26)$ since `max(count.values())` is O(26)

**Verdict:** The optimized version (not recalculating) is **faster** and gives the same answer!

---

## Edge Cases

### 1. All Same Characters
```python
s = "AAAA", k = 2
# Output: 4
# Already all same, no replacements needed
```

### 2. k = 0 (No Replacements Allowed)
```python
s = "ABCDE", k = 0
# Output: 1
# Can only take single character windows
```

### 3. k ≥ String Length
```python
s = "ABCD", k = 5
# Output: 4
# Can replace all characters to make entire string uniform
```

### 4. Two Characters Alternating
```python
s = "ABABAB", k = 2
# Output: 5
# Replace 2 'A's or 2 'B's
```

### 5. Single Character
```python
s = "A", k = 1
# Output: 1
```

---

## Common Mistakes

### ❌ Mistake 1: Recalculating Max Freq Unnecessarily
```python
# Slow! O(26) per iteration
while (right - left + 1) - max(count.values()) > k:
    count[s[left]] -= 1
    left += 1
```

**Fix:** Track `max_freq` as a variable and update only when a character count increases.

### ❌ Mistake 2: Wrong Window Validity Check
```python
# Wrong! Should be >, not >=
while (right - left + 1) - max_freq >= k:
    left += 1
```

**Fix:** Use `> k` because we can replace **at most** k characters.

### ❌ Mistake 3: Not Updating Result After Shrinking
```python
# Wrong! Only updates when expanding
for right in range(len(s)):
    # ... expand and check validity ...
    if valid:
        max_length = max(max_length, right - left + 1)
```

**Fix:** Always update `max_length` after ensuring window is valid (after the while loop).

### ❌ Mistake 4: Deleting from Count Dictionary
```python
# Unnecessary and can cause issues
if count[s[left]] == 0:
    del count[s[left]]
```

**Fix:** Keep zero counts in the dictionary; they don't affect `max_freq` calculation.

---

## Interview Insights

**Question Recognition:**
- "Longest substring" + "with replacements allowed" → Sliding window with frequency tracking
- "At most k changes" → Window validity condition

**Key Points to Mention:**

1. **Why track max frequency?**
   - "We want to keep the most frequent character and replace all others. The cost is (window length - max frequency)."

2. **The optimization:**
   - "We don't need to recalculate max_freq when shrinking because we only care about finding a larger max_freq to beat our record."

3. **Time complexity:**
   - "Each character is added once and removed at most once, giving us O(n) time with the max_freq optimization."

**Follow-up Questions:**

1. **Q:** "What if we want the actual substring, not just length?"
   - **A:** Track the starting index when we update max_length: `if new_length > max_length: start = left`.

2. **Q:** "What if lowercase and uppercase are different?"
   - **A:** The algorithm works the same since we treat each character distinctly.

3. **Q:** "Can you solve with exactly k replacements instead of at most?"
   - **A:** Need different approach—track number of replacements explicitly and maintain exactly k.

4. **Q:** "What if you can only replace specific characters?"
   - **A:** Modify the cost calculation to only count replaceable characters.

**Optimization Discussion:**
- Already optimal at O(n) with the max_freq variable approach
- Without optimization, it's O(26n) which is still linear but slower

---

## Variations

### Variation 1: Return the Actual Substring
```python
def characterReplacementWithSubstring(s, k):
    count = {}
    max_freq = 0
    left = 0
    max_length = 0
    result_start = 0
    
    for right in range(len(s)):
        char = s[right]
        count[char] = count.get(char, 0) + 1
        max_freq = max(max_freq, count[char])
        
        while (right - left + 1) - max_freq > k:
            count[s[left]] -= 1
            left += 1
        
        if (right - left + 1) > max_length:
            max_length = right - left + 1
            result_start = left
    
    return s[result_start : result_start + max_length]
```

### Variation 2: With Lowercase Letters
```python
def characterReplacement_lowercase(s, k):
    # Same algorithm, works for any characters
    count = {}
    max_freq = 0
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        char = s[right]
        count[char] = count.get(char, 0) + 1
        max_freq = max(max_freq, count[char])
        
        while (right - left + 1) - max_freq > k:
            count[s[left]] -= 1
            left += 1
        
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

### Variation 3: Longest Subarray with Ones After Replacement
```python
def longestOnesWithReplacement(nums, k):
    """LeetCode 1004: Max Consecutive Ones III"""
    left = 0
    zero_count = 0
    max_length = 0
    
    for right in range(len(nums)):
        if nums[right] == 0:
            zero_count += 1
        
        while zero_count > k:
            if nums[left] == 0:
                zero_count -= 1
            left += 1
        
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

---

## Related Problems

1. **Max Consecutive Ones III (LeetCode 1004)** - Flip at most k zeros to ones
2. **Longest Substring with At Most K Distinct Characters (LeetCode 340)** - Similar window validity
3. **Minimum Window Substring (LeetCode 76)** - More complex validity condition
4. **Longest Substring Without Repeating Characters (LeetCode 3)** - Simpler sliding window
5. **Subarrays with K Different Integers (LeetCode 992)** - Advanced sliding window

---

## Summary

**Key Takeaways:**

1. **Core Idea:** Keep the most frequent character, replace all others
   - Valid window: `(window_length - max_freq) ≤ k`

2. **Algorithm:** Variable-size sliding window
   - Expand right to explore
   - Shrink left when invalid
   - Track maximum valid window

3. **Optimization:** Don't recalculate `max_freq` when shrinking
   - Only need higher `max_freq` to beat our record
   - Keeping "stale" max_freq is safe and faster

4. **Complexity:**
   - Time: $O(n)$ - Each character visited at most twice
   - Space: $O(26) = O(1)$ - Fixed character set

5. **Mental Model:** Think of trying to "paint" the entire window with the most common color already present. You have k paint strokes. If you need more than k strokes, shrink the window.

**Template:**
```python
count = {}
max_freq = 0
left = 0
max_length = 0

for right in range(len(s)):
    # Expand: add character
    count[s[right]] = count.get(s[right], 0) + 1
    max_freq = max(max_freq, count[s[right]])
    
    # Contract: shrink while invalid
    while (right - left + 1) - max_freq > k:
        count[s[left]] -= 1
        left += 1
    
    # Update result
    max_length = max(max_length, right - left + 1)

return max_length
```