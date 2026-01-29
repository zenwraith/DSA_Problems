
leetcode link : https://leetcode.com/problems/word-break-ii/

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