# Substring with Concatenation of All Words

**LeetCode 30** | **Difficulty:** Hard | **Category:** Sliding Window, Hash Table

---

## Problem Statement

You are given a string `s` and an array of strings `words`. All the strings in `words` are of the **same length**.

A **concatenated substring** in `s` is a substring that contains all the strings of `words` concatenated in **any order**.

Return the starting indices of all the concatenated substrings in `s`. You can return the answer in **any order**.

---

## Examples

### Example 1:
```
Input: s = "barfoothefoobarman", words = ["foo","bar"]
Output: [0,9]

Explanation:
- Starting at index 0: "barfoo" = "bar" + "foo"
- Starting at index 9: "foobar" = "foo" + "bar"
```

### Example 2:
```
Input: s = "wordgoodgoodgoodbestword", words = ["word","good","best","word"]
Output: []

Explanation:
No valid concatenation exists.
```

### Example 3:
```
Input: s = "barfoofoobarthefoobarman", words = ["bar","foo","the"]
Output: [6,9,12]

Explanation:
- Index 6: "foobarthe" = "foo" + "bar" + "the"
- Index 9: "barthefoo" = "bar" + "the" + "foo"
- Index 12: "thefoobar" = "the" + "foo" + "bar"
```

---

## Strategy

### The "Block-by-Block" Window Approach

If the length of each word is $L$, we don't need to slide our window 1 character at a time in every pass. Instead, we can think of the string as being made of **"blocks"** of length $L$.

**Key Insights:**

1. **All words have the same length** $L$
2. We need to find all permutations of `words` concatenated together
3. Instead of checking every character position, we check every **word-sized block**
4. We need to run $L$ separate sliding window passes (one for each offset)

**The Logic:**

1. **Count the words:** Store the frequency of each word in a `word_count` map.

2. **Iterate over offsets:** Since words have length $L$, there are $L$ possible starting positions (0, 1, ..., $L-1$). We run a sliding window for each offset.

3. **Sliding the Window:**
   - Move the right pointer $L$ characters at a time to grab a new word
   - If the word exists in our target, add it to a `current_count` map
   - If we have too many of that word, or the word is invalid, shrink the window from the left
   - When the window size equals (number of words × $L$), we've found a valid starting index!

### Why the `range(word_len)` loop?

**Visual Example:** Imagine `word_len = 3` and the string is `"abcdefghi"`

```
Pass 0 (offset=0): abc | def | ghi
Pass 1 (offset=1):  bcd | efg | hi
Pass 2 (offset=2):   cde | fgh | i
```

By running these $L$ passes, we **cover every possible concatenation** starting at any character index, while still moving in efficient "blocks" rather than character by character.

**Example with words = ["foo", "bar"]:**

```
s = "barfoothefoobarman"
word_len = 3, so we need 3 passes:

Pass 0 (offset=0): bar|foo|the|foo|bar|man
                   ✓ Found at index 0: "barfoo"
                   
Pass 1 (offset=1):  arf|oot|hef|oob|arm|an
                   (no valid combinations)
                   
Pass 2 (offset=2):   rfo|oth|efo|oba|rma|n
                   ✓ Found at index 9: "foobar"
```

**Complexity:**
- **Time:** $O(N \times L)$ where N = length of s, L = word length
  - $L$ passes, each processes $N/L$ words, each word takes $O(L)$ to extract
- **Space:** $O(K + M)$ where K = number of unique words, M = number of words

---

## Solution

```python
from typing import List
from collections import Counter

class Solution:
    def findSubstring(self, s: str, words: List[str]) -> List[int]:
        if not s or not words:
            return []
        
        word_len = len(words[0])
        num_words = len(words)
        total_len = word_len * num_words
        word_count = Counter(words)
        result = []
        
        # There are word_len possible starting offsets
        for i in range(word_len):
            left = i
            right = i
            current_count = Counter()
            count = 0  # Number of valid words currently in the window
            
            # Slide 'right' in steps of word_len
            while right + word_len <= len(s):
                # Extract the word at the right pointer
                word = s[right : right + word_len]
                right += word_len
                
                # If word is in our target list
                if word in word_count:
                    current_count[word] += 1
                    count += 1
                    
                    # If we have more of 'word' than allowed, shrink from left
                    while current_count[word] > word_count[word]:
                        left_word = s[left : left + word_len]
                        current_count[left_word] -= 1
                        count -= 1
                        left += word_len
                    
                    # If we've matched all words, record the starting index
                    if count == num_words:
                        result.append(left)
                
                else:
                    # Word is not in our list, reset the window
                    current_count.clear()
                    count = 0
                    left = right
        
        return result
```

