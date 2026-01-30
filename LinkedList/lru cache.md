# LRU Cache

**LeetCode 146** | **Difficulty:** Medium | **Category:** Hash Table, Linked List, Design, Doubly-Linked List

---

## Problem Statement

Design a data structure that follows the constraints of a **Least Recently Used (LRU) cache**.

Implement the `LRUCache` class:

- `LRUCache(int capacity)` Initialize the LRU cache with **positive** size `capacity`.
- `int get(int key)` Return the value of the `key` if the key exists, otherwise return `-1`. **Move the accessed key to the most recent position.**
- `void put(int key, int value)` Update the value of the `key` if the `key` exists. Otherwise, add the `key-value` pair to the cache. If the number of keys exceeds the `capacity` from this operation, **evict the least recently used key**.

The functions `get` and `put` must each run in **O(1)** average time complexity.

**Constraints:**
- `1 <= capacity <= 3000`
- `0 <= key <= 10^4`
- `0 <= value <= 10^5`
- At most `2 * 10^5` calls will be made to `get` and `put`

---

## Examples

### Example 1:
```
Input:
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]

Output:
[null, null, null, 1, null, -1, null, -1, 3, 4]

Explanation:
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1);  // cache is {1=1}
lRUCache.put(2, 2);  // cache is {1=1, 2=2}
lRUCache.get(1);     // return 1, cache is {2=2, 1=1} (1 becomes most recent)
lRUCache.put(3, 3);  // LRU key was 2, evicts key 2, cache is {1=1, 3=3}
lRUCache.get(2);     // returns -1 (not found)
lRUCache.put(4, 4);  // LRU key was 1, evicts key 1, cache is {4=4, 3=3}
lRUCache.get(1);     // return -1 (not found)
lRUCache.get(3);     // return 3, cache is {4=4, 3=3} (3 becomes most recent)
lRUCache.get(4);     // return 4
```

---

## Strategy

### Hash Map + Doubly-Linked List

**Key Insight:** Combine a hash map for O(1) lookup with a doubly-linked list for O(1) insertion/deletion to track recency order.

**Why This Works:**

```
Requirements:
1. O(1) get(key) → Need hash map for fast lookup
2. O(1) put(key, value) → Need hash map for fast update
3. Track "least recently used" → Need ordering by recency
4. O(1) eviction → Need fast removal from middle and ends

Challenge: Hash map provides O(1) lookup but no ordering.
           List provides ordering but O(n) lookup.

Solution: Combine both!
  - Hash map: key → node reference (O(1) lookup)
  - Doubly-linked list: maintains recency order
    - Most recent: Near tail (right)
    - Least recent: Near head (left)

Doubly-linked list operations:
  - Remove node: O(1) if we have reference
  - Add to end: O(1)
  - Remove from start: O(1)

Why doubly-linked (not singly)?
  - Need to remove from middle: requires prev pointer
  - Need to add before tail: requires prev pointer

Structure:
  [Dummy Head] ↔ [LRU] ↔ ... ↔ [MRU] ↔ [Dummy Tail]
       ↑                                      ↑
     left                                  right

Dummy nodes prevent null checks.
```

**Algorithm Steps:**

**get(key):**
1. If key not in cache: return -1
2. Get node from hash map
3. Remove node from current position
4. Insert node at end (most recent)
5. Return node's value

**put(key, value):**
1. If key exists in cache:
   - Remove old node from list
2. Create new node
3. Add to hash map
4. Insert at end (most recent)
5. If cache exceeds capacity:
   - Remove LRU node (left.next)
   - Delete from hash map

**Complexity:**
- **Time:** O(1) for both get and put
- **Space:** O(capacity) for hash map and list

---

## Solution

