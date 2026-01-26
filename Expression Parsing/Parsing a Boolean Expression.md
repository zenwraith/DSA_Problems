
leetcode link : https://leetcode.com/problems/parsing-a-boolean-expression/



Implementation: The Stack Approach
Using a stack is usually the cleanest way to handle nested parentheses without getting lost in recursion.

The Strategy:
Push every character onto the stack until you hit a closing parenthesis ).

When you see ), pop everything out until you find the matching opening parenthesis (. These are your operands (the t's and f's).

Pop the character before the (—that is your operator (!, &, or |).

Evaluate that small sub-expression and push the result (t or f) back onto the stack.

````python

class Solution:
    def parseBoolExpr(self, expression: str) -> bool:

        stack = []

        for char in expression:
            if char ==',':
                continue
            
            if char != ')':
                stack.append(char) #everything including ( '!' , '& , or '|'
            else:
                operands = set()

                while stack[-1] != '(':
                    operands.add(stack.pop())
                
                stack.pop() # remove '('
                operator = stack.pop() # get the '!' , '& , or '|'

                if operator == '!':
                    result  = 't' if 'f' in operands else 'f'
                elif operator == '&':
                    result = 'f' if 'f' in operands else 't'
                elif operator == '|':
                    result ='t' if 't' in operands else 'f'

                stack.append(result)
        return stack[0] == 't'

 
````


1. The "Calculator" Pattern (Basic Calculator I, II, III)
   These are the most common. You are given a string like "3 + (2 * 4) - 5".

Key Skill: Handling Operator Precedence (multiplication before addition) and Parentheses.

The "Magic" Trick: Use a Stack.

When you see a + or -, you push the current number to the stack (using the sign).

When you see * or /, you pop the last number, multiply it by the current number, and push it back.

When you see (, you use Recursion to solve the inside as a "mini-problem."


2. The "Decoding" Pattern (Decode String)
LeetCode 394: Given 3[a2[c]], return accaccacc.

This is important because it teaches you how to handle nested multipliers.

The Strategy: Use two stacks—one for numbers (the multipliers) and one for strings (the content being built).

The Logic:When you see a number, build it up (e.g., 1 then 12).

When you see [, push the current number and current string to their stacks and "reset.
"When you see ], pop the number and the previous string, repeat your current string $N$ times, and attach it to the previous string.


3. The "True" Interval DP Parsing (Boolean Parenthesization)

As we discussed earlier, this is the "hard" version where parentheses are missing.

Problem: Given T | F & T, how many ways can you add parentheses so the result is True?

Why it's Interval DP: Because you have to choose where to put the brackets. 

You try splitting at every operator.

The State: dp[i][j][True] = number of ways the range $i$ to $j$ can be True.

The Transition: If the operator at split $k$ is &, then:$TotalTrue = LeftTrue \times RightTrue$.