---

## Detailed Walkthrough

### Example: `s = "barfoothefoobarman"`, `words = ["foo", "bar"]`

**Setup:**
- `word_len = 3`
- `num_words = 2`
- `total_len = 6`
- `word_count = {"foo": 1, "bar": 1}`

**Pass 0 (offset = 0):**

| Step | Right | Word | Current Count | Count | Left | Valid? | Action |
|------|-------|------|---------------|-------|------|--------|--------|
| 1 | 0→3 | "bar" | {"bar": 1} | 1 | 0 | ❌ | Continue |
| 2 | 3→6 | "foo" | {"bar": 1, "foo": 1} | 2 | 0 | ✅ | **Add index 0** |
| 3 | 6→9 | "the" | {} | 0 | 9 | ❌ | Reset (invalid word) |
| 4 | 9→12 | "foo" | {"foo": 1} | 1 | 9 | ❌ | Continue |
| 5 | 12→15 | "bar" | {"foo": 1, "bar": 1} | 2 | 9 | ✅ | **Add index 9** |
| 6 | 15→18 | "man" | {} | 0 | 18 | ❌ | Reset |

**Pass 1 (offset = 1):** `"arfoothefoobarman"`
- Words: "arf", "oot", "hef", "oob", "arm", "an"
- None match our target words

**Pass 2 (offset = 2):** `"rfoothefoobarman"`
- Words: "rfo", "oth", "efo", "oba", "rma", "n"
- None match our target words

**Final Result:** `[0, 9]`

---

## Detailed Example with Duplicate Words

### Example: `s = "wordgoodgoodgoodbestword"`, `words = ["word", "good", "best", "good"]`

**Setup:**
- `word_count = {"word": 1, "good": 2, "best": 1}`

**Pass 0 (offset = 0):**

| Step | Right | Word | Current Count | Count | Valid? | Action |
|------|-------|------|---------------|-------|--------|--------|
| 1 | 0→4 | "word" | {"word": 1} | 1 | ❌ | Need 4 words |
| 2 | 4→8 | "good" | {"word": 1, "good": 1} | 2 | ❌ | Need 4 words |
| 3 | 8→12 | "good" | {"word": 1, "good": 2} | 3 | ❌ | Need 4 words |
| 4 | 12→16 | "good" | {"word": 1, "good": 3} | 4 | ❌ | Too many "good"! |
| - | - | Shrink | {"good": 2} | 3 | ❌ | Remove "word" from left |
| 5 | 16→20 | "best" | {"good": 2, "best": 1} | 3 | ❌ | "word" still removed |
| 6 | 20→24 | "word" | {"good": 2, "best": 1, "word": 1} | 4 | ❌ | Wrong position |

**Result:** No valid concatenation (expected `[]`)

---

## Why This Approach Works

### Comparison with Naive Approach

**Naive Approach (Wrong!):**
```python
# Try every starting position
for i in range(len(s) - total_len + 1):
    substring = s[i : i + total_len]
    # Check if substring is a valid permutation
    # This would be O(N × M × L) where M = num_words
```

**Problem:** Too slow! For each position, you'd need to check all permutations.

**Our Approach (Correct!):**
- Only $L$ passes instead of $N$ positions
- Each pass moves in blocks of size $L$
- Uses hash map for $O(1)$ word lookups
- Tracks word frequency efficiently

---

## Edge Cases

### 1. Empty Input
```python
s = "", words = ["foo"]
# Output: []
```

