# Word Ladder

**LeetCode Link:** [https://leetcode.com/problems/word-ladder/](https://leetcode.com/problems/word-ladder/)

---

## Problem Statement

A **transformation sequence** from word `beginWord` to word `endWord` using a dictionary `wordList` is a sequence of words `beginWord -> s₁ -> s₂ -> ... -> sₖ` such that:

- Every adjacent pair of words differs by a single letter
- Every `sᵢ` for `1 <= i <= k` is in `wordList` (Note: `beginWord` does not need to be in `wordList`)
- `sₖ == endWord`

Given two words, `beginWord` and `endWord`, and a dictionary `wordList`, return **the length of the shortest transformation sequence** from `beginWord` to `endWord`, or `0` if no such sequence exists.

**Example 1:**
```
Input: beginWord = "hit", endWord = "cog", 
       wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5
Explanation: One shortest transformation sequence is:
"hit" -> "hot" -> "dot" -> "dog" -> "cog" (5 words)
```

**Example 2:**
```
Input: beginWord = "hit", endWord = "cog", 
       wordList = ["hot","dot","dog","lot","log"]
Output: 0
Explanation: endWord "cog" is not in wordList.
```

**Constraints:**
- $1 \leq \text{beginWord.length} \leq 10$
- `endWord.length` == `beginWord.length`
- $1 \leq \text{wordList.length} \leq 5000$
- `wordList[i].length` == `beginWord.length`
- `beginWord`, `endWord`, and `wordList[i]` consist of lowercase English letters
- `beginWord` ≠ `endWord`
- All strings in `wordList` are **unique**

---

## Strategy: Bidirectional BFS

### The Key Insight

Instead of searching from `beginWord` to `endWord` in one direction (which explores a large search space), we search **simultaneously from both ends** and meet in the middle!

**Why This Is Faster:**
- Normal BFS explores exponentially: Level $k$ has roughly $26^k$ nodes
- Bidirectional BFS explores $26^{k/2}$ from each end
- Meeting in middle: $2 \cdot 26^{k/2} << 26^k$
- **Massive speedup** for large search spaces!

### Visualization

```
Normal BFS (one direction):
BEGIN → · → ·· → ···· → ········ → END
       1    2     4      8        16    (exponential growth)

Bidirectional BFS:
BEGIN → · → ··           ·· ← · ← END
       1    2             2    1
            ↓             ↑
            Meet in middle! (much smaller search space)
```

### Algorithm Steps

1. **Initialization:**
   - Create two sets: `head` (start from begin) and `tail` (start from end)
   - Track visited words globally
   - Start with level = 1

2. **Bidirectional Search:**
   - Always expand the smaller set (optimization!)
   - For each word in current set:
     - Try all possible one-letter transformations
     - **If transformation found in opposite set:** Connection made! Return level + 1
     - **If transformation in wordList and unvisited:** Add to next level
   - Swap sets and continue

3. **Termination:**
   - Return level when sets meet
   - Return 0 if either set becomes empty (no path exists)

---

## Detailed Walkthrough

### Example: `beginWord = "hit"`, `endWord = "cog"`
`wordList = ["hot","dot","dog","lot","log","cog"]`

#### Initialization

```
head = {"hit"}
tail = {"cog"}
visited = {"hit", "cog"}
level = 1
word_set = {"hot","dot","dog","lot","log","cog"}
```

#### Level 1: Expand from "hit"

**head is smaller (size 1), so expand from head**

For word `"hit"`:
- Try position 0: `"ait"`, `"bit"`, ..., `"zit"` → none in word_set
- Try position 1: `"hat"`, `"hbt"`, ..., `"hot"` ✓ in word_set!
  - Is `"hot"` in tail? No
  - Is `"hot"` visited? No
  - Add to next_head
- Try position 2: `"hia"`, `"hib"`, ..., `"hiz"` → none in word_set

```
next_head = {"hot"}
visited = {"hit", "cog", "hot"}
head = {"hot"}
level = 2
```

#### Level 2: Expand from "cog"

**Both sets size 1, so expand from tail (arbitrarily)**

For word `"cog"`:
- Transformations: `"aog"`, `"bog"`, ..., `"dog"` ✓, ..., `"log"` ✓
  - `"dog"` in word_set and unvisited ✓
  - `"log"` in word_set and unvisited ✓

```
tail = {"dog", "log"}
visited = {"hit", "cog", "hot", "dog", "log"}
level = 2
```

#### Level 3: Expand from "hot"

**head smaller (size 1), expand from head**

For word `"hot"`:
- Transformations: `"aot"`, ..., `"dot"` ✓, ..., `"lot"` ✓
  - `"dot"` in word_set, unvisited, NOT in tail
  - `"lot"` in word_set, unvisited, **IS IN TAIL!** 🎯

**Connection found! Return level + 1 = 3 + 1 = 4**

Wait, let me recalculate...

Actually, let me trace through more carefully:

#### Corrected Trace:

**Level 1:**
- head = {"hit"}, tail = {"cog"}
- Expand hit: finds "hot"
- head = {"hot"}, level = 2

**Level 2:**
- head = {"hot"} (size 1), tail = {"cog"} (size 1)
- Expand hot: finds "dot", "lot"
- head = {"dot", "lot"}, level = 3

**Level 3:**
- head = {"dot", "lot"} (size 2), tail = {"cog"} (size 1)
- Swap! Expand tail (cog): finds "dog", "log"
- Check if any in head? "log" from "lot"? No direct connection yet
- tail = {"dog", "log"}, level = 4

**Level 4:**
- head = {"dot", "lot"} (size 2), tail = {"dog", "log"} (size 2)
- Expand head:
  - From "dot": finds "dog" → **dog IS IN TAIL!** 
  - Return level + 1 = 4 + 1 = 5 ✓

**Path:** hit → hot → dot → dog → cog (5 words)

---

## Key Insights

1. **Always Expand Smaller Set:** Minimizes branching factor
   ```python
   if len(head) > len(tail):
       head, tail = tail, head
   ```

2. **Check for Meeting:** Before adding to next level, check if word is in opposite set

3. **Global Visited Set:** Prevents cycles and redundant exploration from both directions

4. **Level Tracking:** Represents number of words in sequence (not edges)

5. **Early Termination:** As soon as sets meet, we've found shortest path

---

## Edge Cases

1. **endWord not in dictionary:** Return 0 immediately
2. **No transformation possible:** One or both sets become empty
3. **Direct transformation:** begin and end differ by one letter
4. **Single word dictionary:** Check if it connects begin and end
5. **Very long words:** Algorithm still efficient due to bidirectional approach

---

## Complexity Analysis

**Time Complexity:** $O(M^2 \cdot N)$
- $N$ = number of words in dictionary
- $M$ = length of each word
- For each word, try $M$ positions × 26 letters = $O(M \cdot 26)$
- String creation for each transformation: $O(M)$
- Total per word: $O(M^2)$
- Process up to $N$ words: $O(M^2 \cdot N)$

**Bidirectional optimization** reduces the practical constant significantly, but asymptotic complexity remains the same.

**Space Complexity:** $O(N \cdot M)$
- `word_set`: $O(N \cdot M)$ to store all words
- `head` and `tail` sets: $O(N)$ in worst case
- `visited` set: $O(N)$
- Total: $O(N \cdot M)$

---

## Implementation

```python

import string

class Solution:
    def ladderLength(self, beginWord: str, endTail: str, wordList: List[str]) -> int:

        word_set = set(wordList)

        if endTail not in word_set:
            return 0
        
        head = {beginWord}
        tail = {endTail}
        visited = {beginWord , endTail}
        level = 1

        while head and tail:

            if len(head) > len(tail):
                head,tail = tail,head
            
            next_head = set()

            for word in head:
                for i in range(len(word)):
                    for char in string.ascii_lowercase:
                        if char == word[i]:
                            continue
                        
                        neighbor = word[:i] + char + word[i+1:]
                        
                        if neighbor in tail:
                            return level +1
                        
                        if neighbor in word_set and neighbor not in visited:
                            visited.add(neighbor)
                            next_head.add(neighbor)

            head = next_head
            level+=1

        return 0




```

### Code Explanation

**Lines 7-9:** Initialize word set and check if endWord exists

**Lines 11-14:** Initialize bidirectional search:
- `head`: Set starting from beginWord
- `tail`: Set starting from endWord  
- `visited`: Track all visited words from both directions
- `level`: Current path length

**Lines 16-39:** Bidirectional BFS loop

**Lines 18-19:** **Optimization:** Always expand the smaller set to minimize branching

**Line 21:** Create set for next level of expansion

**Lines 23-36:** For each word in current set:
- **Lines 24-28:** Try all possible transformations:
  - Each position in word
  - Each letter a-z (skip if same as current)
  - Create new word with letter substitution
  
- **Lines 30-31:** **Meeting check:** If neighbor in opposite set, paths connected!
  - Return level + 1 (add 1 for the connecting edge)

- **Lines 33-35:** If neighbor valid and unvisited:
  - Mark as visited
  - Add to next level

**Lines 37-38:** Move to next level

**Line 41:** No path found, return 0

---

## Visual Representation

```
Example: hit → cog

Word Graph:
    hit
     |
    hot
    / \
  dot lot
   |   |
  dog log
    \ /
    cog

Bidirectional Search:
Level 1: 
head: {hit}         tail: {cog}
        ↓                 ↑

Level 2:
head: {hot}         tail: {cog}
       /  \              ↑

Level 3:
head: {dot, lot}    tail: {cog}
         ↓               / \

Level 4:
head: {dot, lot}    tail: {dog, log}
         ↓               ↓
      MEET AT dog!  (or log)
      
Return: 5
```

---

## Comparison: Normal vs Bidirectional BFS

### Normal BFS

```python
def ladderLength_normal(self, beginWord, endWord, wordList):
    if endWord not in wordList:
        return 0
    
    queue = deque([(beginWord, 1)])
    visited = {beginWord}
    word_set = set(wordList)
    
    while queue:
        word, level = queue.popleft()
        
        if word == endWord:
            return level
        
        for i in range(len(word)):
            for c in 'abcdefghijklmnopqrstuvwxyz':
                neighbor = word[:i] + c + word[i+1:]
                if neighbor in word_set and neighbor not in visited:
                    visited.add(neighbor)
                    queue.append((neighbor, level + 1))
    
    return 0
```

### Comparison Table

| Aspect | Normal BFS | Bidirectional BFS |
|--------|-----------|------------------|
| Time (average case) | $O(26^d)$ | $O(2 \cdot 26^{d/2})$ |
| Search space | Full depth $d$ | Half depth $d/2$ from each end |
| Implementation | Simpler | More complex (two sets) |
| Early termination | When reaching end | When sets meet |
| Memory | One queue | Two sets + visited |
| **Best for** | Small search space | Large search space |

---

## Alternative Approaches

| Approach | Time | Space | Pros | Cons |
|----------|------|-------|------|------|
| Normal BFS | $O(M^2 \cdot N)$ | $O(N)$ | Simple, guaranteed shortest | Slow for deep searches |
| **Bidirectional BFS** | $O(M^2 \cdot N)$ | $O(N)$ | Faster in practice | More complex |
| DFS | $O(26^M)$ | $O(M)$ | - | No shortest path guarantee |
| Dijkstra | $O(N \log N)$ | $O(N)$ | Works with weights | Overkill for unweighted |

---

## Common Pitfalls

1. **Not checking if endWord exists:** Wastes computation
2. **Forgetting to swap head/tail:** Loses bidirectional optimization
3. **Not using global visited:** May visit same word from both directions
4. **Off-by-one in level counting:** Counting edges vs vertices
5. **Not skipping same-character substitutions:** Unnecessary work

---

## When to Use Bidirectional BFS

✅ **Use when:**
- Finding shortest path between two specific nodes
- Large search space with known start and end
- Want significant performance improvement

❌ **Don't use when:**
- Need to find paths to multiple targets
- End node unknown
- Graph too small (overhead not worth it)

---

## Related Problems

- **127. Word Ladder** (this problem)
- **126. Word Ladder II** (find all shortest paths)
- **433. Minimum Genetic Mutation** (same pattern)
- **752. Open the Lock** (bidirectional BFS applicable)

---
