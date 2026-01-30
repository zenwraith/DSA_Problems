# Permutation in String

**LeetCode 567** | **Difficulty:** Medium | **Category:** Sliding Window, Hash Map, String

---

## Problem Statement

Given two strings `s1` and `s2`, return `true` if `s2` contains a **permutation** of `s1`, or `false` otherwise.

In other words, return `true` if one of `s1`'s permutations is a substring of `s2`.

---

## Examples

### Example 1:
```
Input: s1 = "ab", s2 = "eidbaooo"
Output: true

Explanation: 
s2 contains one permutation of s1 ("ba").
```

### Example 2:
```
Input: s1 = "ab", s2 = "eidboaoo"
Output: false

Explanation:
No permutation of s1 exists in s2.
```

### Example 3:
```
Input: s1 = "adc", s2 = "dcda"
Output: true

Explanation:
s2 contains the permutation "cda" which is a permutation of s1.
```

---

## Strategy

### Fixed-Size Sliding Window with Frequency Matching

A **permutation** means the characters appear in the same frequency, but possibly in a different order. We need to find a substring of `s2` with length `len(s1)` that has the exact same character frequencies as `s1`.

**Key Insights:**

1. **Fixed Window Size:** The window size is exactly `len(s1)` because permutations have the same length
2. **Frequency Matching:** Two strings are permutations if their character frequencies match
3. **Sliding Window:** Check all windows of size `len(s1)` in `s2`

**Algorithm:**

1. **Build Frequency Maps:**
   - Create frequency count for `s1`
   - Create frequency count for first `len(s1)` characters of `s2`

2. **Check Initial Window:**
   - If frequencies match, return `true`

3. **Slide Window:**
   - Add new character at right (increment count)
   - Remove old character at left (decrement count)
   - Check if frequencies match

**Why This Works:**
- **Fixed size:** We only need to check windows of exact length
- **Efficient updates:** Only update two characters per slide (one in, one out)
- **Direct comparison:** Arrays can be compared in O(26) for lowercase letters

**Complexity:**
- **Time:** $O(n_1 + n_2 \times 26) = O(n_1 + n_2)$ where $n_1$ = len(s1), $n_2$ = len(s2)
- **Space:** $O(1)$ - Fixed-size arrays (26 for lowercase letters)

---

## Solution

```python
class Solution:
    def checkInclusion(self, s1: str, s2: str) -> bool:
        n1, n2 = len(s1), len(s2)
        
        # If s1 is longer than s2, no permutation can exist
        if n1 > n2:
            return False
        
        # Frequency arrays for lowercase letters (a-z)
        s1_counts = [0] * 26
        s2_counts = [0] * 26
        
        # Build initial window (first n1 characters)
        for i in range(n1):
            s1_counts[ord(s1[i]) - ord('a')] += 1
            s2_counts[ord(s2[i]) - ord('a')] += 1
        
        # Check if initial window matches
        if s1_counts == s2_counts:
            return True
        
        # Slide the window across s2
        for i in range(n1, n2):
            # Add new character at right (entering window)
            s2_counts[ord(s2[i]) - ord('a')] += 1
            
            # Remove old character at left (leaving window)
            s2_counts[ord(s2[i - n1]) - ord('a')] -= 1
            
            # Check if current window matches
            if s1_counts == s2_counts:
                return True
        
        return False
```

---

## Detailed Walkthrough

### Understanding Window Initialization and Sliding

#### 1. Initializing the Window (Indices 0 to $n_1 - 1$)

Before we can start sliding, we have to build the **first window**.

- `s1` has a length of $n_1$
- We need to look at the first $n_1$ characters of `s2` to see if they match

**Code:**
```python
for i in range(n1):
    s1_counts[...] += 1
    s2_counts[...] += 1
```

At the end of this loop, `s2_counts` represents the substring `s2[0:n1]`.

#### 2. Sliding the Window (Indices $n_1$ to $n_2 - 1$)

Now that the first window is ready, we start moving it. To move the window one step to the right:

1. **Add** the character at the new right boundary
2. **Remove** the character that is now outside the left boundary

