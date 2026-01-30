# Text Justification

**LeetCode 68** | **Difficulty:** Hard | **Category:** Array, String, Simulation

---

## Problem Statement

Given an array of strings `words` and a width `maxWidth`, format the text such that each line has exactly `maxWidth` characters and is fully (left and right) justified.

You should pack your words in a greedy approach; that is, pack as many words as you can in each line. Pad extra spaces `' '` when necessary so that each line has exactly `maxWidth` characters.

Extra spaces between words should be distributed as evenly as possible. If the number of spaces on a line does not divide evenly between words, the empty slots on the left will be assigned more spaces than the slots on the right.

For the last line of text, it should be **left-justified**, and no extra space is inserted between words.

**Note:**
- A word is defined as a character sequence consisting of non-space characters only.
- Each word's length is guaranteed to be greater than 0 and not exceed `maxWidth`.
- The input array `words` contains at least one word.

---

## Examples

### Example 1:
```
Input: words = ["This", "is", "an", "example", "of", "text", "justification."], maxWidth = 16
Output:
[
   "This    is    an",
   "example  of text",
   "justification.  "
]
```

### Example 2:
```
Input: words = ["What","must","be","acknowledgment","shall","be"], maxWidth = 16
Output:
[
  "What   must   be",
  "acknowledgment  ",
  "shall be        "
]

Explanation:
Note that the last line is "shall be    " instead of "shall     be",
because the last line must be left-justified instead of fully-justified.
Note that the second line is also left-justified because it contains only one word.
```

### Example 3:
```
Input: words = ["Science","is","what","we","understand","well","enough","to","explain","to","a","computer.","Art","is","everything","else","we","do"], maxWidth = 20
Output:
[
  "Science  is  what we",
  "understand      well",
  "enough to explain to",
  "a  computer.  Art is",
  "everything  else  we",
  "do                  "
]
```

---

## Strategy

### Greedy Line Building with Space Distribution

**Key Insight:** Pack as many words as possible into each line, then distribute extra spaces evenly with left-bias.

**Why This Works:**

```
Challenge: Three distinct cases
1. Regular lines: Fully justify (distribute spaces evenly)
2. Single-word lines: Left-justify (pad right with spaces)
3. Last line: Left-justify (single space between words, pad right)

Approach: Build lines greedily, then format based on type

Step 1: Determine which words fit in current line
  - Words + minimum spaces (one between each) ≤ maxWidth
  - Count: len(words) + number_of_gaps ≤ maxWidth
  - Where: number_of_gaps = len(line) - 1 + 1 (for next word)

Step 2: Distribute extra spaces
  - Total spaces needed = maxWidth - sum(word_lengths)
  - Gaps between words = len(line) - 1
  - Base spaces per gap = total_spaces // gaps
  - Extra spaces = total_spaces % gaps
  - Distribute extra spaces left-to-right

Step 3: Handle edge cases
  - Single word: All spaces go to the right
  - Last line: Single space between words, rest on right
```

**Algorithm Steps:**

1. **Build line:** Greedily add words while they fit
   ```python
   # Check if word fits: current_length + len(line) + len(word) ≤ maxWidth
   # len(line) accounts for spaces between words
   ```

2. **Justify regular line:**
   ```python
   extra_space = maxWidth - sum_of_word_lengths
   gaps = max(1, len(line) - 1)  # At least 1 to avoid division by zero
   spaces_per_gap = extra_space // gaps
   extra_spaces = extra_space % gaps
   
   # Add spaces_per_gap to each word, plus 1 extra for first 'extra_spaces' words
   ```

3. **Justify last line:**
   ```python
   Join words with single space
   Pad right with remaining spaces
   ```

**Complexity:**
- **Time:** $O(n)$ where n = total characters in all words
- **Space:** $O(n)$ for result (required for output)

---

## Solution

