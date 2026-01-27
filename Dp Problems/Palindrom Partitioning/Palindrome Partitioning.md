# **Palindrome Partitioning I**

**LeetCode Link:** [https://leetcode.com/problems/palindrome-partitioning/](https://leetcode.com/problems/palindrome-partitioning/)
## **Naive Approach - Direct Checking**

```python
class Solution:
    def partition(self, s: str) -> List[List[str]]:

        n = len(s)
        self.ans = []

        def dfs(index, temp):
            if index == len(s):
                self.ans.append(temp)
                return 

            for i in range(index, n):
                if s[index : i+1] == s[index:i+1][::-1]:
                    temp = temp + [s[index:i+1]]
                    dfs(i+1, temp)
                    temp = temp[:-1]

        dfs(0, [])
        return self.ans
```

## **The Mindset Shift**

When a problem asks for "all possible" or "every way," DP usually isn't enough. DP is great at telling you a "Best" number, but it's bad at listing combinations. For this, we use Backtracking (DFS).

However, we still use the Interval DP trick we just learned to speed things up!

## **The Strategy**

We will walk through the string from left to right:

1. Pick a prefix.
2. Check: Is this prefix a palindrome?
3. If yes: "Cut" it and recurse on the remaining string.
4. If no: Don't bother; that path is a dead end.

## **Optimized Approach - Interval DP + Backtracking**

```python

class Solution:
    def partition(self, s: str) -> List[List[str]]:

        n = len(s)

        res = []

        isPal = [ [False]* n for i in range(n)]

        # --- Step 1: Pre-calculate Palindromes (Interval DP) ---
        for length in range(1,n+1):
            for i in range(n-length+1):

                j = i+ length -1

                if s[i] == s[j]:
                    if length <=2 or isPal[i+1][j-1]:
                        isPal[i][j] = True
        
        # --- Step 2: Backtracking (DFS) ---
        def backtrack(start, current__path):
            # Base Case: We reached the end of the string
            if start == n:
                res.append(list(current__path))
                return
            
            # Try every possible end position for the next cut
            for end in range(start, n):
                # Check the DP table: is s[start...end] a palindrome?
                if isPal[start][end]:
                    # Choose
                    current__path.append(s[start:end+1])
                    # Explore the rest of the string
                    backtrack(end+1, current__path)
                    # Un-choose (backtrack)
                    current__path.pop()

        backtrack(0,[])
        return res
```

## **Complexity Analysis**
Backtracking is $O(2^N)$ because a string of length $N$ has $N-1$ possible "cut slots," and each slot can either have a cut or not.

## **Why use the DP table here?**

In Palindrome Partitioning I, you could skip the DP table and just use `s[start:end+1] == s[start:end+1][::-1]`. But using the table is "cleaner" and faster if the string is long because you don't keep reversing strings.