**The Math for the "Left" Index:**

When the loop index `i` is at position $n_1$, the character we need to remove is at `i - n1`:

- When $i = n_1$ and window size is $n_1$: Remove index $n_1 - n_1 = 0$
- When $i = n_1 + 1$: Remove index $(n_1 + 1) - n_1 = 1$
- When $i = n_1 + 2$: Remove index $(n_1 + 2) - n_1 = 2$

**Why start at $n_1$?** If we started at 0, we would try to remove characters at negative indices and double-count the beginning!

### Example: `s1 = "ab"`, `s2 = "eidbaooo"`

**Setup:**
- `n1 = 2`, `n2 = 8`
- `s1_counts = [1, 1, 0, 0, ..., 0]` (one 'a', one 'b')

**Step-by-Step Execution:**

| Step | i | Action | Window | s2_counts | Match? |
|------|---|--------|--------|-----------|--------|
| Init | 0-1 | Build initial window | "ei" | [0,0,0,0,1,0,0,0,1,...] | ❌ |
| Slide 1 | 2 | Add 'd', Remove 'e' | "id" | [0,0,0,1,0,0,0,0,1,...] | ❌ |
| Slide 2 | 3 | Add 'b', Remove 'i' | "db" | [0,1,0,1,0,0,0,0,0,...] | ❌ |
| Slide 3 | 4 | Add 'a', Remove 'd' | "ba" | [1,1,0,0,0,0,0,0,0,...] | ✅ **Match!** |

**Detailed Steps:**

**Initialization (i = 0 to 1):**
```
s2[0] = 'e' → s2_counts[4] = 1
s2[1] = 'i' → s2_counts[8] = 1
Window: "ei"
s2_counts ≠ s1_counts
```

**Slide 1 (i = 2):**
```
Add: s2[2] = 'd' → s2_counts[3] += 1
Remove: s2[2-2] = s2[0] = 'e' → s2_counts[4] -= 1
Window: "id"
s2_counts ≠ s1_counts
```

**Slide 2 (i = 3):**
```
Add: s2[3] = 'b' → s2_counts[1] += 1
Remove: s2[3-2] = s2[1] = 'i' → s2_counts[8] -= 1
Window: "db"
s2_counts ≠ s1_counts
```

**Slide 3 (i = 4):**
```
Add: s2[4] = 'a' → s2_counts[0] += 1
Remove: s2[4-2] = s2[2] = 'd' → s2_counts[3] -= 1
Window: "ba"
s2_counts = [1,1,0,...,0] = s1_counts ✅
Return true!
```

**Visual Window Movement:**

| Step | Loop Index i | Action | Window in s2 |
|------|-------------|--------|--------------|
| Init | 0 to n1-1 | Fill initial counts | s2[0...n1-1] = "ei" |
| Slide 1 | n1 (2) | Add s2[2], Remove s2[0] | s2[1...2] = "id" |
| Slide 2 | n1+1 (3) | Add s2[3], Remove s2[1] | s2[2...3] = "db" |
| Slide 3 | n1+2 (4) | Add s2[4], Remove s2[2] | s2[3...4] = "ba" ✅ |

---

## Optimization: The Matches Variable

### The $O(26)$ vs $O(1)$ Comparison

In the code above, we compare two arrays of size 26 in every step. This is technically $O(26 \times n_2) = O(n_2)$, but we can optimize further.

**Current Approach:**
```python
if s1_counts == s2_counts:  # O(26) comparison
    return True
```

**Optimized Approach:**
To get to **true $O(1)$ per comparison**, maintain a `matches` variable (0 to 26):
- Initialize `matches` to count how many indices already match
- When sliding, only update `matches` for the two characters that changed (one entering, one leaving)
- Return `true` when `matches == 26`

**Optimized Implementation:**