```python
class Node:
    """Doubly-linked list node."""
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None


class LRUCache:
    """
    LRU Cache implementation using hash map + doubly-linked list.
    
    Structure:
        Hash map: key → node (O(1) lookup)
        Doubly-linked list: [left] ↔ LRU ↔ ... ↔ MRU ↔ [right]
    
    Key insight: Combine hash map's fast lookup with linked list's
    fast insertion/deletion to maintain recency order.
    
    Operations:
        - get(key): Move accessed node to MRU position (right)
        - put(key, val): Add to MRU, evict LRU if full
    
    Edge cases:
        - Capacity 1: Each put evicts previous
        - Update existing key: Remove old, add new
        - Access updates recency: Move to MRU position
    """
    
    def __init__(self, capacity: int):
        """
        Initialize LRU cache with given capacity.
        
        Args:
            capacity: Maximum number of items in cache
        """
        self.cap = capacity
        self.cache = {}  # key → node
        
        # Dummy nodes to avoid null checks
        self.left = Node(0, 0)   # Dummy head (before LRU)
        self.right = Node(0, 0)  # Dummy tail (after MRU)
        
        # Connect dummy nodes
        self.left.next = self.right
        self.right.prev = self.left
    
    def remove(self, node):
        """
        Remove node from linked list.
        
        O(1) operation using prev/next pointers.
        
        Args:
            node: Node to remove from list
        """
        prev, nxt = node.prev, node.next
        prev.next = nxt
        nxt.prev = prev
    
    def insert(self, node):
        """
        Insert node at end (most recent position, before right dummy).
        
        O(1) operation.
        
        Args:
            node: Node to insert at MRU position
        """
        prev, nxt = self.right.prev, self.right
        prev.next = node
        nxt.prev = node
        node.prev = prev
        node.next = nxt
    
    def get(self, key: int) -> int:
        """
        Get value for key, mark as most recently used.
        
        Strategy:
            1. Check if key exists
            2. If yes: remove from current position
            3. Insert at end (MRU position)
            4. Return value
            5. If no: return -1
        
        Time: O(1)
        
        Args:
            key: Key to lookup
            
        Returns:
            Value if key exists, -1 otherwise
        """
        if key in self.cache:
            # Move to most recent position
            node = self.cache[key]
            self.remove(node)
            self.insert(node)
            return node.val
        
        return -1
    
    def put(self, key: int, value: int) -> None:
        """
        Add/update key-value pair, evict LRU if needed.
        
        Strategy:
            1. If key exists: remove old node
            2. Create new node
            3. Add to cache and insert at MRU position
            4. If over capacity: evict LRU (left.next)
        
        Time: O(1)
        
        Args:
            key: Key to add/update
            value: Value to store
        """
        # If key exists, remove old node
        if key in self.cache:
            self.remove(self.cache[key])
        
        # Create and insert new node
        self.cache[key] = Node(key, value)
        self.insert(self.cache[key])
        
        # Evict LRU if over capacity
        if len(self.cache) > self.cap:
            # LRU is left.next (first real node)
            lru = self.left.next
            self.remove(lru)
            del self.cache[lru.key]
```

---

## Detailed Walkthrough

### Example: Capacity = 2

**Initial State:**
```
Cache: {}
List: [left] ↔ [right]
```

---

**Operation 1: `put(1, 1)`**

```
Create node(1, 1)
Add to cache: {1: node(1,1)}
Insert at MRU position

List: [left] ↔ [1:1] ↔ [right]
             LRU      MRU
```

---

**Operation 2: `put(2, 2)`**

```
Create node(2, 2)
Add to cache: {1: node(1,1), 2: node(2,2)}
Insert at MRU position

List: [left] ↔ [1:1] ↔ [2:2] ↔ [right]
             LRU      MRU
```

---

**Operation 3: `get(1)`**

```
Key 1 exists!
Remove node(1,1) from current position:
  List: [left] ↔ [2:2] ↔ [right]

Insert at MRU position:
  List: [left] ↔ [2:2] ↔ [1:1] ↔ [right]
             LRU      MRU

Return: 1
```

---

**Operation 4: `put(3, 3)`**

```
Create node(3, 3)
Add to cache: {1: node(1,1), 2: node(2,2), 3: node(3,3)}
Insert at MRU position:
  List: [left] ↔ [2:2] ↔ [1:1] ↔ [3:3] ↔ [right]

Check capacity: 3 > 2!
Evict LRU (left.next = node(2,2)):
  Remove node(2,2) from list
  Delete cache[2]

Final:
  Cache: {1: node(1,1), 3: node(3,3)}
  List: [left] ↔ [1:1] ↔ [3:3] ↔ [right]
               LRU      MRU
```

