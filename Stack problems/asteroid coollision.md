# Asteroid Collision

**LeetCode 735** | **Difficulty:** Medium | **Category:** Stack, Array, Simulation

---

## Problem Statement

We are given an array `asteroids` of integers representing asteroids in a row.

For each asteroid, the absolute value represents its size, and the sign represents its direction (positive meaning right, negative meaning left). Each asteroid moves at the same speed.

Find out the state of the asteroids after all collisions. If two asteroids meet, the smaller one will explode. If both are the same size, both will explode. Two asteroids moving in the same direction will never meet.

**Constraints:**
- `2 <= asteroids.length <= 10^4`
- `-1000 <= asteroids[i] <= 1000`
- `asteroids[i] != 0`

---

## Examples

### Example 1:
```
Input: asteroids = [5,10,-5]
Output: [5,10]

Explanation:
The 10 and -5 collide resulting in 10. The 5 and 10 never collide.
```

### Example 2:
```
Input: asteroids = [8,-8]
Output: []

Explanation:
The 8 and -8 collide exploding each other.
```

### Example 3:
```
Input: asteroids = [10,2,-5]
Output: [10]

Explanation:
The 2 and -5 collide resulting in -5. The 10 and -5 collide resulting in 10.
```

### Example 4:
```
Input: asteroids = [-2,-1,1,2]
Output: [-2,-1,1,2]

Explanation:
All asteroids moving left never collide with asteroids moving right.
```

---

## Strategy

### Stack-Based Simulation

**Key Insight:** Only asteroids moving right (positive) can collide with asteroids moving left (negative). Use a stack to track surviving asteroids and simulate collisions.

**Why This Works:**

```
Collision Conditions:
1. Positive + Positive → No collision (both moving right)
2. Negative + Negative → No collision (both moving left)
3. Negative + Positive → No collision (moving apart)
4. Positive + Negative → COLLISION! (moving toward each other)

Stack Pattern:
- Stack holds surviving asteroids from left to right
- When we encounter a left-moving asteroid (-):
  - It collides with right-moving asteroids (+) in stack
  - Keep popping smaller right-moving asteroids
  - Stop when:
    a) Left-moving asteroid destroyed (break)
    b) Equal size collision (both destroyed, break)
    c) Stack empty or left-moving asteroid in stack (no collision)

Key: Use while loop with else clause
  - while: Handle collisions
  - else: If no break occurred, asteroid survives
```

**Algorithm Steps:**

1. **Initialize empty stack**

2. **For each asteroid:**
   - **Collision check:** While stack not empty AND current is negative AND stack top is positive:
     - Compare sizes:
       - If current larger: Pop stack, continue checking
       - If equal: Pop stack, destroy current (break)
       - If current smaller: Current destroyed (break)
   - **No collision:** If while loop completes without break, add current to stack

3. **Return stack** (surviving asteroids)

**Complexity:**
- **Time:** $O(n)$ - Each asteroid pushed/popped at most once
- **Space:** $O(n)$ - Stack in worst case

---

## Solution

```python
class Solution:
    def asteroidCollision(self, asteroids: List[int]) -> List[int]:
        """
        Simulate asteroid collisions using stack.
        
        Strategy: Stack-based simulation
        1. Process asteroids left to right
        2. Collision only occurs when: stack top > 0 (right) and current < 0 (left)
        3. Compare absolute values to determine survivor
        4. Use while-else pattern for clean logic
        
        Key insight: Only right-moving asteroids in stack can collide
        with left-moving incoming asteroids.
        
        Edge cases:
        - All same direction: No collisions
        - Equal size collision: Both destroyed
        - Chain collisions: One asteroid destroys multiple
        - Empty stack during collision check: Add asteroid
        
        Args:
            asteroids: Array of asteroid sizes with direction
            
        Returns:
            Final state after all collisions
        """
        stack = []
        
        for a in asteroids:
            # Check for collision: current moving left, stack top moving right
            while stack and a < 0 < stack[-1]:
                # Compare absolute values
                if abs(a) > stack[-1]:
                    # Current asteroid larger, destroy stack top
                    stack.pop()
                    continue  # Check next collision
                    
                elif abs(a) == stack[-1]:
                    # Equal size, both destroyed
                    stack.pop()
                
                # Current asteroid destroyed (or equal case handled)
                break
            else:
                # No collision occurred, or all collisions resolved
                # Add asteroid to stack
                stack.append(a)
        
        return stack
```