```python
class Solution:
    def checkInclusion(self, s1: str, s2: str) -> bool:
        n1, n2 = len(s1), len(s2)
        
        if n1 > n2:
            return False
        
        s1_counts = [0] * 26
        s2_counts = [0] * 26
        
        # Build initial window
        for i in range(n1):
            s1_counts[ord(s1[i]) - ord('a')] += 1
            s2_counts[ord(s2[i]) - ord('a')] += 1
        
        # Count initial matches
        matches = 0
        for i in range(26):
            if s1_counts[i] == s2_counts[i]:
                matches += 1
        
        # Check initial window
        if matches == 26:
            return True
        
        # Slide the window
        for i in range(n1, n2):
            # Add character at right
            right_char_idx = ord(s2[i]) - ord('a')
            
            # Check if adding this character affects matches
            if s2_counts[right_char_idx] == s1_counts[right_char_idx]:
                matches -= 1  # Was matching, now won't match
            s2_counts[right_char_idx] += 1
            if s2_counts[right_char_idx] == s1_counts[right_char_idx]:
                matches += 1  # Now matches
            
            # Remove character at left
            left_char_idx = ord(s2[i - n1]) - ord('a')
            
            # Check if removing this character affects matches
            if s2_counts[left_char_idx] == s1_counts[left_char_idx]:
                matches -= 1  # Was matching, now won't match
            s2_counts[left_char_idx] -= 1
            if s2_counts[left_char_idx] == s1_counts[left_char_idx]:
                matches += 1  # Now matches
            
            if matches == 26:
                return True
        
        return False
```

**Complexity:**
- **Time:** $O(n_1 + n_2)$ - True linear time with O(1) comparisons
- **Space:** $O(1)$ - Still just two arrays of size 26

---

## Edge Cases

### 1. s1 Longer Than s2
```python
s1 = "abc", s2 = "ab"
# Output: false
# Impossible to have a permutation
```

### 2. Exact Match
```python
s1 = "abc", s2 = "abc"
# Output: true
# s2 is itself a permutation of s1
```

### 3. Single Character
```python
s1 = "a", s2 = "ab"
# Output: true
# First character matches
```

### 4. All Same Characters
```python
s1 = "aaa", s2 = "aaacb"
# Output: true
# First three characters are a permutation
```

### 5. No Match Exists
```python
s1 = "ab", s2 = "eidboaoo"
# Output: false
# No window of size 2 has one 'a' and one 'b'
```

### 6. Permutation at End
```python
s1 = "ab", s2 = "oooba"
# Output: false
# "ba" would be at indices [4,5], but s2 is only length 5
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Loop Starting Point
```python
# Wrong! Should start at n1, not 0
for i in range(0, n2):
    s2_counts[ord(s2[i]) - ord('a')] += 1
    s2_counts[ord(s2[i - n1]) - ord('a')] -= 1
```

**Problem:** When `i = 0`, `i - n1` is negative!

**Fix:** Start loop at `n1`.

### ❌ Mistake 2: Forgetting to Check Initial Window
```python
# Wrong! Missing initial check
for i in range(n1):
    s1_counts[...] += 1
    s2_counts[...] += 1

# Directly start sliding without checking initial window
for i in range(n1, n2):
    ...
