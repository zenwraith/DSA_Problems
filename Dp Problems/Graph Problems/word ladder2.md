


# Word Ladder II

## Problem Statement

Given two words (`beginWord` and `endWord`) and a dictionary `wordList`, find **all shortest transformation sequences** from `beginWord` to `endWord`, such that:
- Only one letter can be changed at a time
- Each transformed word must exist in the word list

**Example 1:**
```
Input: beginWord = "hit", endWord = "cog", 
       wordList = ["hot","dot","dog","lot","log","cog"]
Output: [["hit","hot","dot","dog","cog"],
         ["hit","hot","lot","log","cog"]]
Explanation: There are 2 shortest transformation sequences.
```

**Example 2:**
```
Input: beginWord = "hit", endWord = "cog", 
       wordList = ["hot","dot","dog","lot","log"]
Output: []
Explanation: The endWord "cog" is not in wordList, therefore no solution.
```

**Constraints:**
- $1 \leq \text{beginWord.length} \leq 5$
- `endWord.length` == `beginWord.length`
- $1 \leq \text{wordList.length} \leq 500$
- All strings consist of lowercase English letters
- `beginWord` ≠ `endWord`
- All words in `wordList` are unique

---

## Strategy: Hybrid BFS + DFS Approach

### Why This Problem is Challenging

**The DFS-Only Trap:** Using only DFS would lead to:
- Infinite loops without proper tracking
- Finding non-shortest paths first
- Exponential exploration of all possible paths

**The BFS-Only Trap:** Using only BFS would require:
- Storing all paths in the queue: `[ ["hit", "hot"], ["hit", "dot"] ]`
- Memory consumption of $O(2^N)$ for large word lists
- Memory Limit Exceeded on large test cases

### The Hybrid Solution

**Phase 1 - BFS: Build a "Layered Graph"**
- Calculate the shortest distance from `beginWord` to all reachable words
- Build a parent-child adjacency list tracking which words lead to others at the next level
- Process level-by-level to ensure shortest paths

**Phase 2 - DFS: Backtrack from End to Start**
- Start from `endWord` and work backwards to `beginWord`
- Only move to words that are exactly one "layer" closer to the start
- Reconstruct all shortest paths efficiently

### Key Insights

1. **Level-by-Level Processing:** Process entire BFS levels before removing words from the word set, allowing multiple parents to point to the same child if they're at the same distance

2. **Parent Tracking:** Store parent pointers instead of full paths during BFS
   - `adj[next_word].append(word)` means `word` → `next_word` is a valid shortest path edge

3. **Reverse Backtracking:** Working backwards from `endWord` prunes all paths that never reached the goal

4. **Memory Optimization:** Store only graph structure ($O(N \cdot M)$) instead of all paths ($O(2^N)$)

---

## Detailed Walkthrough

### Example: `beginWord = "hit"`, `endWord = "cog"`
`wordList = ["hot","dot","dog","lot","log","cog"]`

#### Phase 1: BFS - Building the Layered Graph

**Initial State:**
```
queue = ["hit"]
distance = {"hit": 0}
adj = {}
wordSet = {"hot","dot","dog","lot","log","cog"}
```

**Level 0:** Process `"hit"`
- Try all transformations: `"ait"`, `"bit"`, ..., `"hot"`, ...
- Found: `"hot"` (in wordSet)
- Update:
  ```
  distance["hot"] = 1
  adj["hot"] = ["hit"]
  queue = ["hot"]
  current_level_visited = {"hot"}
  ```
- After level: Remove `"hot"` from wordSet

**Level 1:** Process `"hot"`
- Transformations of `"hot"`:
  - `"dot"` ✓ (new word, distance 2)
  - `"lot"` ✓ (new word, distance 2)
- Update:
  ```
  distance = {"hit": 0, "hot": 1, "dot": 2, "lot": 2}
  adj = {"hot": ["hit"], "dot": ["hot"], "lot": ["hot"]}
  queue = ["dot", "lot"]
  ```

**Level 2:** Process `"dot"` and `"lot"`
- From `"dot"`:
  - `"dog"` ✓ (new, distance 3)
- From `"lot"`:
  - `"log"` ✓ (new, distance 3)
- Update:
  ```
  distance = {..., "dog": 3, "log": 3}
  adj = {..., "dog": ["dot"], "log": ["lot"]}
  ```

**Level 3:** Process `"dog"` and `"log"`
- From `"dog"`:
  - `"cog"` ✓ (found endWord!)
  - `adj["cog"] = ["dog"]`
- From `"log"`:
  - `"cog"` (already at distance 4, add another parent)
  - `adj["cog"].append("log")` → `["dog", "log"]`

**Final Graph Structure:**
```
hit (0) → hot (1) → dot (2) → dog (3) ↘
                ↘ lot (2) → log (3) → cog (4)
```

#### Phase 2: DFS - Backtracking Paths

Start from `"cog"` with `path = ["cog"]`:

**Branch 1:** `cog` → `dog` → `dot` → `hot` → `hit`
- Path: `["hit", "hot", "dot", "dog", "cog"]`

**Branch 2:** `cog` → `log` → `lot` → `hot` → `hit`
- Path: `["hit", "hot", "lot", "log", "cog"]`