---

## Detailed Walkthrough

### Example 1: `asteroids = [5, 10, -5]`

**Processing:**

| Step | Current | stack (before) | Collision Check | Action | stack (after) |
|------|---------|----------------|-----------------|--------|---------------|
| 1 | 5 | [] | No (stack empty) | Add 5 | [5] |
| 2 | 10 | [5] | No (10 > 0) | Add 10 | [5, 10] |
| 3 | -5 | [5, 10] | Yes! (-5 < 0 < 10) | Compare | |

**Collision Details for Step 3:**

```
Current: -5, Stack top: 10

Collision condition: -5 < 0 < 10 ✓

Compare: |-5| = 5 vs 10
  5 < 10 → Current asteroid -5 destroyed
  Break immediately (don't add to stack)

Final stack: [5, 10]
```

**Output:** `[5, 10]` ✓

---

### Example 2: `asteroids = [8, -8]`

**Processing:**

```
Step 1: Current = 8
  stack = []
  No collision (stack empty)
  Add 8
  stack = [8]

Step 2: Current = -8
  stack = [8]
  Collision check: -8 < 0 < 8 ✓
  
  Compare: |-8| = 8 vs 8
    8 == 8 → Both destroyed!
    Pop stack (remove 8)
    Break (don't add -8)
  
  stack = []
```

**Output:** `[]` ✓

---

### Example 3: `asteroids = [10, 2, -5]` (Chain Collision)

**Processing:**

```
Step 1: Current = 10
  stack = []
  Add 10
  stack = [10]

Step 2: Current = 2
  stack = [10]
  No collision (2 > 0)
  Add 2
  stack = [10, 2]

Step 3: Current = -5
  stack = [10, 2]
  
  Collision 1: -5 vs 2
    Condition: -5 < 0 < 2 ✓
    Compare: |-5| = 5 vs 2
      5 > 2 → Pop 2, continue
    stack = [10]
  
  Collision 2: -5 vs 10
    Condition: -5 < 0 < 10 ✓
    Compare: |-5| = 5 vs 10
      5 < 10 → -5 destroyed, break
    stack = [10]
```

**Output:** `[10]` ✓

**Key observation:** The -5 asteroid destroys 2 but then gets destroyed by 10!

---

### Example 4: `asteroids = [-2, -1, 1, 2]` (No Collisions)

**Processing:**

```
Step 1: Current = -2
  stack = []
  No collision (stack empty)
  Add -2
  stack = [-2]

Step 2: Current = -1
  stack = [-2]
  No collision (-2 < 0, condition needs stack[-1] > 0)
  Add -1
  stack = [-2, -1]

Step 3: Current = 1
  stack = [-2, -1]
  No collision (condition needs current < 0)
  Add 1
  stack = [-2, -1, 1]

Step 4: Current = 2
  stack = [-2, -1, 1]
  No collision (2 > 0)
  Add 2
  stack = [-2, -1, 1, 2]
```

**Output:** `[-2, -1, 1, 2]` ✓

**Key observation:** Left-moving asteroids never collide with right-moving asteroids behind them!

---

## Why This Algorithm Works

### Collision Condition Analysis

```
For collision to occur, we need:
1. Current asteroid moving left (a < 0)
2. Stack top moving right (stack[-1] > 0)

Written as: a < 0 < stack[-1]

Why this is sufficient:
  - Positive asteroids moving right
  - Negative asteroids moving left
  - They meet only when right-moving is already placed
    and left-moving arrives later
```

### While-Else Pattern

```python
while stack and a < 0 < stack[-1]:
    # Handle collision
    if abs(a) > stack[-1]:
        stack.pop()
        continue  # Check next asteroid in stack
    elif abs(a) == stack[-1]:
        stack.pop()
    break  # Current destroyed (or equal case)
else:
    # Executes only if while loop completes without break
    stack.append(a)
```

**Why this works:**

```
Case 1: No collision (while condition false)
  → else executes → add asteroid

Case 2: Current destroys all asteroids in path
  → while continues until stack empty or no collision
  → else executes → add asteroid

Case 3: Current destroyed or equal collision
  → break statement
  → else SKIPPED → asteroid not added

The else clause naturally handles "survivor" logic!
```

### Chain Collision Handling

