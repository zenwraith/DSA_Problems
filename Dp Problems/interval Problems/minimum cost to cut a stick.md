
# **Minimum Cost to Cut a Stick**

**LeetCode Link:** [https://leetcode.com/problems/minimum-cost-to-cut-a-stick/](https://leetcode.com/problems/minimum-cost-to-cut-a-stick/)



````python
class Solution:
    def minCost(self, n: int, cuts: List[int]) -> int:

        cuts.sort()
        cuts = [0] + cuts + [n]

        memo = {}

        def solve(i,j):
            # i,j will be the elements of array cuts 

            if j-i <=1:
                return 0
            if (i,j) in memo:
                return memo[(i,j)]
            
            current_stick_length = cuts[j] - cuts[i]
            res = float('inf')

            for k in range(i+1,j):

                # cutting at index k 

                cost = current_stick_length + solve(i, k) + solve(k,j)

                res = min(res , cost)

            memo[(i,j)] = res
            return res

        return solve(0 , len(cuts) - 1)


````


## **Bottom Up Approach**

### **1. The "Gap" Strategy**

Imagine you have a stick with cut points at indices 0, 1, 2, 3, 4.

- **Gap of 1:** Intervals like (0,1), (1,2), (2,3). No cut points between these, so cost is 0.
- **Gap of 2:** Intervals like (0,2), (1,3), (2,4). Exactly one cut point in the middle.
- **Gap of 3:** Intervals like (0,3), (1,4). Two possible cut points to choose from.

We must solve all "Gap 2" problems before we can even touch "Gap 3" problems.

### **2. The Three Nested Loops**

# M is the number of cut points (including 0 and n)
for gap in range(2, m):               # LOOP 1: How big is the piece?
    for i in range(m - gap):          # LOOP 2: Where does the piece start?
        j = i + gap                    # (j is where the piece ends)
        
        # We are now looking at the interval from cuts[i] to cuts[j]
        res = float('inf')
        
        for k in range(i + 1, j):      # LOOP 3: Where do we make the split?
            # Try splitting at every possible point 'k' inside the interval
            res = min(res, dp[i][k] + dp[k][j])
            
        # The cost to cut the stick [i...j] is:
        # (Cheapest split found) + (Current length of the stick)
        dp[i][j] = res + (cuts[j] - cuts[i])
```

### **3. Visualizing the Loops**
Imagine a 2D matrix where i is the row and j is the column.

- **Loop 1 (Gap):** Controls which "diagonal" we are filling. We start near the center diagonal (small gaps) and move toward the top-right corner (the full stick).
- **Loop 2 (Start/i):** Slides our "window" across the stick. If the gap is 2, it looks at [0,2], then [1,3], then [2,4].
- **Loop 3 (Split/k):** This is the "Search." For a fixed window [i, j], it tries every k in the middle to find the best place to break it.

## **Feature Comparison**
Logic,"""Break this big piece into two.""","""Take these two small pieces and join them."""
Loop 1,None (Recursion),gap (The size of the puzzle piece)
Loop 2,i (Start of range),i (Start of range)
Loop 3,k (The split point),k (The split point)




To visualize the Bottom-Up Interval DP table, let’s use a concrete example:Stick length $n=7$ with cuts at $[1, 3, 4, 5]$.
Our boundaries and cuts are: $C = [0, 1, 3, 4, 5, 7]$.The DP table $dp[i][j]$ stores the minimum cost to cut the stick between index $i$ and index $j$ of our array $C$.1. 

The Empty Table ($6 \times 6$)Initially, the table is mostly empty. The diagonal (where $i=j$ or $j = i+1$) is always 0 because you can't cut a stick that has no cut points in the middle.

i∖j,0 (0),1 (1),2 (3),3 (4),4 (5),5 (7)
0 (0),0,0,.,.,.,.
1 (1),-,0,0,.,.,.
2 (3),-,-,0,0,.,.
3 (4),-,-,-,0,0,.
4 (5),-,-,-,-,0,0
5 (7),-,-,-,-,-,0


2. Filling Gap = 2 (The first diagonal)We look at intervals that have one cut point in the middle.

$dp[0][2]$: Stick $[0,3]$. Cut at index 1 ($1$). Cost = $length(3-0) + 0 + 0 = \mathbf{3}$.
$dp[1][3]$: Stick $[1,4]$. Cut at index 2 ($3$). Cost = $length(4-1) + 0 + 0 = \mathbf{3}$.
$dp[2][4]$: Stick $[3,5]$. Cut at index 3 ($4$). Cost = $length(5-3) + 0 + 0 = \mathbf{2}$.
$dp[3][5]$: Stick $[4,7]$. Cut at index 4 ($5$). Cost = $length(7-4) + 0 + 0 = \mathbf{3}$.


1. The Empty Table ($6 \times 6$)Initially, the table is mostly empty. The diagonal (where $i=j$ or $j = i+1$) 
2. is always 0 because you can't cut a stick that has no cut points in the middle.


i∖j,0 (0),1 (1),2 (3),3 (4),4 (5),5 (7)
0 (0),0,0,3,.,.,.
1 (1),-,0,0,3,.,.
2 (3),-,-,0,0,2,.
3 (4),-,-,-,0,0,3
4 (5),-,-,-,-,0,0

2. Filling Gap = 2 (The first diagonal)
   We look at intervals that have one cut point in the middle.


$dp[0][2]$: Stick $[0,3]$. Cut at index 1 ($1$). Cost = $length(3-0) + 0 + 0 = \mathbf{3}$.
$dp[1][3]$: Stick $[1,4]$. Cut at index 2 ($3$). Cost = $length(4-1) + 0 + 0 = \mathbf{3}$.
$dp[2][4]$: Stick $[3,5]$. Cut at index 3 ($4$). Cost = $length(5-3) + 0 + 0 = \mathbf{2}$.
$dp[3][5]$: Stick $[4,7]$. Cut at index 4 ($5$). Cost = $length(7-4) + 0 + 0 = \mathbf{3}$.


i∖j,0 (0),1 (1),2 (3),3 (4),4 (5),5 (7)
0 (0),0,0,3,.,.,.
1 (1),-,0,0,3,.,.
2 (3),-,-,0,0,2,.
3 (4),-,-,-,0,0,3
4 (5),-,-,-,-,0,0

3. Filling Gap = 3 (The second diagonal)
   Now we have two cut points in the middle. We try both and pick the minimum.

$dp[1][4]$: Stick $[1,5]$. Cuts available at indices 2 and 3 ($3, 4$).

Cut at 3: $length(5-1) + dp[1][2] + dp[2][4] = 4 + 0 + 2 = \mathbf{6}$.
Cut at 4: $length(5-1) + dp[1][3] + dp[3][4] = 4 + 3 + 0 = \mathbf{7}$.
We pick 6.


4. The Final Completed TableAs we increase the gap, we eventually reach $dp[0][5]$, which covers the entire stick.

i∖j,0 (0),1 (1),2 (3),3 (4),4 (5),5 (7)
0 (0),0,0,3,7,10,16
1 (1),-,0,0,3,6,12
2 (3),-,-,0,0,2,6
3 (4),-,-,-,0,0,3
4 (5),-,-,-,-,0,0
5 (7),-,-,-,-,-,0


Key Observations for your Visual Memory:

The Starting Line: You always start with 0s on the main diagonal and the one immediately above it ($j = i+1$).
The Movement: You fill the table diagonally from the bottom-left toward the top-right.
The Calculation: To calculate any cell, the code "looks down" and "looks left" at the cells you already finished in the previous gap sizes.

The Result: The answer is always in the top-right corner ($dp[0][m-1]$), representing the full range of the stick.



The "Mapping" Concept
In almost every Interval DP problem (like Burst Balloons or Matrix Chain Multiplication), there are two types of values:

The Scale: (The length of the stick, the value of the balloon).

The Decision Points: (The locations of the cuts, the indices of the balloons).

The Rule of Thumb: Your DP table should almost always be sized based on the Decision Points, not the Scale.

````python




class Solution:
    def minCost(self, n: int, cuts: List[int]) -> int:

        # prepare the cuts (indices are the key!)
        cuts.sort()

        cuts = [0] + cuts + [n]
        m = len(cuts)

        dp = [ [0] *(m) for i in range(m)]

        for gap in range(2,m):
            for i in range(m-gap):

                j = i+gap

                # Use a variable to track minimum, NOT a list comprehension
                min_val = float('inf')
                for k in range(i+1,j):
                    # Total cost = left part + right part
                    current_split_cost = dp[i][k] + dp[k][j]

                    if current_split_cost < min_val:
                        min_val = current_split_cost

                dp[i][j] = min_val + (cuts[j] - cuts[i])

        return dp[0][m-1]

````