**Result:** 2 shortest paths found!

---

## Edge Cases

1. **EndWord not in dictionary:** Return `[]` immediately
2. **No path exists:** BFS completes without reaching `endWord`, return `[]`
3. **Single transformation:** `beginWord` → `endWord` directly
4. **Multiple paths at same length:** All captured by parent tracking
5. **Large word lists:** Hybrid approach handles efficiently

---

## Complexity Analysis

**Time Complexity:** $O(N \cdot M^2 \cdot 26 + P)$
- $N$ = number of words in wordList
- $M$ = length of each word
- BFS phase: For each word ($N$), try each position ($M$), try 26 letters, create new string ($M$) → $O(N \cdot M^2 \cdot 26)$
- DFS phase: $P$ = number of shortest paths to reconstruct
- Simplified: $O(N \cdot M^2 + P)$

**Space Complexity:** $O(N \cdot M)$
- `distance` dictionary: $O(N)$
- `adj` adjacency list: $O(N \cdot M)$ for storing word strings
- BFS queue: $O(N)$ at most
- DFS recursion stack: $O(M)$ for path length
- Total: $O(N \cdot M)$

---

## Implementation



```python
class Solution:
    def findLadders(self, beginWord: str, endWord: str, wordList: List[str]) -> List[List[str]]:

        wordSet = set(wordList)

        if endWord not in wordSet:
            return []
        # 1. BFS to find the shortest distance to each word
        # distance[word] = min steps from beginWord
        distance = {beginWord: 0}
        # adj[word] = list of previous words that lead to it in shortest path
        adj = defaultdict(list)

        queue = deque([beginWord])
        found = False


        while queue and not found:
            # Process level by level
            current_level_visited = set()

            for _ in range(len(queue)):
                word = queue.popleft()
                # Try all possible 1-letter transformations
                for i in range(len(word)):

                    for char in 'abcdefghijklmnopqrstuvwxyz':

                        next_word = word[:i] + char + word[i+1:]

                        if next_word in wordSet:
                            # If this is the first time we see this word at this level
                            if next_word not in distance:
                                distance[next_word] = distance[word] +1
                                queue.append(next_word)
                                current_level_visited.add(next_word)

                            # If next_word was reached from 'word' in the shortest way
                            if distance.get(next_word) == distance[word] + 1:
                                adj[next_word].append(word)
                                if next_word == endWord:
           
                                    found = True
           # Crucial: update wordSet to prevent cycles, but only after processing the level
            wordSet -= current_level_visited

        if not found :
            return []

        # 2. Backtracking (DFS) to reconstruct paths from endWord to beginWord
        res = [] 
        def backtrack(curr_word, path):
            if curr_word == beginWord:
                res.append(path[::-1]) # Reverse to get begin -> end
                return
            
            for prev_word in adj[curr_word]:
                path.append(prev_word)
                backtrack(prev_word, path)
                path.pop()
        backtrack(endWord, [endWord])
        return res

```

### Code Explanation

**Lines 5-7:** Early termination if `endWord` not in dictionary

**Lines 9-12:** Initialize data structures:
- `distance`: Maps each word to its minimum distance from `beginWord`
- `adj`: Adjacency list storing parent words (reverse edges)

**Lines 17-41:** BFS Phase - Build layered graph
- Process entire levels at once (line 21)
- Try all 26-letter transformations for each position (lines 25-27)
- Add parent pointer only if word is reachable at `distance[word] + 1` (line 35)
- Remove visited words after processing full level (line 43)

**Lines 48-56:** DFS Phase - Backtrack to reconstruct paths
- Start from `endWord`, work backwards using parent pointers
- Reverse final path to get `beginWord` → `endWord` order (line 51)
- Recursively explore all parent paths

**Lines 57-58:** Trigger backtracking and return all shortest paths

---

## Visual Representation

```
Word Transformation Graph (Example):

     hit
      |
     hot ─── Distance 1
    /   \
  dot   lot ─── Distance 2
   |     |
  dog   log ─── Distance 3
    \   /
     cog ─── Distance 4

Parent Pointers (adj):
hot  → [hit]
dot  → [hot]
lot  → [hot]
dog  → [dot]
log  → [lot]
cog  → [dog, log]  ← Multiple parents!
```

---

## Alternative Approaches Comparison

| Approach | Time | Space | Pros | Cons |
|----------|------|-------|------|------|
| Pure DFS | $O(26^M)$ | $O(M)$ | Simple | Non-shortest paths, infinite loops |
| Pure BFS (with paths) | $O(N \cdot M^2)$ | $O(2^N)$ | Finds shortest | Memory explosion |
| **BFS + DFS (this)** | $O(N \cdot M^2)$ | $O(N \cdot M)$ | Optimal memory, shortest paths | More complex |
| Bidirectional BFS | $O(N \cdot M^2)$ | $O(N)$ | Faster for length only | Hard to get all paths |

---

## Common Pitfalls

1. **Removing words too early:** Must process entire level before removing from wordSet
2. **Not handling multiple parents:** Missing paths when multiple words lead to same next word
3. **Storing full paths in BFS queue:** Causes memory limit exceeded
4. **Not checking if endWord exists:** Wastes computation time
5. **Forward DFS instead of backward:** Explores many dead-end paths

---