### 2. Words Longer Than String
```python
s = "foo", words = ["foobar"]
# Output: []
```

### 3. Duplicate Words
```python
s = "wordgoodgoodgoodbestword", words = ["word", "good", "best", "good"]
# Must track frequency correctly
# Output: []
```

### 4. Single Word
```python
s = "barfoo", words = ["bar"]
# Output: [0]
```

### 5. Overlapping Matches
```python
s = "ababab", words = ["ab", "ab"]
# Output: [0, 2]
```

---

## Common Mistakes

### ❌ Mistake 1: Checking Every Character Position
```python
# Wrong! Too slow
for i in range(len(s)):
    # Check if valid concatenation starts here
```

**Fix:** Use $L$ offset passes, moving in word-sized blocks.

### ❌ Mistake 2: Not Resetting Window on Invalid Word
```python
# Wrong! Should reset completely
if word not in word_count:
    left += word_len  # Only moves left slightly
```

**Fix:**
```python
if word not in word_count:
    current_count.clear()
    count = 0
    left = right  # Jump to right position
```

### ❌ Mistake 3: Wrong Frequency Tracking
```python
# Wrong! Doesn't handle duplicates
if word in word_count:
    current_count[word] = 1  # Should increment, not set to 1
```

**Fix:**
```python
if word in word_count:
    current_count[word] += 1  # Increment properly
```

### ❌ Mistake 4: Not Handling Excess Words
```python
# Wrong! Doesn't shrink when too many of one word
if word in word_count:
    current_count[word] += 1
    count += 1
    # Missing: check if current_count[word] > word_count[word]
```

**Fix:** Use while loop to shrink until counts are valid.

---

## Interview Insights

**Question Recognition:**
- "Concatenation of words" + "same length" → Block-by-block sliding window
- "All permutations" + "substring" → Hash map for frequency tracking

**Key Points to Mention:**

1. **Why $L$ passes?**
   - "We need to check all possible starting offsets (0 to L-1) to ensure we don't miss any valid concatenations."

2. **Why reset on invalid word?**
   - "If we encounter a word not in our list, any substring containing it is invalid, so we can reset the entire window."

3. **Time Complexity Explanation:**
   - "We have $L$ passes, each processes roughly $N/L$ words, and extracting each word is $O(L)$, giving us $O(N \times L)$ overall."

**Follow-up Questions:**

1. **Q:** "What if words have different lengths?"
   - **A:** Problem becomes much harder—need to try all permutations or use more complex DP approaches.

2. **Q:** "Can you optimize space?"
   - **A:** Current approach already uses optimal $O(K)$ space for hash maps.

3. **Q:** "What if words can repeat unlimited times?"
   - **A:** Remove the frequency limit check, just ensure all unique words appear.

---

## Related Problems

1. **Minimum Window Substring (LeetCode 76)** - Similar sliding window with frequency tracking
2. **Permutation in String (LeetCode 567)** - Simpler version with single word
3. **Find All Anagrams in a String (LeetCode 438)** - Fixed-size window with frequency
4. **Longest Substring Without Repeating Characters (LeetCode 3)** - Variable window

---

## Summary

**Key Takeaways:**

1. **Algorithm:** Block-by-block sliding window
   - $L$ passes for $L$ different offsets
   - Move in word-sized blocks, not character by character

2. **Data Structures:**
   - `word_count`: Target frequency of each word
   - `current_count`: Current frequency in window
   - `count`: Total number of valid words in window

3. **Optimization Techniques:**
   - Reset window completely on invalid word
   - Shrink window when word frequency exceeds target
   - Record index when `count == num_words`

4. **Complexity:**
   - Time: $O(N \times L)$ - Much better than naive $O(N \times M! \times L)$
   - Space: $O(K + M)$ - Hash maps for word tracking

5. **Mental Model:** Think of the string as divided into word-sized blocks. For each offset, slide a window that "collects" these blocks until you have the right combination or encounter an invalid block.

**Pattern Recognition:**
- Fixed-length substrings → Consider block-based approach
- All permutations → Use hash map frequency counting
- Multiple passes needed → One pass per offset
