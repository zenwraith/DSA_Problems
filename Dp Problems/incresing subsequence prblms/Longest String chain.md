
# **Longest String Chain**

**LeetCode Link:** [https://leetcode.com/problems/longest-string-chain/](https://leetcode.com/problems/longest-string-chain/)

## **The Strategy: Sort by Length**

In **Longest Increasing Subsequence (LIS)**, we sort by value. In **Longest String Chain**, the "value" is the length of the string. A word of length 5 can only be a successor to a word of length 4.

### **Step-by-Step Approach:**

1. **Sort the words by their lengths** - This ensures we process shorter words first.

2. **DP State:** `dp[word]` = the length of the longest chain ending with this word.

3. **The Transition:** For each word, try deleting every possible character one by one to see if the resulting "predecessor" exists in our DP table.

## **Key Insight**

A word can be extended from a predecessor if:
- The predecessor is exactly 1 character shorter
- The predecessor exists in our DP table
- The chain length is updated as: `dp[word] = max(dp[word], dp[predecessor] + 1)`

## **Solution**

```python
class Solution:
    def longestStrChain(self, words: List[str]) -> int:
        # sort by length so that we only look for shorter words
        words.sort(key=len)

        dp = {}
        max_chain = 0

        for word in words:
            # Every word is a chain of at least 1
            dp[word] = 1
            # try removing each character to find a predecessor
            for i in range(len(word)):
                # create a predecessor by skipping the character at index i
                prev = word[:i] + word[i+1:]

                if prev in dp:
                    dp[word] = max(dp[word], dp[prev] + 1)

            max_chain = max(max_chain, dp[word])

        return max_chain

        
```