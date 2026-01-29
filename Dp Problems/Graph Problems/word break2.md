# **Word Break II**

**LeetCode Link:** [https://leetcode.com/problems/word-break-ii/](https://leetcode.com/problems/word-break-ii/)

---

## **Problem Statement**

Given a string `s` and a dictionary of strings `wordDict`, add spaces in `s` to construct a sentence where each word is a valid dictionary word. Return **all such possible sentences** in any order.

**Note** that the same word in the dictionary may be reused multiple times in the segmentation.

**Example:**
```
Input: s = "catsanddog", wordDict = ["cat","cats","and","sand","dog"]
Output: ["cats and dog","cat sand dog"]

Input: s = "pineapplepenapple", wordDict = ["apple","pen","applepen","pine","pineapple"]
Output: ["pine apple pen apple","pineapple pen apple","pine applepen apple"]

Input: s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
Output: []
```

---

## **Strategy: Backtracking with Memoization**

### **Why Not Simple Backtracking?**

A naive backtracking approach would explore every possible prefix at every position, leading to exponential time complexity without any optimization.

### **Key Optimization: Memoization**

Instead of recomputing the valid sentences for a substring multiple times, we **cache the results**. If we've already computed all valid sentences for substring `s[i:]`, we simply return the cached result.

---

## **Solution 1: Backtracking with Global Result**

```python 

class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> List[str]:


        if not s:
            return []

        self.result = []

        self.dp(s, wordDict, [])

        return self.result 


    def dp(self, s, wordDict, path):

        if len(s) == 0:
            self.result.append(' '.join(path))
            return
        
        for word in wordDict:

            if s[:len(word)] == word:

                path.append(word)
                self.dp(s[len(word):], wordDict, path)
                path.pop()
```


## **Solution 2: DFS with Memoization (Optimized)**

```python

class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> List[str]:

        word_set = set(wordDict)
        memo = {}

        def dfs(current_s):
            if current_s in memo:
                return memo[current_s]
            if not current_s:
                return [""] # base case: success !
            
            res = []

            for i in range(1, len(current_s) + 1):

                prefix = current_s[:i]
                if prefix in word_set:
                    suffixes = dfs(current_s[i:])
                    for sub in suffixes:

                        res.append( prefix + (" "+ sub if sub else ""))
            memo[current_s] = res
            return res

        return dfs(s)
```

---

## **Example Walkthrough**

**Input:** `s = "catsanddog"`, `wordDict = ["cat", "cats", "and", "sand", "dog"]`

### **DFS Execution Tree**

```
dfs("catsanddog")
├─ prefix="cat" ✓
│  └─ dfs("sanddog")
│     └─ prefix="sand" ✓
│        └─ dfs("dog")
│           └─ prefix="dog" ✓
│              └─ dfs("") → return [""]
│              → return ["dog"]
│        → return ["sand dog"]
│  → return ["cat sand dog"]
│
└─ prefix="cats" ✓
   └─ dfs("anddog")
      └─ prefix="and" ✓
         └─ dfs("dog")
            └─ prefix="dog" ✓ (memoized!)
               → return ["dog"]
         → return ["and dog"]
   → return ["cats and dog"]

Final: ["cat sand dog", "cats and dog"]
```

### **Memoization Effect**

```
memo = {
    "": [""],
    "dog": ["dog"],
    "anddog": ["and dog"],
    "sanddog": ["sand dog"],
    "catsanddog": ["cat sand dog", "cats and dog"]
}
```

When we reach `dfs("dog")` the second time, we immediately return `["dog"]` without recomputation!

---

## **Complexity Analysis**

### **Approach 1: Backtracking Without Memoization**

**Time Complexity:** $O(2^n)$  
In the worst case, we explore every possible split of the string.

**Space Complexity:** $O(n)$  
Recursion stack depth.

### **Approach 2: DFS with Memoization**

**Time Complexity:** $O(n^3)$  
- There are $O(n)$ unique substrings
- For each substring, we try $O(n)$ prefixes
- Each prefix check takes $O(n)$ time (string slicing and comparison)
- **Total:** $O(n \times n \times n) = O(n^3)$

**Space Complexity:** $O(n^2)$  
- Memoization cache stores results for $O(n)$ substrings
- Each result can contain up to $O(n)$ sentences
- Recursion stack: $O(n)$