---

**Operation 5: `get(2)`**

```
Key 2 not in cache
Return: -1
```

---

**Operation 6: `put(4, 4)`**

```
Create node(4, 4)
Add to cache: {1: node(1,1), 3: node(3,3), 4: node(4,4)}
Insert at MRU position:
  List: [left] ↔ [1:1] ↔ [3:3] ↔ [4:4] ↔ [right]

Check capacity: 3 > 2!
Evict LRU (left.next = node(1,1)):
  Remove node(1,1)
  Delete cache[1]

Final:
  Cache: {3: node(3,3), 4: node(4,4)}
  List: [left] ↔ [3:3] ↔ [4:4] ↔ [right]
               LRU      MRU
```

---

**Operation 7: `get(1)`**

```
Key 1 not in cache
Return: -1
```

---

**Operation 8: `get(3)`**

```
Key 3 exists!
Remove node(3,3):
  List: [left] ↔ [4:4] ↔ [right]

Insert at MRU:
  List: [left] ↔ [4:4] ↔ [3:3] ↔ [right]
               LRU      MRU

Return: 3
```

---

**Operation 9: `get(4)`**

```
Key 4 exists!
Remove node(4,4):
  List: [left] ↔ [3:3] ↔ [right]

Insert at MRU:
  List: [left] ↔ [3:3] ↔ [4:4] ↔ [right]
               LRU      MRU

Return: 4
```

---

## Why This Algorithm Works

### Hash Map for O(1) Lookup

```
Without hash map:
  To find node with key k: O(n) scan through list

With hash map:
  cache[k] → node reference: O(1)

Trade-off: O(n) space for O(1) time
```

### Doubly-Linked List for O(1) Removal

```
Why doubly-linked?

Removal requires updating neighbors:
  node.prev.next = node.next
  node.next.prev = node.prev

With singly-linked list:
  - Can't access prev without O(n) scan
  - Would break O(1) guarantee

Doubly-linked:
  - Direct access to both neighbors
  - O(1) removal from any position
```

### Dummy Nodes Simplify Edge Cases

```
Without dummy nodes:
  - Need null checks for head/tail
  - Special cases for empty list
  - Complex insertion logic

With dummy nodes:
  - left.next always points to LRU (or right if empty)
  - right.prev always points to MRU (or left if empty)
  - No null checks needed
  - Uniform insertion/removal code

Example:
  Empty list: [left] ↔ [right]
  One node: [left] ↔ [node] ↔ [right]
  
  left.next is always valid!
```

### Why Insert at Right (Before right dummy)

```
Convention: Right = Most Recent

get(key):
  - Access means "recently used"
  - Move to right (MRU position)

put(key, value):
  - New item is "recently used"
  - Insert at right (MRU position)

Eviction:
  - left.next is LRU
  - Remove from left side
  - Natural ordering: LRU ← ... → MRU
```

---

## Edge Cases

### 1. Capacity 1
```python
cache = LRUCache(1)
cache.put(1, 1)  # {1: 1}
cache.put(2, 2)  # Evicts 1, {2: 2}
cache.get(1)     # -1
cache.get(2)     # 2
```

### 2. Update Existing Key
```python
cache = LRUCache(2)
cache.put(1, 1)  # {1: 1}
cache.put(2, 2)  # {1: 1, 2: 2}
cache.put(1, 10) # Update: {1: 10, 2: 2}, 1 becomes MRU
cache.put(3, 3)  # Evicts 2: {1: 10, 3: 3}
```

### 3. All Gets
```python
cache = LRUCache(2)
cache.put(1, 1)
cache.put(2, 2)
cache.get(1)  # 1, order: [2, 1]
cache.get(2)  # 2, order: [1, 2]
cache.get(1)  # 1, order: [2, 1]
# No evictions
```

### 4. Alternating Access
```python
cache = LRUCache(2)
cache.put(1, 1)
cache.put(2, 2)
cache.get(1)     # 1, order: [2, 1]
cache.put(3, 3)  # Evicts 2: {1, 3}
cache.get(1)     # 1, order: [3, 1]
cache.put(4, 4)  # Evicts 3: {1, 4}
```