```
Example: [10, 2, 1, -5]

When -5 arrives:
  Iteration 1: -5 vs 1 → 5 > 1 → pop 1, continue
  Iteration 2: -5 vs 2 → 5 > 2 → pop 2, continue
  Iteration 3: -5 vs 10 → 5 < 10 → break (don't add -5)

The while loop naturally handles multiple collisions!
```

---

## Edge Cases

### 1. All Moving Right
```python
asteroids = [1, 2, 3, 4, 5]
# No collisions, all added to stack
# Output: [1, 2, 3, 4, 5]
```

### 2. All Moving Left
```python
asteroids = [-5, -4, -3, -2, -1]
# No collisions (all moving same direction)
# Output: [-5, -4, -3, -2, -1]
```

### 3. Single Asteroid
```python
asteroids = [10]
# Output: [10]

asteroids = [-10]
# Output: [-10]
```

### 4. Two Asteroids Same Size Opposite Directions
```python
asteroids = [5, -5]
# Both destroyed
# Output: []
```

### 5. Large Left Destroying All Right
```python
asteroids = [1, 2, 3, 4, -10]
# -10 destroys all right-moving asteroids
# Output: [-10]
```

### 6. Small Left Destroyed Immediately
```python
asteroids = [10, -1]
# -1 destroyed by 10
# Output: [10]
```

### 7. Multiple Groups
```python
asteroids = [5, 10, -5, -10]
# First collision: 10 vs -5 → 10 survives
# -10 arrives: 10 vs -10 → Both destroyed
# 5 remains
# Output: [5]
```

### 8. Left Then Right (No Collision)
```python
asteroids = [-5, -10, 5, 10]
# Left-moving asteroids placed first
# Right-moving asteroids added (moving away from left ones)
# Output: [-5, -10, 5, 10]
```

### 9. Alternating with Survivors
```python
asteroids = [10, -5, 5, -5]
# 10 vs -5: 10 survives
# Add 5
# 5 vs -5: Both destroyed
# Output: [10]
```

### 10. Equal Chain
```python
asteroids = [5, 5, 5, -5, -5, -5]
# 5,5,5 in stack
# First -5: 5 vs -5, both destroyed
# Second -5: 5 vs -5, both destroyed  
# Third -5: 5 vs -5, both destroyed
# Output: []
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Collision Condition
```python
# Wrong! Checks if both negative
while stack and a < 0 and stack[-1] < 0:
    # This never causes collision (both moving left)

# Correct: Right-moving meets left-moving
while stack and a < 0 < stack[-1]:
```

### ❌ Mistake 2: Not Using abs() for Comparison
```python
# Wrong! Comparing signed values
if a > stack[-1]:  # -5 > 3 is False, but |-5| > 3 is True!
    stack.pop()

# Correct: Compare absolute values (sizes)
if abs(a) > stack[-1]:  # stack[-1] already positive
```

### ❌ Mistake 3: Missing continue After Pop
```python
# Wrong! Breaks after first pop
while stack and a < 0 < stack[-1]:
    if abs(a) > stack[-1]:
        stack.pop()
        # Missing continue! Falls through to break
    break

# Correct: Continue to check next asteroid
if abs(a) > stack[-1]:
    stack.pop()
    continue  # Check next collision
```

### ❌ Mistake 4: Not Using While-Else Pattern
```python
# Wrong! Manual flag tracking
destroyed = False
while stack and a < 0 < stack[-1]:
    if abs(a) > stack[-1]:
        stack.pop()
    elif abs(a) == stack[-1]:
        stack.pop()
        destroyed = True
        break
    else:
        destroyed = True
        break

if not destroyed:
    stack.append(a)

# Correct: Python while-else is cleaner
while stack and a < 0 < stack[-1]:
    # collision logic with break
    ...
else:
    stack.append(a)
```

### ❌ Mistake 5: Breaking on First Collision Always
```python
# Wrong! Stops after first collision check
while stack and a < 0 < stack[-1]:
    if abs(a) > stack[-1]:
        stack.pop()
    break  # Wrong! Should only break if current destroyed

# Breaks even when current asteroid survives and should continue
```

### ❌ Mistake 6: Adding Before Collision Check
```python
# Wrong! Adds asteroid before checking collisions
stack.append(a)
while stack and len(stack) >= 2 and stack[-1] < 0 < stack[-2]:
    # Now logic becomes much more complex!

