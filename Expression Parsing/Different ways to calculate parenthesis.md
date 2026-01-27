
leetcode link : https://leetcode.com/problems/different-ways-to-add-parentheses/

```python

class Solution:
    def diffWaysToCompute(self, expression: str) -> List[int]:


        memo = {}

        def solve(expr):

            if expr in memo:
                return memo[expr]
            
            res = []

            for i,char in enumerate(expr):
                if char in '+-*':
                    # Split into two sub-problems
                    left_results = solve(expr[:i])
                    right_results = solve(expr[i+1:])

                    # 3. Combine results from both sides
                    for l in left_results:
                        for r in right_results:
                            if char == '+':
                                res.append(l + r)
                            elif char == '-':
                                res.append(l-r)
                            elif char == '*':
                                res.append(l*r)

            if not res:
                res.append( int(expr))
        
            memo[expr] = res
            return res

        return solve(expression)


```