### 5. Same Key Multiple Updates
```python
cache = LRUCache(1)
cache.put(1, 1)
cache.put(1, 2)
cache.put(1, 3)
cache.get(1)  # 3
```

### 6. Large Capacity, No Evictions
```python
cache = LRUCache(100)
for i in range(50):
    cache.put(i, i)
# All 50 items remain
```

### 7. Zero Values
```python
cache = LRUCache(2)
cache.put(0, 0)
cache.get(0)  # 0 (valid value, not error)
```

### 8. Sequential Evictions
```python
cache = LRUCache(2)
cache.put(1, 1)
cache.put(2, 2)
cache.put(3, 3)  # Evicts 1
cache.put(4, 4)  # Evicts 2
cache.get(1)     # -1
cache.get(2)     # -1
```

---

## Common Mistakes

### ❌ Mistake 1: Using Singly-Linked List
```python
# Wrong! Can't remove from middle in O(1)
class Node:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.next = None  # Missing prev!

# Remove requires finding prev: O(n)
```

### ❌ Mistake 2: Not Removing Before Re-inserting
```python
# Wrong! Creates duplicates in list
def get(self, key):
    if key in self.cache:
        # Missing: self.remove(self.cache[key])
        self.insert(self.cache[key])  # Now in list twice!
        return self.cache[key].val
```

### ❌ Mistake 3: Forgetting to Update Hash Map on Eviction
```python
# Wrong! Stale entry in hash map
if len(self.cache) > self.cap:
    lru = self.left.next
    self.remove(lru)
    # Missing: del self.cache[lru.key]
```

### ❌ Mistake 4: Storing Value in Node Without Key
```python
# Wrong! Can't delete from hash map during eviction
class Node:
    def __init__(self, val):
        self.val = val  # Missing key!

# On eviction:
lru = self.left.next
# Can't do: del self.cache[lru.key]  # No key stored!
```

### ❌ Mistake 5: Not Using Dummy Nodes
```python
# Wrong! Complex null checks
def __init__(self, capacity):
    self.cap = capacity
    self.cache = {}
    self.head = None  # No dummy
    self.tail = None

# Remove becomes complex:
def remove(self, node):
    if node.prev:
        node.prev.next = node.next
    else:
        self.head = node.next  # Special case!
    # More special cases...
```

### ❌ Mistake 6: Wrong Insert Position
```python
# Wrong! Inserts at head (LRU position) instead of tail (MRU)
def insert(self, node):
    node.next = self.left.next
    self.left.next.prev = node
    self.left.next = node
    node.prev = self.left
    # This makes new items LRU instead of MRU!
```

### ❌ Mistake 7: Not Creating New Node on Update
```python
# Wrong! Reuses node without updating value
def put(self, key, value):
    if key in self.cache:
        self.remove(self.cache[key])
        # Missing: update node.val or create new node
        self.insert(self.cache[key])  # Old value!
```

### ❌ Mistake 8: Checking Capacity Before Insertion
```python
# Wrong! Checks capacity before adding new node
if len(self.cache) >= self.cap:  # Should be >
    # Evict
# This evicts too early
```

---

## Interview Insights

**Question Recognition:**
- "LRU cache" + "O(1)" → Hash map + doubly-linked list
- "Most/least recently used" → Ordering required
- "Design a cache" → Data structure design problem

**Key Points to Mention:**

1. **Dual data structure:**
   - "I'll combine a hash map for O(1) lookup with a doubly-linked list to maintain recency order."

2. **Why doubly-linked:**
   - "Doubly-linked list allows O(1) removal from any position, which is essential when moving accessed items to the MRU position."

3. **Dummy nodes:**
   - "I use dummy head and tail nodes to avoid null checks and simplify edge cases."

4. **Why store key in node:**
   - "When evicting the LRU node, I need its key to delete from the hash map, so each node stores both key and value."

5. **Time complexity:**
   - "Both get and put are O(1) because hash map gives O(1) lookup and doubly-linked list gives O(1) insertion/deletion."

**Follow-up Questions:**

1. **Q:** "What if we need LFU (Least Frequently Used) instead?"
   - **A:** "Would need to track frequency counts. Use hash map for key→node, another for frequency→list of nodes, and track min frequency."

