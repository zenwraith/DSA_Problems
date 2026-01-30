# Longest Substring Without Repeating Characters

**LeetCode 3** | **Difficulty:** Medium | **Category:** Sliding Window, Hash Map

---

## Problem Statement

Given a string `s`, find the length of the **longest substring** without repeating characters.

---

## Examples

### Example 1:
```
Input: s = "abcabcbb"
Output: 3

Explanation: The answer is "abc", with the length of 3.
```

### Example 2:
```
Input: s = "bbbbb"
Output: 1

Explanation: The answer is "b", with the length of 1.
```

### Example 3:
```
Input: s = "pwwkew"
Output: 3

Explanation: The answer is "wke", with the length of 3.
Notice that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

### Example 4:
```
Input: s = ""
Output: 0

Explanation: Empty string has no characters.
```

---

## Strategy

### The "Expanding Frontier" Approach

We use two pointers, `left` and `right`, to represent the current substring (window). As we move `right` to include a new character, we check: **"Have we seen this character already in our current window?"**

**Decision Flow:**

1. **If No (character is new):** 
   - The window is still valid
   - Calculate the length $(right - left + 1)$ and update the maximum

2. **If Yes (character is duplicate):** 
   - The window is now invalid
   - Move `left` to the position right after the last occurrence of the duplicate character
   - This effectively removes the duplicate from the window

**Key Insight:** We use a **hash map** (`char_map`) to store the most recent index of each character. This allows us to instantly know where we last saw a character and jump `left` directly to the right position.

**Complexity:**
- **Time:** $O(n)$ - Each character is visited at most twice (once by right, once when updating left)
- **Space:** $O(min(n, m))$ where m = size of character set (e.g., 128 for ASCII)

---

## Solution

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        char_map = {}
        left = 0
        max_length = 0
        
        for right in range(len(s)):
            char = s[right]
            
            # If the character is already in the map and within the current window
            if char in char_map and char_map[char] >= left:
                # Move 'left' to the position right after the last occurrence
                left = char_map[char] + 1
            
            # Update or add the character's latest index
            char_map[char] = right
            
            # Calculate current window size and update max_length
            max_length = max(max_length, right - left + 1)
        
        return max_length
```

---

## Detailed Walkthrough

### Example: `s = "abcabcbb"`

**Visual Representation:**

| Step | Right | Char | Char Map | Left | Window | Length | Max Length | Action |
|------|-------|------|----------|------|--------|--------|------------|--------|
| 0 | 0 | 'a' | {'a':0} | 0 | "a" | 1 | 1 | New char |
| 1 | 1 | 'b' | {'a':0, 'b':1} | 0 | "ab" | 2 | 2 | New char |
| 2 | 2 | 'c' | {'a':0, 'b':1, 'c':2} | 0 | "abc" | 3 | 3 | New char |
| 3 | 3 | 'a' | {'a':3, 'b':1, 'c':2} | 1 | "bca" | 3 | 3 | Duplicate! Move left to 1 |
| 4 | 4 | 'b' | {'a':3, 'b':4, 'c':2} | 2 | "cab" | 3 | 3 | Duplicate! Move left to 2 |
| 5 | 5 | 'c' | {'a':3, 'b':4, 'c':5} | 3 | "abc" | 3 | 3 | Duplicate! Move left to 3 |
| 6 | 6 | 'b' | {'a':3, 'b':6, 'c':5} | 5 | "cb" | 2 | 3 | Duplicate! Move left to 5 |
| 7 | 7 | 'b' | {'a':3, 'b':7, 'c':5} | 7 | "b" | 1 | 3 | Duplicate! Move left to 7 |

**Final Answer:** `3` (substring "abc")

**Step-by-Step Explanation:**

1. **Steps 0-2:** Build window "abc" with no duplicates → max_length = 3
2. **Step 3:** See 'a' again (was at index 0), move left to 1 → window becomes "bca"
3. **Step 4:** See 'b' again (was at index 1), move left to 2 → window becomes "cab"
4. **Step 5:** See 'c' again (was at index 2), move left to 3 → window becomes "abc"
5. **Steps 6-7:** Continue with duplicates, but max_length remains 3

---

## Critical Understanding: The `char_map[char] >= left` Check

### Why This Check Is Essential

**This is the most common bug in interviews!** Let's understand with an example:

### Example: `s = "abba"`

**Without the check (WRONG):**

| Step | Right | Char | Char Map | Left (Wrong) | What Happens |
|------|-------|------|----------|--------------|--------------|
| 0 | 0 | 'a' | {'a':0} | 0 | - |
| 1 | 1 | 'b' | {'a':0, 'b':1} | 0 | - |
| 2 | 2 | 'b' | {'a':0, 'b':2} | 2 | Duplicate 'b', left → 2 |
| 3 | 3 | 'a' | {'a':3, 'b':2} | **1** | 'a' was at index 0, left moves to 0+1=1 **WRONG!** |

**Problem:** Left moves **backward** from 2 to 1, which would include the duplicate 'b' again!

**With the check (CORRECT):**

| Step | Right | Char | Char Map | Left (Correct) | Window | Action |
|------|-------|------|----------|----------------|--------|--------|
| 0 | 0 | 'a' | {'a':0} | 0 | "a" | - |
| 1 | 1 | 'b' | {'a':0, 'b':1} | 0 | "ab" | - |
| 2 | 2 | 'b' | {'a':0, 'b':2} | 2 | "b" | Duplicate, left → 2 |
| 3 | 3 | 'a' | {'a':3, 'b':2} | **2** | "ba" | 'a' at index 0 < left (2), skip! |

**Why it works:**
- When we see 'a' at index 3, the map says it was last seen at index 0
- But our `left` is already at 2, which is past index 0
- The check `char_map[char] >= left` fails (0 >= 2 is False)
- So we **don't move left backward**
- Window remains valid: "ba"

**Key Rule:** The window can only **shrink or stay the same size**, never expand backward by moving `left` to the left.

---

## Alternative Approach: Using a Set

### Set-Based Solution

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        char_set = set()
        left = 0
        max_length = 0
        
        for right in range(len(s)):
            # Shrink window until no duplicates
            while s[right] in char_set:
                char_set.remove(s[left])
                left += 1
            
            # Add current character
            char_set.add(s[right])
            
            # Update max length
            max_length = max(max_length, right - left + 1)
        
        return max_length
```

**Comparison:**

| Approach | Space | Logic | Performance |
|----------|-------|-------|-------------|
| **Hash Map** | $O(n)$ | Jump left directly to position | Fewer iterations |
| **Set** | $O(n)$ | Shrink left one by one | More iterations, simpler logic |

**Trade-off:**
- **Hash Map:** Faster (can skip multiple characters), but requires careful boundary checking
- **Set:** Slightly slower (removes one character at a time), but simpler to understand

---

## Edge Cases

### 1. Empty String
```python
s = ""
# Output: 0
# No characters, no substring
```

### 2. All Same Characters
```python
s = "aaaa"
# Output: 1
# Every character is duplicate, max length is 1
```

### 3. All Unique Characters
```python
s = "abcdef"
# Output: 6
# Entire string is the answer
```

### 4. Single Character
```python
s = "a"
# Output: 1
```

### 5. Two Characters Alternating
```python
s = "abababab"
# Output: 2
# "ab" is the longest
```

### 6. Duplicates at Different Positions
```python
s = "dvdf"
# Output: 3
# "vdf" is the longest
```

---

## Common Mistakes

### ❌ Mistake 1: Not Checking if Character is in Current Window
```python
# Wrong! Doesn't check if char is in current window
if char in char_map:
    left = char_map[char] + 1  # Might move left backward!
```

**Fix:**
```python
# Correct! Check if it's within current window
if char in char_map and char_map[char] >= left:
    left = char_map[char] + 1
```

### ❌ Mistake 2: Updating Map Before Moving Left
```python
# Wrong order!
char_map[char] = right  # Update first
if char in char_map and char_map[char] >= left:
    left = char_map[char] + 1  # Now char_map[char] == right, always true!
```

**Fix:** Check first, then update.

### ❌ Mistake 3: Calculating Length Wrong
```python
# Wrong! Off-by-one error
max_length = max(max_length, right - left)  # Should be +1
```

**Fix:**
```python
# Correct! Window from left to right inclusive
max_length = max(max_length, right - left + 1)
```

### ❌ Mistake 4: Using While Loop to Remove Single Character
```python
# Inefficient! Hash map allows direct jump
while char in char_set:
    char_set.remove(s[left])
    left += 1