```python
class Solution:
    def fullJustify(self, words: List[str], maxWidth: int) -> List[str]:
        """
        Format text with full justification.
        
        Strategy: Greedy line packing + space distribution
        1. Pack as many words as possible per line
        2. For regular lines: distribute spaces evenly (left-biased)
        3. For last line: left-justify with single spaces
        
        Edge cases:
        - Single word per line: all spaces go right
        - Last line: left-justified, not fully justified
        
        Args:
            words: List of words to format
            maxWidth: Maximum width of each line
            
        Returns:
            List of formatted lines
        """
        line = []        # Current line's words
        length = 0       # Sum of word lengths in current line
        res = []         # Result lines
        i = 0
        n = len(words)
        
        while i < n:
            # Check if current word fits in line
            # Need space for: words + spaces between them + new word
            # len(line) accounts for minimum single spaces between words
            if length + len(line) + len(words[i]) > maxWidth:
                # Current line is full, justify it
                
                # Calculate extra spaces to distribute
                extra_space = maxWidth - length
                
                # Number of gaps (at least 1 to avoid division by zero)
                gaps = max(1, len(line) - 1)
                
                # Spaces per gap and remainder
                spaces = extra_space // gaps
                remainder = extra_space % gaps
                
                # Distribute spaces
                for j in range(gaps):
                    line[j] += ' ' * spaces
                    # Left-bias: first 'remainder' gaps get extra space
                    if remainder > 0:
                        line[j] += ' '
                        remainder -= 1
                
                # Add justified line to result
                res.append(''.join(line))
                
                # Reset for next line
                line = []
                length = 0
            
            # Add word to current line
            line.append(words[i])
            length += len(words[i])
            i += 1
        
        # Handle last line (left-justified)
        last_line = ' '.join(line)
        trailing_spaces = maxWidth - len(last_line)
        res.append(last_line + ' ' * trailing_spaces)
        
        return res
```

---

## Detailed Walkthrough

### Example 1: `words = ["This", "is", "an", "example", "of", "text", "justification."], maxWidth = 16`

**Line 1: Building**

| i | word | length | len(line) | Check | Action |
|---|------|--------|-----------|-------|--------|
| 0 | "This" | 4 | 0 | 4+0+4=8 ≤ 16 ✓ | Add to line |
| 1 | "is" | 2 | 1 | 6+1+2=9 ≤ 16 ✓ | Add to line |
| 2 | "an" | 2 | 2 | 8+2+2=12 ≤ 16 ✓ | Add to line |
| 3 | "example" | 7 | 3 | 10+3+7=20 > 16 ✗ | Line full! |

**Line 1: Justifying ["This", "is", "an"]**

```
length = 4 + 2 + 2 = 8
extra_space = 16 - 8 = 8
gaps = max(1, 3 - 1) = 2
spaces = 8 // 2 = 4
remainder = 8 % 2 = 0

Distribution:
  line[0] = "This" + "    " (4 spaces)
  line[1] = "is" + "    " (4 spaces)
  line[2] = "an" (last word, no spaces added in loop)

Result: "This    is    an" (length 16) ✓
```

**Line 2: Building and Justifying**

```
Words: ["example", "of", "text"]
length = 7 + 2 + 4 = 13
extra_space = 16 - 13 = 3
gaps = 2
spaces = 3 // 2 = 1
remainder = 3 % 2 = 1

Distribution:
  line[0] = "example" + " " + " " (1 base + 1 extra)
  line[1] = "of" + " " (1 base)
  line[2] = "text"

Result: "example  of text" (length 16) ✓
```

**Line 3: Last Line ["justification."]**

```
Only one word: "justification."
Length = 15
Trailing spaces = 16 - 15 = 1

Result: "justification. " (length 16) ✓
```

**Final Output:**
```
[
   "This    is    an",
   "example  of text",
   "justification.  "
]
```

---

### Example 2: Single Word Lines