2. **Q:** "How would you make this thread-safe?"
   - **A:** "Add locks around get/put operations. Use ReentrantLock or synchronized blocks. Consider read-write locks for better concurrency."

3. **Q:** "What if capacity is very large?"
   - **A:** "Same approach works. Hash map and linked list scale linearly. Memory might be a concern, could consider serialization for overflow."

4. **Q:** "Can you use OrderedDict in Python?"
   - **A:** "Yes, Python's OrderedDict maintains insertion order and can be used with move_to_end(), but implementing from scratch shows understanding."

5. **Q:** "What about TTL (Time To Live) for cache entries?"
   - **A:** "Would need to store timestamp with each node. On access, check if expired. Could use a separate thread to periodically clean up."

**Complexity Analysis:**
- Time: O(1) for get and put
- Space: O(capacity) for hash map and linked list

---

## Related Problems

1. **LFU Cache (LeetCode 460)** - Least Frequently Used variant
2. **Design In-Memory File System (LeetCode 588)** - Similar design problem
3. **Insert Delete GetRandom O(1) (LeetCode 380)** - Hash map + array design
4. **All O`one Data Structure (LeetCode 432)** - Similar dual structure design
5. **Design HashMap (LeetCode 706)** - Hash map implementation

---

## Variations

### Variation 1: With TTL (Time To Live)
```python
import time

class NodeWithTTL:
    def __init__(self, key, val, ttl):
        self.key = key
        self.val = val
        self.expiry = time.time() + ttl
        self.prev = None
        self.next = None

class LRUCacheWithTTL:
    def __init__(self, capacity, default_ttl):
        self.cap = capacity
        self.default_ttl = default_ttl
        self.cache = {}
        self.left = NodeWithTTL(0, 0, float('inf'))
        self.right = NodeWithTTL(0, 0, float('inf'))
        self.left.next = self.right
        self.right.prev = self.left
    
    def is_expired(self, node):
        return time.time() > node.expiry
    
    def get(self, key):
        if key in self.cache:
            node = self.cache[key]
            if self.is_expired(node):
                self.remove(node)
                del self.cache[key]
                return -1
            self.remove(node)
            self.insert(node)
            return node.val
        return -1
    
    # Similar put with TTL...
```

### Variation 2: LFU (Least Frequently Used)
```python
class LFUCache:
    """
    LFU Cache: Evicts least frequently used item.
    If tie, evicts least recently used among them.
    """
    def __init__(self, capacity):
        self.cap = capacity
        self.cache = {}  # key → (value, freq)
        self.freq_map = {}  # freq → OrderedDict of keys
        self.min_freq = 0
    
    def get(self, key):
        if key not in self.cache:
            return -1
        
        val, freq = self.cache[key]
        # Update frequency
        self.freq_map[freq].pop(key)
        if not self.freq_map[freq] and freq == self.min_freq:
            self.min_freq += 1
        
        self.cache[key] = (val, freq + 1)
        if freq + 1 not in self.freq_map:
            self.freq_map[freq + 1] = OrderedDict()
        self.freq_map[freq + 1][key] = val
        
        return val
```

### Variation 3: Size-Aware Cache
```python
class SizeAwareLRUCache:
    """
    LRU cache where items have different sizes.
    Evict based on total size rather than count.
    """
    def __init__(self, max_size):
        self.max_size = max_size
        self.current_size = 0
        self.cache = {}
        self.left = Node(0, 0)
        self.right = Node(0, 0)
        self.left.next = self.right
        self.right.prev = self.left
    
    def put(self, key, value, size):
        # Evict until we have space
        while self.current_size + size > self.max_size:
            lru = self.left.next
            self.current_size -= lru.size
            self.remove(lru)
            del self.cache[lru.key]
        
        # Add new item
        # ... similar logic