# Correct: Check collisions first, then add if survived
```

### ❌ Mistake 7: Not Handling Equal Size
```python
# Wrong! Forgets equal size case
while stack and a < 0 < stack[-1]:
    if abs(a) > stack[-1]:
        stack.pop()
        continue
    # Missing: elif abs(a) == stack[-1]
    break

# Both should be destroyed, but this keeps current in stack
```

---

## Interview Insights

**Question Recognition:**
- "Collision" + "direction" + "size" → Stack simulation
- "Opposite directions meet" → Stack-based approach
- "Process elements, some cancel out" → Stack pattern

**Key Points to Mention:**

1. **Collision condition:**
   - "Collision only happens when a right-moving asteroid (positive) in the stack meets a left-moving asteroid (negative) coming from the right."

2. **While-else pattern:**
   - "I use Python's while-else to elegantly handle whether the asteroid survives. If the loop breaks, the asteroid is destroyed; if the loop completes, it's added."

3. **Chain collisions:**
   - "The while loop naturally handles chain collisions where one asteroid destroys multiple smaller ones in sequence."

4. **Time complexity:**
   - "Each asteroid is pushed and popped at most once, giving us O(n) time complexity."

5. **Why stack works:**
   - "The stack represents asteroids from left to right. New left-moving asteroids only collide with the most recent right-moving ones, which is exactly what stack top provides."

**Follow-up Questions:**

1. **Q:** "What if asteroids have different speeds?"
   - **A:** "Much more complex! Would need to simulate time steps and track positions. Current problem assumes equal speeds, making it stack-friendly."

2. **Q:** "Can you do this without a stack?"
   - **A:** "You could use the input array in-place with a pointer, but it's essentially the same as a stack. Stack is the natural data structure."

3. **Q:** "What if we want to know how many collisions occurred?"
   - **A:** "Add a counter and increment it each time we pop from the stack or when equal sizes destroy each other."

4. **Q:** "What about 2D space (asteroids moving in all directions)?"
   - **A:** "Completely different problem! Would need collision detection algorithms, possibly using sweep line or spatial hashing."

5. **Q:** "What if sizes are not guaranteed to be integers?"
   - **A:** "Same algorithm works! Just use abs() and compare floats. Might need epsilon for equality checks."

**Complexity Analysis:**
- Time: O(n) - Each asteroid processed once
- Space: O(n) - Stack in worst case (all same direction)

---

## Related Problems

1. **Daily Temperatures (LeetCode 739)** - Monotonic stack pattern
2. **Remove All Adjacent Duplicates In String (LeetCode 1047)** - Stack-based cancellation
3. **Removing Stars From a String (LeetCode 2390)** - Stack removal pattern
4. **Valid Parentheses (LeetCode 20)** - Stack matching/cancellation
5. **Exclusive Time of Functions (LeetCode 636)** - Stack-based simulation
6. **Car Fleet (LeetCode 853)** - Stack with collision-like logic

---

## Variations

### Variation 1: Count Total Collisions
```python
def countCollisions(self, asteroids):
    """
    Return number of collisions that occur.
    """
    stack = []
    collisions = 0
    
    for a in asteroids:
        while stack and a < 0 < stack[-1]:
            collisions += 1  # A collision occurred
            
            if abs(a) > stack[-1]:
                stack.pop()
                continue
            elif abs(a) == stack[-1]:
                stack.pop()
            
            break
        else:
            stack.append(a)
    
    return collisions
```

### Variation 2: Track Collision History
```python
def asteroidCollisionHistory(self, asteroids):
    """
    Return list of collision events.
    """
    stack = []
    events = []
    
    for a in asteroids:
        while stack and a < 0 < stack[-1]:
            if abs(a) > stack[-1]:
                events.append(f"{stack[-1]} destroyed by {a}")
                stack.pop()
                continue
            elif abs(a) == stack[-1]:
                events.append(f"{stack[-1]} and {a} both destroyed")
                stack.pop()
            else:
                events.append(f"{a} destroyed by {stack[-1]}")
            
            break
        else:
            stack.append(a)
    
    return events, stack
```

### Variation 3: With Asteroid IDs
```python
def asteroidCollisionWithIDs(self, asteroids):
    """
    Track which asteroids survive (by original index).
    """
    stack = []  # Store (value, index) tuples
    
    for i, a in enumerate(asteroids):
        while stack and a < 0 < stack[-1][0]:
            if abs(a) > stack[-1][0]:
                stack.pop()
                continue
            elif abs(a) == stack[-1][0]:
                stack.pop()
            
            break
        else:
            stack.append((a, i))
    
    return [idx for val, idx in stack]