**Input:** `words = ["What","must","be","acknowledgment","shall","be"], maxWidth = 16`

**Line 1: ["What", "must", "be"]**

```
length = 4 + 4 + 2 = 10
extra_space = 16 - 10 = 6
gaps = 2
spaces = 6 // 2 = 3
remainder = 0

Result: "What   must   be"
```

**Line 2: ["acknowledgment"]**

```
Single word! length = 14
extra_space = 16 - 14 = 2
gaps = max(1, 1 - 1) = max(1, 0) = 1  (Important!)

spaces = 2 // 1 = 2
remainder = 0

line[0] = "acknowledgment" + "  "

Result: "acknowledgment  "
```

**Line 3: Last line ["shall", "be"]**

```
Left-justified: "shall be" + "        "
Result: "shall be        "
```

---

## Why This Algorithm Works

### The Space Distribution Formula

**Goal:** Distribute N spaces across G gaps as evenly as possible with left-bias.

```
Example: 11 spaces, 3 gaps

Even distribution: 11 / 3 = 3.67
Not possible! Need integers.

Solution:
  base = 11 // 3 = 3
  extra = 11 % 3 = 2
  
  Gap 1: 3 + 1 = 4 spaces (gets extra)
  Gap 2: 3 + 1 = 4 spaces (gets extra)
  Gap 3: 3 + 0 = 3 spaces (no extra)
  
  Total: 4 + 4 + 3 = 11 ✓
```

**Left-bias ensures:** When spaces don't divide evenly, left gaps get more.

### Edge Case: Single Word Line

```
Word: "acknowledgment" (14 chars), maxWidth = 16

Problem: No gaps between words (len(line) - 1 = 0)
Division by zero!

Solution: gaps = max(1, len(line) - 1)
  gaps = max(1, 0) = 1
  All 2 extra spaces go to the right

Result: "acknowledgment  "
```

### Last Line Special Handling

```
Regular line: Distribute spaces evenly
Last line: Single space between words, rest on right

Why? Problem specification requires left-justification for last line.

Implementation:
  ' '.join(line)  # Single spaces
  + ' ' * trailing_spaces  # Pad right
```

---

## Edge Cases

### 1. Single Word
```python
words = ["justified"], maxWidth = 10
# Output: ["justified "]
# All spaces go to the right
```

### 2. Very Long Word
```python
words = ["a"], maxWidth = 5
# Output: ["a    "]
# Pad with 4 spaces
```

### 3. Exact Fit
```python
words = ["hello", "world"], maxWidth = 11
# "hello world" = 11 chars exactly
# Output: ["hello world"]
# Single line, left-justified (last line)
```

### 4. Two Words, Many Spaces
```python
words = ["a", "b"], maxWidth = 10
# Line not full, becomes last line
# Output: ["a b       "]
# Left-justified with trailing spaces
```

### 5. Multiple Single-Word Lines
```python
words = ["abcdefghij", "klmnopqrst"], maxWidth = 10
# Output: ["abcdefghij", "klmnopqrst"]
# Each word fills line exactly
```

### 6. Last Line With Multiple Words
```python
words = ["a","b","c"], maxWidth = 5
# All fit in one line, treated as last line
# Output: ["a b c"]
# Left-justified, not fully justified
```

---

## Common Mistakes

### ❌ Mistake 1: Forgetting max(1, ...) for Gaps
```python
# Wrong! Division by zero for single word
gaps = len(line) - 1
spaces = extra_space // gaps  # Crashes!
```

**Fix:**
```python
gaps = max(1, len(line) - 1)  # At least 1
```

### ❌ Mistake 2: Wrong Line Capacity Check
```python
# Wrong! Doesn't account for spaces between words
if length + len(words[i]) > maxWidth:
```

**Fix:**
```python
if length + len(line) + len(words[i]) > maxWidth:
    # len(line) = number of spaces needed between words
```