```

**Fix:** Check `if s1_counts == s2_counts` after building initial window.

### ❌ Mistake 3: Comparing Wrong Indices
```python
# Wrong! Should compare with i - n1, not i - 1
s2_counts[ord(s2[i - 1]) - ord('a')] -= 1
```

**Fix:** Use `i - n1` to remove the character that's leaving the window.

### ❌ Mistake 4: Using Hash Map for Fixed Character Set
```python
# Inefficient! Use array for known character set
s1_counts = Counter(s1)
s2_counts = Counter()
```

**Better:** Use array `[0] * 26` for lowercase letters (faster and simpler).

---

## Interview Insights

**Question Recognition:**
- "Permutation" + "substring" → Fixed-size sliding window with frequency matching
- "Contains any arrangement" → Check character frequencies

**Key Points to Mention:**

1. **Why fixed-size window?**
   - "Permutations have the same length, so we only check windows of size len(s1)."

2. **Array vs. Hash Map:**
   - "Since we know the character set (lowercase letters), using an array of size 26 is more efficient than a hash map."

3. **Optimization opportunity:**
   - "We can optimize from O(26n) to O(n) by maintaining a matches counter instead of comparing arrays each time."

**Follow-up Questions:**

1. **Q:** "What if the strings contain uppercase letters and special characters?"
   - **A:** Use a hash map instead of array, or use array of size 128 for ASCII.

2. **Q:** "How would you find all starting indices of permutations?"
   - **A:** Similar approach, but collect all indices where frequencies match instead of returning early.

3. **Q:** "What if s1 is very large?"
   - **A:** Algorithm is still O(n), but initial window building takes O(n1) time.

4. **Q:** "Can you solve without extra space?"
   - **A:** Arrays of size 26 are O(1) space (constant, not dependent on input size).

---

## Variations

### Variation 1: Find All Permutation Indices
```python
def findAnagrams(s, p):
    """LeetCode 438: Find All Anagrams in a String"""
    result = []
    n1, n2 = len(p), len(s)
    
    if n1 > n2:
        return result
    
    p_counts = [0] * 26
    s_counts = [0] * 26
    
    for i in range(n1):
        p_counts[ord(p[i]) - ord('a')] += 1
        s_counts[ord(s[i]) - ord('a')] += 1
    
    if p_counts == s_counts:
        result.append(0)
    
    for i in range(n1, n2):
        s_counts[ord(s[i]) - ord('a')] += 1
        s_counts[ord(s[i - n1]) - ord('a')] -= 1
        
        if p_counts == s_counts:
            result.append(i - n1 + 1)
    
    return result
```

### Variation 2: With Hash Map (General Character Set)
```python
def checkInclusion_hashmap(s1, s2):
    from collections import Counter
    
    n1, n2 = len(s1), len(s2)
    
    if n1 > n2:
        return False
    
    s1_counts = Counter(s1)
    window_counts = Counter(s2[:n1])
    
    if s1_counts == window_counts:
        return True
    
    for i in range(n1, n2):
        # Add new character
        window_counts[s2[i]] += 1
        
        # Remove old character
        window_counts[s2[i - n1]] -= 1
        if window_counts[s2[i - n1]] == 0:
            del window_counts[s2[i - n1]]
        
        if s1_counts == window_counts:
            return True
    
    return False
```

---

## Related Problems

1. **Find All Anagrams in a String (LeetCode 438)** - Return all starting indices
2. **Minimum Window Substring (LeetCode 76)** - Variable-size window version
3. **Longest Substring Without Repeating Characters (LeetCode 3)** - Variable-size window
4. **Substring with Concatenation of All Words (LeetCode 30)** - Multiple fixed-size patterns
5. **Valid Anagram (LeetCode 242)** - Simpler version comparing two strings

---

## Summary

**Key Takeaways:**

1. **Pattern:** Fixed-size sliding window with frequency matching
   - Window size = len(s1)
   - Check if character frequencies match

2. **Algorithm Steps:**
   - Build frequency maps for s1 and first window of s2
   - Check initial window
   - Slide window: add right, remove left, check frequencies

3. **Optimization:** Matches variable
   - Track how many indices match (0-26)
   - Update only for changed characters
   - O(1) comparison instead of O(26)

4. **Complexity:**
   - Time: $O(n_1 + n_2)$ - Linear in both string lengths
   - Space: $O(1)$ - Fixed-size arrays

5. **Mental Model:** Think of a fixed-size magnifying glass sliding over s2, looking for a window that has the exact same character composition as s1.

**Template:**
```python
n1, n2 = len(s1), len(s2)

if n1 > n2:
    return False

s1_counts = [0] * 26
s2_counts = [0] * 26

# Build initial window
for i in range(n1):
    s1_counts[ord(s1[i]) - ord('a')] += 1
    s2_counts[ord(s2[i]) - ord('a')] += 1

if s1_counts == s2_counts:
    return True

# Slide window
for i in range(n1, n2):
    s2_counts[ord(s2[i]) - ord('a')] += 1
    s2_counts[ord(s2[i - n1]) - ord('a')] -= 1
    
    if s1_counts == s2_counts:
        return True

return False
```