```

### Variation 4: Minimum Destruction Force
```python
def minDestructionForce(self, asteroids):
    """
    Find minimum force needed to destroy all asteroids.
    Add an asteroid of size X at the end moving left.
    """
    # Maximum right-moving asteroid determines minimum force
    max_right = max((a for a in asteroids if a > 0), default=0)
    return max_right + 1 if max_right > 0 else 1
```

### Variation 5: With Shield (One Protection)
```python
def asteroidCollisionWithShield(self, asteroids):
    """
    Each asteroid has one shield (survives first collision equal or smaller).
    """
    stack = []  # Store (value, has_shield)
    
    for a in asteroids:
        while stack and a < 0 < stack[-1][0]:
            val, shield = stack[-1]
            
            if abs(a) > val:
                stack.pop()
                continue
            elif abs(a) == val:
                if shield:
                    # Use shield
                    stack[-1] = (val, False)
                    break  # Current destroyed
                else:
                    # No shield, both destroyed
                    stack.pop()
            
            break
        else:
            stack.append((a, True))
    
    return [val for val, _ in stack]
```

### Variation 6: Bidirectional Collisions
```python
def bidirectionalCollision(self, left_moving, right_moving):
    """
    Two arrays: one moving left from right side, one moving right from left.
    Determine collision point and survivors.
    """
    # Simplified: They meet in middle
    survivors_right = []
    survivors_left = []
    
    i, j = 0, len(left_moving) - 1
    
    while i < len(right_moving) and j >= 0:
        r = right_moving[i]
        l = left_moving[j]
        
        if r > l:
            j -= 1  # Left destroyed
        elif l > r:
            i += 1  # Right destroyed
        else:
            i += 1
            j -= 1  # Both destroyed
    
    # Survivors
    survivors_right = right_moving[i:]
    survivors_left = left_moving[:j+1]
    
    return survivors_left + survivors_right
```

### Variation 7: 3D Space (Multiple Directions)
```python
def asteroidCollision3D(self, asteroids):
    """
    Asteroids in 3D: (size, dx, dy, dz) where d is direction.
    Much more complex - need actual collision detection.
    """
    # Simplified version: Check if directions oppose on any axis
    stack = []
    
    for size, dx, dy, dz in asteroids:
        # This is a sketch - real implementation much more complex
        # Would need to check vector dot products, collision times, etc.
        pass
    
    return stack
    # Note: Real solution requires physics simulation
```

---

## Summary

**Key Takeaways:**

1. **Stack Simulation Pattern:**
   - Stack represents asteroids from left to right
   - Process new asteroids against stack top
   - Collision only when directions oppose (+ meets -)

2. **Collision Condition:**
   - `a < 0 < stack[-1]`
   - Current moving left (negative)
   - Stack top moving right (positive)
   - This is the ONLY collision case

3. **While-Else Pattern:**
   - While loop: Handle collision chain
   - Break: Current destroyed, don't add
   - Else: No break occurred, add to stack
   - Elegant way to handle survival logic

4. **Chain Collisions:**
   - One asteroid can destroy multiple
   - While loop naturally continues checking
   - Use `continue` after pop to keep going

5. **Size Comparison:**
   - Use `abs(a)` vs `stack[-1]` (stack top already positive)
   - Three outcomes: current wins, tie, current loses
   - Tie and lose both use `break` (current destroyed)

**Mental Model:**
```
Think of asteroids as a queue entering a tunnel:
- Left side: Already in tunnel (stack)
- Right side: Entering tunnel (current)
- Collision: When entering left-mover meets existing right-mover
- Stack top: The "gate" that incoming asteroid must pass
- Multiple pops: Incoming asteroid is a wrecking ball!
```

**Template (Asteroid Collision):**
```python
def asteroidCollision(self, asteroids):
    stack = []
    
    for a in asteroids:
        # Collision: current left, stack top right
        while stack and a < 0 < stack[-1]:
            if abs(a) > stack[-1]:
                stack.pop()
                continue  # Check next
            elif abs(a) == stack[-1]:
                stack.pop()  # Both destroyed
            break  # Current destroyed
        else:
            # Survived or no collision
            stack.append(a)
    
    return stack
```

This is a beautiful example of using stack for simulation with cancellation logic!