### ❌ Mistake 3: Applying Full Justification to Last Line
```python
# Wrong! Last line should be left-justified
# Don't justify last line with space distribution
```

**Fix:**
```python
# Handle last line separately after main loop
last_line = ' '.join(line)  # Single spaces only
last_line += ' ' * (maxWidth - len(last_line))  # Pad right
```

### ❌ Mistake 4: Not Distributing Extra Spaces Left-to-Right
```python
# Wrong! All extra spaces go to last gap
for j in range(gaps):
    line[j] += ' ' * spaces
line[-1] += ' ' * remainder  # Wrong position!
```

**Fix:**
```python
for j in range(gaps):
    line[j] += ' ' * spaces
    if remainder > 0:  # Distribute from left
        line[j] += ' '
        remainder -= 1
```

### ❌ Mistake 5: Modifying Last Word in Line
```python
# Wrong! Adds spaces to last word
for j in range(len(line)):  # Should be gaps, not len(line)
    line[j] += spaces
```

**Fix:**
```python
for j in range(gaps):  # gaps = len(line) - 1
    line[j] += spaces
    # Last word (line[-1]) is never modified
```

---

## Interview Insights

**Question Recognition:**
- "Text formatting" + "justify" + "distribute spaces" → Simulation problem
- "Greedy packing" indicates line-by-line processing

**Key Points to Mention:**

1. **Three cases:**
   - "There are three distinct cases: regular lines with full justification, single-word lines that pad right, and the last line which is left-justified."

2. **Space distribution:**
   - "Extra spaces are distributed evenly across gaps with left-bias. If I have 11 spaces and 3 gaps, I use 11//3=3 per gap, with 11%3=2 extra going to the leftmost gaps."

3. **Edge case - single word:**
   - "For single-word lines, there are no gaps, so I use max(1, len(line)-1) to avoid division by zero and put all spaces on the right."

4. **Greedy packing:**
   - "I pack as many words as possible into each line by checking if length + spaces_needed + next_word_length fits."

**Follow-up Questions:**

1. **Q:** "What if maxWidth is very large?"
   - **A:** All words might fit in one line, which becomes the last line (left-justified).

2. **Q:** "What if a word is exactly maxWidth?"
   - **A:** It forms a line by itself with no trailing spaces needed.

3. **Q:** "How would you handle different justification types (left, right, center)?"
   - **A:** Abstract the justification logic into separate functions based on type.

4. **Q:** "What about hyphenation?"
   - **A:** Much more complex. Would need dictionary of valid hyphenation points and try to fit partial words.

5. **Q:** "Can you make it O(1) space?"
   - **A:** Not really, since we need to return the result which is O(n). Current solution is optimal.

**Complexity Analysis:**
- Time: O(n) where n = total characters
- Space: O(n) for the result (required by output)

---

## Related Problems

1. **Zigzag Conversion (LeetCode 6)** - String formatting pattern
2. **Reverse Words in a String (LeetCode 151)** - String manipulation
3. **String Compression (LeetCode 443)** - String processing
4. **Valid Word Abbreviation (LeetCode 408)** - String validation
5. **Text Wrap (Python textwrap module)** - Similar problem in practice

---

## Variations

### Variation 1: Left Justification Only
```python
def leftJustify(self, words, maxWidth):
    """All lines left-justified (simpler)."""
    res = []
    line = []
    length = 0
    
    for word in words:
        if length + len(line) + len(word) > maxWidth:
            # Line full, format it
            line_str = ' '.join(line)
            line_str += ' ' * (maxWidth - len(line_str))
            res.append(line_str)
            line = []
            length = 0
        
        line.append(word)
        length += len(word)
    
    # Last line
    if line:
        line_str = ' '.join(line)
        line_str += ' ' * (maxWidth - len(line_str))
        res.append(line_str)
    
    return res
```

