
# **Longest String Chain**

**LeetCode Link:** [https://leetcode.com/problems/longest-string-chain/](https://leetcode.com/problems/longest-string-chain/)

---

## **Problem Statement**

You are given an array of `words` where each word consists of lowercase English letters.

`word_A` is a **predecessor** of `word_B` if and only if we can insert **exactly one letter** anywhere in `word_A` **without changing the order of the other characters** to make it equal to `word_B`.

A **word chain** is a sequence of words `[word_1, word_2, ..., word_k]` where `k >= 1`, and each `word_i` is a predecessor of `word_{i+1}`.

Return the **length of the longest possible word chain** with words chosen from the given list of `words`.

**Example:**
```
Input: words = ["a","b","ba","bca","bda","bdca"]
Output: 4
Explanation: One of the longest word chains is ["a","ba","bda","bdca"].

Input: words = ["xbc","pcxbcf","xb","cxbc","pcxbc"]
Output: 5
Explanation: All the words can be put in a word chain ["xb", "xbc", "cxbc", "pcxbc", "pcxbcf"].
```

---

## **Strategy: Sort by Length + DP**

In **Longest Increasing Subsequence (LIS)**, we sort by value. In **Longest String Chain**, the "value" is the **length of the string**. A word of length 5 can only be a successor to a word of length 4.

### **Step-by-Step Approach:**

1. **Sort the words by their lengths** - This ensures we process shorter words first.
2. **DP State:** `dp[word]` = the length of the longest chain ending with this word.
3. **The Transition:** For each word, try **deleting every possible character one by one** to see if the resulting "predecessor" exists in our DP table.

### **Key Insight**

A word can be extended from a predecessor if:
- The predecessor is **exactly 1 character shorter**
- The predecessor exists in our DP table
- The chain length is updated as: `dp[word] = max(dp[word], dp[predecessor] + 1)`

---

## **Solution**

```python
class Solution:
    def longestStrChain(self, words: List[str]) -> int:
        # Sort by length so that we process shorter words first
        words.sort(key=len)
        
        dp = {}
        max_chain = 0
        
        for word in words:
            # Every word is a chain of at least 1
            dp[word] = 1
            
            # Try removing each character to find a predecessor
            for i in range(len(word)):
                # Create a predecessor by skipping the character at index i
                prev = word[:i] + word[i+1:]
                
                if prev in dp:
                    dp[word] = max(dp[word], dp[prev] + 1)
            
            max_chain = max(max_chain, dp[word])
        
        return max_chain
```

---

## **Example Walkthrough**

**Input:** `words = ["a", "b", "ba", "bca", "bda", "bdca"]`

### **Step 1: Sort by Length**
```
words = ["a", "b", "ba", "bca", "bda", "bdca"]
# Already sorted by length!
```

### **Step 2: DP Computation**

| Word | Try Removing | Predecessor | In DP? | dp[word] |
|------|--------------|-------------|--------|----------|
| "a" | - | - | - | 1 |
| "b" | - | - | - | 1 |
| "ba" | Remove 'b' | "a" | ✓ | max(1, dp["a"]+1) = 2 |
|      | Remove 'a' | "b" | ✓ | max(2, dp["b"]+1) = 2 |
| "bca" | Remove 'b' | "ca" | ✗ | 1 |
|       | Remove 'c' | "ba" | ✓ | max(1, dp["ba"]+1) = 3 |
|       | Remove 'a' | "bc" | ✗ | 3 |
| "bda" | Remove 'b' | "da" | ✗ | 1 |
|       | Remove 'd' | "ba" | ✓ | max(1, dp["ba"]+1) = 3 |
|       | Remove 'a' | "bd" | ✗ | 3 |
| "bdca" | Remove 'b' | "dca" | ✗ | 1 |
|        | Remove 'd' | "bca" | ✓ | max(1, dp["bca"]+1) = 4 |
|        | Remove 'c' | "bda" | ✓ | max(4, dp["bda"]+1) = 4 |
|        | Remove 'a' | "bdc" | ✗ | 4 |

### **Final DP State**
```
dp = {
    "a": 1,
    "b": 1,
    "ba": 2,
    "bca": 3,
    "bda": 3,
    "bdca": 4
}

max_chain = 4
```

**Longest Chain:** `["a", "ba", "bda", "bdca"]` or `["a", "ba", "bca", "bdca"]`

---

## **Why Sorting by Length is Crucial**

**Without sorting:**
```
words = ["bdca", "a", "ba", "bda"]

Process "bdca" first:
  Try removing chars to find predecessors like "bda", "bca", etc.
  But these aren't in dp yet! ❌
```

**With sorting:**
```
words = ["a", "ba", "bda", "bdca"]

Process "a" first: dp["a"] = 1
Process "ba": finds "a" ✓
Process "bda": finds "ba" ✓
Process "bdca": finds "bda" ✓
```

By processing shorter words first, we guarantee that all potential predecessors have already been computed.

---

## **Why Try All Deletions?**

**Question:** Why not just check if any existing word is a predecessor?

**Answer:** Because we need to check if the current word can **extend** any previous word, not the other way around.

**Example:**
```
Current word: "abc"

Need to check if "abc" extends:
  "ab" (delete 'c')
  "ac" (delete 'b')
  "bc" (delete 'a')

Checking from existing words would be inefficient:
  For each existing word w, check if we can insert a char to make "abc"
  This requires trying 26 letters at each position!
```

**Our approach:**
```python
for i in range(len(word)):
    prev = word[:i] + word[i+1:]  # O(n) to create string
    if prev in dp:                 # O(1) hash lookup
        dp[word] = max(dp[word], dp[prev] + 1)
```

Only $O(n)$ deletions to try, each with $O(1)$ lookup!

---

## **Comparison with Standard LIS**

| Aspect | Standard LIS | Longest String Chain |
|--------|--------------|----------------------|
| **Sort By** | Value (numbers) | Length (strings) |
| **Condition** | `nums[i] > nums[j]` | Remove 1 char from word[i] gives word[j] |
| **DP Type** | Array | HashMap (words as keys) |
| **Lookup** | Index-based | Hash-based |
| **Complexity** | O(n²) or O(n log n) | O(n × L) where L is max word length |

---

## **Edge Cases**

### **1. All Same Length**
```
Input: ["abc", "def", "ghi"]
Output: 1
No word can extend another (all same length)
```

### **2. No Valid Chains**
```
Input: ["a", "cde"]
Output: 1
Gap is too large (length 1 → length 3)
```

### **3. Multiple Chains**
```
Input: ["a", "ab", "c", "cd"]
Output: 2
Two chains: ["a", "ab"] and ["c", "cd"]
```

### **4. One Long Chain**
```
Input: ["a", "ab", "abc", "abcd"]
Output: 4
Chain: ["a", "ab", "abc", "abcd"]
```

---

## **Complexity Analysis**

**Time Complexity:** $O(n \times L^2)$  
Where $n$ is the number of words and $L$ is the maximum length of a word.
- Sorting: $O(n \log n)$
- For each word (n words):
  - Try L deletions
  - Each deletion creates a string of length L: $O(L)$
  - Hash lookup: $O(1)$ average
  - **Per word:** $O(L \times L) = O(L^2)$
- **Total:** $O(n \times L^2)$

**Space Complexity:** $O(n \times L)$  
- DP hashmap stores n words, each of average length L.