```

### Variation 4: Multi-Level Cache
```python
class MultiLevelCache:
    """
    L1 (fast, small) → L2 (medium) → L3 (slow, large)
    Automatically promotes frequently accessed items to L1.
    """
    def __init__(self, l1_cap, l2_cap, l3_cap):
        self.l1 = LRUCache(l1_cap)
        self.l2 = LRUCache(l2_cap)
        self.l3 = LRUCache(l3_cap)
    
    def get(self, key):
        # Check L1
        val = self.l1.get(key)
        if val != -1:
            return val
        
        # Check L2, promote to L1
        val = self.l2.get(key)
        if val != -1:
            self.l1.put(key, val)
            return val
        
        # Check L3, promote to L2
        val = self.l3.get(key)
        if val != -1:
            self.l2.put(key, val)
            return val
        
        return -1
```

### Variation 5: Write-Through Cache
```python
class WriteThroughCache:
    """
    LRU cache that also writes to persistent storage.
    """
    def __init__(self, capacity, database):
        self.lru = LRUCache(capacity)
        self.db = database
    
    def get(self, key):
        val = self.lru.get(key)
        if val != -1:
            return val
        
        # Cache miss: load from DB
        val = self.db.get(key)
        if val is not None:
            self.lru.put(key, val)
        return val if val is not None else -1
    
    def put(self, key, value):
        # Write to cache
        self.lru.put(key, value)
        # Write through to database
        self.db.set(key, value)
```

### Variation 6: With Eviction Callback
```python
class LRUCacheWithCallback:
    """
    LRU cache that calls a callback function on eviction.
    Useful for cleanup, logging, or writing to disk.
    """
    def __init__(self, capacity, on_evict=None):
        self.cap = capacity
        self.cache = {}
        self.left = Node(0, 0)
        self.right = Node(0, 0)
        self.left.next = self.right
        self.right.prev = self.left
        self.on_evict = on_evict
    
    def evict_lru(self):
        lru = self.left.next
        if lru != self.right:
            self.remove(lru)
            del self.cache[lru.key]
            if self.on_evict:
                self.on_evict(lru.key, lru.val)
```

---

## Summary

**Key Takeaways:**

1. **Dual Data Structure Design:**
   - Hash map: O(1) lookup (key → node)
   - Doubly-linked list: O(1) insertion/deletion + ordering
   - Combined: O(1) get and put operations

2. **Doubly-Linked List is Essential:**
   - Need prev pointer for O(1) removal from middle
   - Singly-linked would require O(n) to find previous node
   - Enables moving accessed items to MRU position efficiently

3. **Dummy Nodes Pattern:**
   - Simplify edge cases (empty list, single item)
   - Eliminate null checks
   - left dummy: before LRU, right dummy: after MRU
   - Always valid: left.next and right.prev

4. **Store Key in Node:**
   - Needed for eviction: must delete from hash map
   - When evicting LRU, we have node but need key
   - Essential for maintaining consistency

5. **Access Updates Recency:**
   - get() moves item to MRU position
   - put() adds new item at MRU position
   - Eviction always from LRU position (left.next)

**Mental Model:**
```
Think of a line of people (queue):
  - Front (left): Least recently served
  - Back (right): Most recently served

When someone gets served (get/put):
  - Move them to the back (MRU)

When line is too long:
  - Remove person at front (LRU)

Hash map is like a register:
  - Quickly find someone by name
  - Points to their position in line
```

**Template (LRU Cache):**
```python
class Node:
    def __init__(self, key, val):
        self.key, self.val = key, val
        self.prev = self.next = None

class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.cache = {}
        self.left, self.right = Node(0, 0), Node(0, 0)
        self.left.next, self.right.prev = self.right, self.left
    
    def remove(self, node):
        prev, nxt = node.prev, node.next
        prev.next, nxt.prev = nxt, prev
    
    def insert(self, node):
        prev, nxt = self.right.prev, self.right
        prev.next = nxt.prev = node
        node.prev, node.next = prev, nxt
    
    def get(self, key):
        if key in self.cache:
            self.remove(self.cache[key])
            self.insert(self.cache[key])
            return self.cache[key].val
        return -1
    
    def put(self, key, value):
        if key in self.cache:
            self.remove(self.cache[key])
        self.cache[key] = Node(key, value)
        self.insert(self.cache[key])
        if len(self.cache) > self.cap:
            lru = self.left.next
            self.remove(lru)
            del self.cache[lru.key]
```

This is THE classic design problem showcasing how to combine data structures for optimal performance!