### Variation 2: Right Justification
```python
def rightJustify(self, words, maxWidth):
    """All lines right-justified."""
    res = []
    line = []
    length = 0
    
    for word in words:
        if length + len(line) + len(word) > maxWidth:
            # Right-justify: spaces on left
            line_str = ' '.join(line)
            leading_spaces = maxWidth - len(line_str)
            res.append(' ' * leading_spaces + line_str)
            line = []
            length = 0
        
        line.append(word)
        length += len(word)
    
    # Last line
    if line:
        line_str = ' '.join(line)
        leading_spaces = maxWidth - len(line_str)
        res.append(' ' * leading_spaces + line_str)
    
    return res
```

### Variation 3: Center Justification
```python
def centerJustify(self, words, maxWidth):
    """All lines center-justified."""
    res = []
    line = []
    length = 0
    
    for word in words:
        if length + len(line) + len(word) > maxWidth:
            # Center: distribute spaces evenly on both sides
            line_str = ' '.join(line)
            total_spaces = maxWidth - len(line_str)
            left_spaces = total_spaces // 2
            right_spaces = total_spaces - left_spaces
            res.append(' ' * left_spaces + line_str + ' ' * right_spaces)
            line = []
            length = 0
        
        line.append(word)
        length += len(word)
    
    # Last line
    if line:
        line_str = ' '.join(line)
        total_spaces = maxWidth - len(line_str)
        left_spaces = total_spaces // 2
        right_spaces = total_spaces - left_spaces
        res.append(' ' * left_spaces + line_str + ' ' * right_spaces)
    
    return res
```

### Variation 4: With Line Numbers
```python
def fullJustifyWithLineNumbers(self, words, maxWidth):
    """Add line numbers (reduces effective width)."""
    lines = self.fullJustify(words, maxWidth - 4)  # Reserve "1. " space
    
    return [f"{i+1}. {line[3:]}" if len(line) > 3 else f"{i+1}. "
            for i, line in enumerate(lines)]
```

---

## Summary

**Key Takeaways:**

1. **Three Distinct Cases:**
   - Regular lines: Full justification with even space distribution
   - Single-word lines: All spaces on right (special case of full)
   - Last line: Left justification with single spaces between words

2. **Greedy Line Building:**
   - Check: `length + len(line) + len(word) <= maxWidth`
   - len(line) accounts for minimum spaces between words
   - Pack as many words as possible per line

3. **Space Distribution Formula:**
   - `base = extra_space // gaps`
   - `remainder = extra_space % gaps`
   - First `remainder` gaps get one extra space (left-bias)

4. **Edge Case Handling:**
   - Single word: `gaps = max(1, len(line) - 1)` avoids division by zero
   - Last line: Handled separately, always left-justified
   - Exact fit: Treated as last line if no more words

5. **Implementation Details:**
   - Don't modify last word in line (only add spaces to first N-1 words)
   - Reset line and length after justifying
   - Track word lengths separately from spaces

**Template:**
```python
def fullJustify(self, words, maxWidth):
    res = []
    line = []
    length = 0
    
    for word in words:
        # Check if word fits
        if length + len(line) + len(word) > maxWidth:
            # Justify current line
            extra_space = maxWidth - length
            gaps = max(1, len(line) - 1)
            spaces = extra_space // gaps
            remainder = extra_space % gaps
            
            for j in range(gaps):
                line[j] += ' ' * spaces
                if remainder > 0:
                    line[j] += ' '
                    remainder -= 1
            
            res.append(''.join(line))
            line = []
            length = 0
        
        line.append(word)
        length += len(word)
    
    # Last line (left-justified)
    last_line = ' '.join(line)
    res.append(last_line + ' ' * (maxWidth - len(last_line)))
    
    return res
```

**Mental Model:**
- Think of it as a word processor's justify feature
- Build lines greedily (maximize words per line)
- Distribute "slack" (extra spaces) evenly
- Special handling for edge cases (single word, last line)
- This is a classic simulation problem requiring careful case handling!