```

**Improvement:** Use hash map to jump directly to position after duplicate.

---

## Interview Insights

**Question Recognition:**
- "Longest substring" + "without repeating" → Sliding window with hash map
- "Contiguous" + "condition" → Sliding window pattern

**Key Points to Mention:**

1. **Why hash map over set?**
   - "Hash map stores indices, allowing us to jump left pointer directly to the right position instead of incrementing one by one."

2. **The boundary check:**
   - "We must check `char_map[char] >= left` to ensure we don't move left backward, which would break the window validity."

3. **Time complexity:**
   - "Each character is visited at most twice—once when right expands, and once when left catches up. This gives us O(n)."

**Follow-up Questions:**

1. **Q:** "What if we need to find the actual substring, not just length?"
   - **A:** Track `start` and `max_length` separately, return `s[start:start+max_length]`.

2. **Q:** "What if we allow at most K duplicate characters?"
   - **A:** Use a frequency map and track count of characters with frequency > 0.

3. **Q:** "Can you solve with O(1) space?"
   - **A:** If character set is fixed (e.g., lowercase letters only), use array of size 26 instead of hash map.

4. **Q:** "What about Unicode characters?"
   - **A:** Hash map handles any character set, unlike fixed-size arrays.

**Optimization Discussion:**
- Already optimal at O(n) time
- Space can be reduced to O(1) if character set is bounded and small

---

## Variations

### Variation 1: Return the Actual Substring
```python
def longestSubstringText(s):
    char_map = {}
    left = 0
    max_length = 0
    start_index = 0
    
    for right in range(len(s)):
        char = s[right]
        
        if char in char_map and char_map[char] >= left:
            left = char_map[char] + 1
        
        char_map[char] = right
        
        if right - left + 1 > max_length:
            max_length = right - left + 1
            start_index = left
    
    return s[start_index : start_index + max_length]
```

### Variation 2: At Most K Distinct Characters
```python
def lengthOfLongestSubstringKDistinct(s, k):
    char_count = {}
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        char = s[right]
        char_count[char] = char_count.get(char, 0) + 1
        
        # Shrink if more than k distinct characters
        while len(char_count) > k:
            left_char = s[left]
            char_count[left_char] -= 1
            if char_count[left_char] == 0:
                del char_count[left_char]
            left += 1
        
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

### Variation 3: Fixed Character Set (Lowercase Letters Only)
```python
def lengthOfLongestSubstring_array(s):
    # Use array for O(1) space with fixed character set
    last_index = [-1] * 26  # For 'a' to 'z'
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        char_idx = ord(s[right]) - ord('a')
        
        if last_index[char_idx] >= left:
            left = last_index[char_idx] + 1
        
        last_index[char_idx] = right
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

---

## Related Problems

1. **Longest Substring with At Most Two Distinct Characters (LeetCode 159)** - Similar sliding window
2. **Longest Substring with At Most K Distinct Characters (LeetCode 340)** - Generalized version
3. **Minimum Window Substring (LeetCode 76)** - More complex shrinking condition
4. **Permutation in String (LeetCode 567)** - Fixed-size window variant
5. **Longest Repeating Character Replacement (LeetCode 424)** - With allowed changes

---

## Summary

**Key Takeaways:**

1. **Pattern:** Variable-size sliding window with hash map
   - Expand right to explore
   - Shrink left when duplicate found

2. **Critical Check:** `char_map[char] >= left`
   - Prevents moving left backward
   - Ensures window validity

3. **Data Structure Choice:**
   - Hash map: Jump directly to position (faster)
   - Set: Shrink one by one (simpler)

4. **Complexity:**
   - Time: $O(n)$ - Each character visited at most twice
   - Space: $O(min(n, m))$ - m = character set size

5. **Mental Model:** Think of a rubber band stretching right. When it hits a duplicate, it snaps back just past the previous occurrence of that character, maintaining uniqueness.

**Template:**
```python
char_map = {}
left = 0
max_length = 0

for right in range(len(s)):
    # Check for duplicate in current window
    if s[right] in char_map and char_map[char] >= left:
        left = char_map[s[right]] + 1
    
    # Update position
    char_map[s[right]] = right
    
    # Update result
    max_length = max(max_length, right - left + 1)

return max_length
```
