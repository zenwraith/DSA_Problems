

Leetcode url : https://leetcode.com/problems/longest-valid-parentheses/


````python

class Solution:
    def longestValidParentheses(self, s: str) -> int:

        n = len(s)

        dp = [0]*n
        
        ans = 0

        for i in range(1, n):

            if s[i] == ')':

                if s[i-1] =='(':

                    # solving small parenthesis

                    dp[i] = (dp[i-2] if i-2>=0 else 0) + 2

                elif i - dp[i-1] > 0 and s[i - 1 - dp[i-1]] == '(':

                    # big nested parenthtesis inside 

                    inner_length = dp[i-1]
                    length_before_inner_start = dp[ i - dp[i-1] - 2] if i - dp[i-1] - 2 >=0 else 0

                    dp[i] = inner_length + 2 + length_before_inner_start

                # apart from that all case are invalid

            ans = max(ans , dp[i])

        return ans








````

stack solution 

````python

class Solution:
    def longestValidParentheses(self, s: str) -> int:

        n = len(s)

        stack = [ -1 ]

        ans = 0

        for i, char in enumerate(s):

            if char == '(':
                stack.append(i)
            else:

                stack.pop()

                if not stack:
                    stack.append(i)
                    continue
                else:
                    curr = i- stack[-1]
                    ans = max(ans , curr)

        return ans

````