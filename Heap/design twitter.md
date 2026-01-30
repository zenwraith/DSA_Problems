# Design Twitter

**LeetCode 355** | **Difficulty:** Medium | **Category:** Heap, Design, Hash Table

---

## Problem Statement

Design a simplified version of Twitter where users can post tweets, follow/unfollow another user, and see the 10 most recent tweets in their news feed.

Implement the `Twitter` class:

- `Twitter()` Initializes your twitter object.
- `void postTweet(int userId, int tweetId)` Composes a new tweet with ID `tweetId` by the user `userId`. Each call to this function will be made with a unique `tweetId`.
- `List<Integer> getNewsFeed(int userId)` Retrieves the 10 most recent tweet IDs in the user's news feed. Each item in the news feed must be posted by users who the user followed or by the user themself. Tweets must be **ordered from most recent to least recent**.
- `void follow(int followerId, int followeeId)` The user with ID `followerId` started following the user with ID `followeeId`.
- `void unfollow(int followerId, int followeeId)` The user with ID `followerId` started unfollowing the user with ID `followeeId`.

---

## Examples

### Example 1:
```
Input:
["Twitter", "postTweet", "getNewsFeed", "follow", "postTweet", "getNewsFeed", "unfollow", "getNewsFeed"]
[[], [1, 5], [1], [1, 2], [2, 6], [1], [1, 2], [1]]

Output:
[null, null, [5], null, null, [6, 5], null, [5]]

Explanation:
Twitter twitter = new Twitter();
twitter.postTweet(1, 5);    // User 1 posts tweet 5
twitter.getNewsFeed(1);     // return [5] (User 1's own tweet)
twitter.follow(1, 2);       // User 1 follows User 2
twitter.postTweet(2, 6);    // User 2 posts tweet 6
twitter.getNewsFeed(1);     // return [6, 5] (tweet 6 from User 2, then 5 from User 1)
twitter.unfollow(1, 2);     // User 1 unfollows User 2
twitter.getNewsFeed(1);     // return [5] (only User 1's tweet)
```

### Example 2:
```
Input:
["Twitter", "postTweet", "postTweet", "postTweet", "getNewsFeed"]
[[], [1, 1], [1, 2], [1, 3], [1]]

Output:
[null, null, null, null, [3, 2, 1]]

Explanation:
User 1 posts three tweets. News feed returns them in reverse order (most recent first).
```

### Example 3:
```
Input:
["Twitter", "postTweet", "follow", "follow", "postTweet", "postTweet", "getNewsFeed"]
[[], [1, 5], [1, 2], [1, 3], [2, 6], [3, 7], [1]]

Output:
[null, null, null, null, null, null, [7, 6, 5]]

Explanation:
User 1 posts tweet 5, follows Users 2 and 3
User 2 posts tweet 6
User 3 posts tweet 7
News feed for User 1: [7, 6, 5] (most recent from all followed users)
```

---

## Strategy

### Merge K Sorted Lists with Min-Heap

**Key Insight:** Each user's tweets form a reverse-chronological list. Getting a news feed is like merging k sorted lists (where k = number of followed users).

**Why This Works:**

```
Problem breakdown:
1. Each user has tweets in chronological order
2. News feed needs most recent 10 from ALL followed users
3. Can't sort all tweets (too slow if many users/tweets)

Solution: Merge k sorted lists pattern
- Each followed user = one sorted list (newest to oldest)
- Use min-heap to efficiently find next most recent tweet
- Only explore what we need (stop at 10)

Data Structures:
1. tweetMap: userId → list of [timestamp, tweetId]
2. followMap: userId → set of followeeIds
3. Heap: [timestamp, tweetId, userId, index] for merging
```

**Algorithm Design:**

**postTweet(userId, tweetId):**
```python
1. Assign global timestamp (decreasing for min-heap)
2. Append [timestamp, tweetId] to user's tweet list
```

**getNewsFeed(userId):**
```python
1. Add userId to their own follow list (see own tweets)
2. Initialize heap with most recent tweet from each followed user
3. Pop heap 10 times (or until empty):
   - Add tweet to result
   - If that user has more tweets, add next one to heap
4. Return result
```

**follow(followerId, followeeId):**
```python
Add followeeId to followerId's follow set
```

**unfollow(followerId, followeeId):**
```python
Remove followeeId from followerId's follow set (if exists)
```

**Complexity:**
- **postTweet:** $O(1)$ - Append to list
- **getNewsFeed:** $O(k \log k + 10 \log k)$ where k = followed users
  - Initialize heap: O(k log k)
  - Extract 10: O(10 log k)
- **follow/unfollow:** $O(1)$ - Set operations
- **Space:** $O(U + T)$ - U users, T total tweets

---

## Solution

```python
import heapq
from collections import defaultdict

class Twitter:
    """
    Design Twitter with news feed functionality.
    
    Strategy: Merge k sorted lists using min-heap
    - Each user's tweets stored in chronological order
    - News feed merges tweets from all followed users
    - Heap efficiently finds next most recent tweet
    
    Data Structures:
    - tweetMap: userId → list of [timestamp, tweetId]
    - followMap: userId → set of followeeIds
    - count: Global timestamp (decreasing for min-heap)
    """

    def __init__(self):
        """Initialize Twitter object."""
        # Global timestamp (negative for min-heap to act as max-heap)
        self.count = 0
        
        # userId → list of [timestamp, tweetId]
        self.tweetMap = defaultdict(list)
        
        # userId → set of followeeIds
        self.followMap = defaultdict(set)

    def postTweet(self, userId: int, tweetId: int) -> None:
        """
        User posts a tweet.
        
        Store with timestamp for chronological ordering.
        Use negative count so min-heap behaves like max-heap.
        
        Time: O(1)
        """
        self.tweetMap[userId].append([self.count, tweetId])
        self.count -= 1

    def getNewsFeed(self, userId: int) -> List[int]:
        """
        Get 10 most recent tweets from followed users.
        
        Strategy: Merge k sorted lists
        1. User follows themselves (see own tweets)
        2. Initialize heap with most recent tweet from each followed user
        3. Pop heap up to 10 times, adding next tweet from same user
        
        Time: O(k log k + 10 log k) where k = followed users
        """
        res = []
        min_heap = []
        
        # User follows themselves
        self.followMap[userId].add(userId)
        
        # Initialize heap with most recent tweet from each followed user
        for followeeId in self.followMap[userId]:
            if followeeId in self.tweetMap:
                # Get most recent tweet (last in list)
                index = len(self.tweetMap[followeeId]) - 1
                count, tweetId = self.tweetMap[followeeId][index]
                
                # Push: [timestamp, tweetId, followeeId, next_index]
                # timestamp is negative, so min-heap gives most recent
                heapq.heappush(min_heap, [count, tweetId, followeeId, index - 1])
        
        # Extract up to 10 most recent tweets
        while min_heap and len(res) < 10:
            count, tweetId, followeeId, idx = heapq.heappop(min_heap)
            res.append(tweetId)
            
            # If this user has more tweets, add next one to heap
            if idx >= 0:
                count, tweetId = self.tweetMap[followeeId][idx]
                heapq.heappush(min_heap, [count, tweetId, followeeId, idx - 1])
        
        return res

    def follow(self, followerId: int, followeeId: int) -> None:
        """
        Follower follows followee.
        
        Time: O(1)
        """
        self.followMap[followerId].add(followeeId)

    def unfollow(self, followerId: int, followeeId: int) -> None:
        """
        Follower unfollows followee.
        
        Time: O(1)
        """
        if followeeId in self.followMap[followerId]:
            self.followMap[followerId].remove(followeeId)


# Usage
# obj = Twitter()
# obj.postTweet(userId, tweetId)
# param_2 = obj.getNewsFeed(userId)
# obj.follow(followerId, followeeId)
# obj.unfollow(followerId, followeeId)
```

---

## Detailed Walkthrough

### Example 1: Full Flow

**Initialize:**
```python
twitter = Twitter()
# count = 0
# tweetMap = {}
# followMap = {}
```

**Operation 1: postTweet(1, 5)**
```python
tweetMap[1] = [[0, 5]]
count = -1
```

**Operation 2: getNewsFeed(1)**
```python
Step 1: followMap[1].add(1) → followMap[1] = {1}

Step 2: Initialize heap
  User 1 has tweets: [[0, 5]]
  Most recent: index = 0, count = 0, tweetId = 5
  Heap: [[0, 5, 1, -1]]

Step 3: Extract tweets (limit 10)
  Pop: [0, 5, 1, -1]
  res = [5]
  idx = -1 (no more tweets)

Return: [5] ✓
```

**Operation 3: follow(1, 2)**
```python
followMap[1] = {1, 2}
```

**Operation 4: postTweet(2, 6)**
```python
tweetMap[2] = [[-1, 6]]
count = -2
```

**Operation 5: getNewsFeed(1)**
```python
Step 1: followMap[1] = {1, 2} (already includes 1)

Step 2: Initialize heap
  User 1: [[0, 5]], most recent at index 0
    Push: [0, 5, 1, -1]
  User 2: [[-1, 6]], most recent at index 0
    Push: [-1, 6, 2, -1]
  
  Heap: [[-1, 6, 2, -1], [0, 5, 1, -1]]
  (min-heap, so -1 < 0, tweet 6 is on top)

Step 3: Extract tweets
  Pop 1: [-1, 6, 2, -1]
    res = [6]
    idx = -1 (no more from user 2)
  
  Pop 2: [0, 5, 1, -1]
    res = [6, 5]
    idx = -1 (no more from user 1)

Return: [6, 5] ✓
```

**Operation 6: unfollow(1, 2)**
```python
followMap[1].remove(2) → followMap[1] = {1}
```

**Operation 7: getNewsFeed(1)**
```python
Step 1: followMap[1] = {1}

Step 2: Initialize heap
  User 1: [[0, 5]], most recent at index 0
    Push: [0, 5, 1, -1]
  
  Heap: [[0, 5, 1, -1]]

Step 3: Extract tweets
  Pop: [0, 5, 1, -1]
    res = [5]

Return: [5] ✓
```

---

### Example 2: Multiple Tweets from Same User

**Operations:**
```python
twitter = Twitter()
twitter.postTweet(1, 1)  # count = 0
twitter.postTweet(1, 2)  # count = -1
twitter.postTweet(1, 3)  # count = -2

# tweetMap[1] = [[0, 1], [-1, 2], [-2, 3]]
```

**getNewsFeed(1):**

| Step | Heap | Action | Result |
|------|------|--------|--------|
| Init | [[-2, 3, 1, 1]] | Most recent: index 2, count=-2, tweet=3 | - |
| 1 | [[-1, 2, 1, 0]] | Pop tweet 3, push next (index 1) | [3] |
| 2 | [[0, 1, 1, -1]] | Pop tweet 2, push next (index 0) | [3, 2] |
| 3 | [] | Pop tweet 1, no more (index -1) | [3, 2, 1] |

**Return:** `[3, 2, 1]` ✓ (reverse chronological)

---

### Example 3: Multiple Followed Users

**Setup:**
```python
twitter.postTweet(1, 5)    # User 1: [[0, 5]], count = -1
twitter.follow(1, 2)       # followMap[1] = {1, 2}
twitter.follow(1, 3)       # followMap[1] = {1, 2, 3}
twitter.postTweet(2, 6)    # User 2: [[-1, 6]], count = -2
twitter.postTweet(3, 7)    # User 3: [[-2, 7]], count = -3
```

**getNewsFeed(1):**

```
Initialize heap with most recent from each:
  User 1: [0, 5, 1, -1]
  User 2: [-1, 6, 2, -1]
  User 3: [-2, 7, 3, -1]

Heap (after heapify): [[-2, 7, 3, -1], [-1, 6, 2, -1], [0, 5, 1, -1]]

Extract:
  Pop: [-2, 7, 3, -1] → res = [7]
  Pop: [-1, 6, 2, -1] → res = [7, 6]
  Pop: [0, 5, 1, -1] → res = [7, 6, 5]
```

**Return:** `[7, 6, 5]` ✓

---

## Why This Algorithm Works

### The Merge K Sorted Lists Pattern

**Conceptual Model:**

```
Each user's tweets form a sorted list (newest to oldest):
  User 1: [tweet_10, tweet_7, tweet_3, tweet_1]  (timestamps: -10, -7, -3, -1)
  User 2: [tweet_9, tweet_5, tweet_2]            (timestamps: -9, -5, -2)
  User 3: [tweet_8, tweet_6, tweet_4]            (timestamps: -8, -6, -4)

Goal: Merge these lists to get 10 most recent overall

Brute Force: Collect all tweets, sort → O(T log T) where T = total tweets
Our Solution: Only explore what we need → O(k log k + 10 log k)

How? Use heap as priority queue:
1. Start with most recent from each list (3 tweets in heap)
2. Pop minimum (most recent due to negative timestamps)
3. Add next tweet from same list
4. Repeat until 10 tweets collected
```

**Why Negative Timestamps?**

```
Python's heapq is a min-heap (smallest on top)
We want most recent first (largest timestamp)

Solution: Use negative timestamps
  Actual time: 0, 1, 2, 3, 4, 5, 6
  Stored as:   0, -1, -2, -3, -4, -5, -6
  
  Min-heap of [-6, -5, -4] gives -6 first
  Which corresponds to time 6 (most recent)
  
This trick converts min-heap to max-heap behavior!
```

**Why Track Index?**

```
When we pop a tweet, we need to know:
1. Which user it came from (to access their tweet list)
2. Which tweet to add next from that user

Store in heap: [timestamp, tweetId, userId, nextIndex]

Example:
  User 1 tweets: [[0, 5], [-1, 4], [-2, 3]]
  
  Initial: [0, 5, 1, 1] (index 2, next is index 1)
  After pop: Add [-1, 4, 1, 0] (index 1, next is index 0)
  After pop: Add [-2, 3, 1, -1] (index 0, next is -1 = none)
```

### Efficiency Analysis

**Why Not Sort All Tweets?**

```
Sorting approach:
1. Collect all tweets from followed users: O(T)
2. Sort by timestamp: O(T log T)
3. Take first 10: O(1)
Total: O(T log T)

If users have many tweets, this is wasteful.

Heap approach:
1. Initialize heap (k followed users): O(k log k)
2. Extract 10 tweets: O(10 log k)
Total: O(k log k)

Much better when T >> 10 and k is small!
```

**Space Optimization:**

```
Heap size never exceeds k (number of followed users)
We only store one tweet per user in heap at a time
Even if users have 1000s of tweets, heap stays small
```

---

## Edge Cases

### 1. User Follows No One
```python
twitter = Twitter()
twitter.postTweet(1, 5)
twitter.getNewsFeed(1)
# Output: [5]
# User follows themselves by default in getNewsFeed
```

### 2. No Tweets Posted
```python
twitter = Twitter()
twitter.getNewsFeed(1)
# Output: []
# No tweets exist
```

### 3. Follow/Unfollow Same User Multiple Times
```python
twitter.follow(1, 2)
twitter.follow(1, 2)  # No duplicate (set)
twitter.unfollow(1, 2)
twitter.unfollow(1, 2)  # Safe (check before remove)
```

### 4. More Than 10 Tweets Available
```python
twitter.postTweet(1, 1)
twitter.postTweet(1, 2)
# ... 20 tweets total
twitter.getNewsFeed(1)
# Output: [20, 19, 18, ..., 11] (only 10 most recent)
```

### 5. User Unfollows Themselves
```python
twitter.postTweet(1, 5)
twitter.unfollow(1, 1)  # Try to unfollow self
twitter.getNewsFeed(1)
# Output: [5]
# User is re-added to followMap in getNewsFeed
```

### 6. Circular Follows
```python
twitter.follow(1, 2)
twitter.follow(2, 1)
# No issue, each user's news feed is independent
```

### 7. User with No Tweets
```python
twitter.follow(1, 2)  # User 2 hasn't posted
twitter.postTweet(1, 5)
twitter.getNewsFeed(1)
# Output: [5]
# User 2 is checked but has no tweets (tweetMap[2] empty)
```

---

## Common Mistakes

### ❌ Mistake 1: Not Following Self
```python
# Wrong! User doesn't see their own tweets
def getNewsFeed(self, userId):
    # Missing: self.followMap[userId].add(userId)
    for followeeId in self.followMap[userId]:
        # ...
```

**Fix:**
```python
def getNewsFeed(self, userId):
    self.followMap[userId].add(userId)  # Follow self
    # ...
```

### ❌ Mistake 2: Using Positive Timestamps
```python
# Wrong! Min-heap gives oldest first, not newest
self.count += 1  # Should be -=
```

**Fix:**
```python
self.count -= 1  # Negative for reverse order
```

### ❌ Mistake 3: Wrong Initial Index
```python
# Wrong! Starts from beginning (oldest)
index = 0
```

**Fix:**
```python
index = len(self.tweetMap[followeeId]) - 1  # Start from newest
```

### ❌ Mistake 4: Not Checking Index Bounds
```python
# Wrong! Crashes when idx < 0
count, tweetId = self.tweetMap[followeeId][idx]
heapq.heappush(min_heap, [count, tweetId, followeeId, idx - 1])
```

**Fix:**
```python
if idx >= 0:  # Check bounds first
    count, tweetId = self.tweetMap[followeeId][idx]
    heapq.heappush(min_heap, [count, tweetId, followeeId, idx - 1])
```

### ❌ Mistake 5: Not Checking User Has Tweets
```python
# Wrong! Crashes if user has no tweets
for followeeId in self.followMap[userId]:
    index = len(self.tweetMap[followeeId]) - 1  # Crashes if empty
```

**Fix:**
```python
for followeeId in self.followMap[userId]:
    if followeeId in self.tweetMap:  # Check first
        index = len(self.tweetMap[followeeId]) - 1
        # ...
```

### ❌ Mistake 6: Unfollow Without Checking
```python
# Wrong! Crashes if not following
def unfollow(self, followerId, followeeId):
    self.followMap[followerId].remove(followeeId)  # KeyError if not following
```

**Fix:**
```python
def unfollow(self, followerId, followeeId):
    if followeeId in self.followMap[followerId]:
        self.followMap[followerId].remove(followeeId)
```

---

## Interview Insights

**Question Recognition:**
- "Design" + "social media" + "news feed" → System design with heap
- "Most recent k items" from multiple sources → Merge k sorted lists

**Key Points to Mention:**

1. **Data structure choices:**
   - "I'll use hash maps for O(1) user/tweet lookups and sets for O(1) follow/unfollow operations."

2. **Merge pattern:**
   - "Getting a news feed is like merging k sorted lists. Each user's tweets are sorted, and I need the top 10 overall."

3. **Timestamp trick:**
   - "I use negative timestamps to make Python's min-heap behave like a max-heap for chronological ordering."

4. **Efficiency:**
   - "Instead of collecting and sorting all tweets (O(T log T)), I use a heap to only explore what I need (O(k log k + 10 log k))."

**Follow-up Questions:**

1. **Q:** "What if we need more than 10 tweets?"
   - **A:** Pass limit as parameter, change while condition to `len(res) < limit`.

2. **Q:** "How would you handle millions of tweets per user?"
   - **A:** Store tweets in database, paginate. Keep recent N tweets in memory, fetch older ones on demand.

3. **Q:** "What about retweets?"
   - **A:** Treat as new tweet with original timestamp, or add retweeterId field to distinguish.

4. **Q:** "How to handle deleted tweets?"
   - **A:** Add deletion set, check before adding to result. Or mark tweets as deleted in tweetMap.

5. **Q:** "What if we need to sort by engagement instead of time?"
   - **A:** Use engagement score instead of timestamp in heap. Recompute scores periodically.

6. **Q:** "Scale to millions of users?"
   - **A:** Shard users across servers, cache popular users' feeds, pre-compute feeds for active users.

**Complexity Analysis:**
- postTweet: O(1)
- getNewsFeed: O(k log k + 10 log k) where k = followed users
- follow/unfollow: O(1)
- Space: O(U + T) for U users and T tweets

---

## Related Problems

1. **Merge k Sorted Lists (LeetCode 23)** - Same merging pattern
2. **Design Hit Counter (LeetCode 362)** - Time-based design
3. **LRU Cache (LeetCode 146)** - Cache design with efficient operations
4. **Design Search Autocomplete System (LeetCode 642)** - Similar system design
5. **Design In-Memory File System (LeetCode 588)** - Complex data structure design
6. **Top K Frequent Elements (LeetCode 347)** - Heap for ranking

---

## Variations

### Variation 1: With Retweets
```python
class TwitterWithRetweets(Twitter):
    def __init__(self):
        super().__init__()
        self.retweetMap = defaultdict(list)  # tweetId → list of retweeters
    
    def retweet(self, userId: int, tweetId: int) -> None:
        """User retweets a tweet (appears in their timeline)."""
        # Find original tweet's timestamp
        # Add to user's timeline with same timestamp
        self.retweetMap[tweetId].append(userId)
        
        # For simplicity, treat as new tweet with current timestamp
        self.postTweet(userId, tweetId)
```

### Variation 2: With Likes/Ranking
```python
class TwitterWithLikes(Twitter):
    def __init__(self):
        super().__init__()
        self.likeMap = defaultdict(int)  # tweetId → like count
    
    def like(self, userId: int, tweetId: int) -> None:
        """User likes a tweet."""
        self.likeMap[tweetId] += 1
    
    def getNewsFeedByEngagement(self, userId: int) -> List[int]:
        """Get news feed sorted by engagement (likes + recency)."""
        res = []
        min_heap = []
        
        self.followMap[userId].add(userId)
        
        for followeeId in self.followMap[userId]:
            if followeeId in self.tweetMap:
                index = len(self.tweetMap[followeeId]) - 1
                count, tweetId = self.tweetMap[followeeId][index]
                
                # Score: combine timestamp and likes
                score = count - self.likeMap[tweetId] * 100  # Boost likes
                heapq.heappush(min_heap, [score, tweetId, followeeId, index - 1])
        
        while min_heap and len(res) < 10:
            score, tweetId, followeeId, idx = heapq.heappop(min_heap)
            res.append(tweetId)
            
            if idx >= 0:
                count, tweetId = self.tweetMap[followeeId][idx]
                score = count - self.likeMap[tweetId] * 100
                heapq.heappush(min_heap, [score, tweetId, followeeId, idx - 1])
        
        return res
```

### Variation 3: With Hashtags/Topics
```python
class TwitterWithHashtags(Twitter):
    def __init__(self):
        super().__init__()
        self.hashtagMap = defaultdict(list)  # hashtag → list of [count, tweetId, userId]
    
    def postTweetWithHashtags(self, userId: int, tweetId: int, hashtags: List[str]) -> None:
        """Post tweet with hashtags."""
        self.postTweet(userId, tweetId)
        
        # Index by hashtags
        count = self.count + 1  # Get current timestamp
        for tag in hashtags:
            self.hashtagMap[tag].append([count, tweetId, userId])
    
    def getNewsFeedByHashtag(self, hashtag: str, limit: int = 10) -> List[int]:
        """Get most recent tweets with specific hashtag."""
        if hashtag not in self.hashtagMap:
            return []
        
        # Sort by timestamp (already stored)
        tweets = sorted(self.hashtagMap[hashtag], key=lambda x: x[0])
        return [tweetId for _, tweetId, _ in tweets[:limit]]
```

### Variation 4: Persistent Timeline (Load More)
```python
class TwitterWithPagination(Twitter):
    def getNewsFeed(self, userId: int, lastSeenTimestamp: int = 0) -> List[int]:
        """
        Get news feed with pagination support.
        lastSeenTimestamp: Load tweets older than this.
        """
        res = []
        min_heap = []
        
        self.followMap[userId].add(userId)
        
        for followeeId in self.followMap[userId]:
            if followeeId in self.tweetMap:
                # Find first tweet older than lastSeenTimestamp
                tweets = self.tweetMap[followeeId]
                for i in range(len(tweets) - 1, -1, -1):
                    if tweets[i][0] < lastSeenTimestamp:
                        count, tweetId = tweets[i]
                        heapq.heappush(min_heap, [count, tweetId, followeeId, i - 1])
                        break
        
        while min_heap and len(res) < 10:
            count, tweetId, followeeId, idx = heapq.heappop(min_heap)
            res.append(tweetId)
            
            if idx >= 0:
                count, tweetId = self.tweetMap[followeeId][idx]
                heapq.heappush(min_heap, [count, tweetId, followeeId, idx - 1])
        
        return res
```

---

## Summary

**Key Takeaways:**

1. **Design Pattern: Merge K Sorted Lists**
   - Each user's tweets = sorted list (newest to oldest)
   - News feed = merge lists from followed users
   - Heap efficiently finds next most recent

2. **Data Structure Choices:**
   - Hash map for tweets: O(1) access per user
   - Set for follows: O(1) follow/unfollow
   - List for tweets: Chronological order maintained
   - Heap for merging: O(log k) per operation

3. **Timestamp Trick:**
   - Use negative timestamps
   - Converts min-heap to max-heap behavior
   - Simple and efficient

4. **Efficiency Optimization:**
   - Don't collect all tweets (wasteful)
   - Only explore what's needed (10 tweets)
   - Stop early when result is full

5. **Edge Case Handling:**
   - User follows themselves (see own tweets)
   - Check if user has tweets before accessing
   - Validate bounds before array access
   - Safe unfollow (check existence)

**Template:**
```python
class Twitter:
    def __init__(self):
        self.count = 0
        self.tweetMap = defaultdict(list)  # userId → [[timestamp, tweetId], ...]
        self.followMap = defaultdict(set)  # userId → {followeeIds}
    
    def postTweet(self, userId, tweetId):
        self.tweetMap[userId].append([self.count, tweetId])
        self.count -= 1
    
    def getNewsFeed(self, userId):
        res, min_heap = [], []
        self.followMap[userId].add(userId)
        
        # Initialize heap with most recent from each followed user
        for followeeId in self.followMap[userId]:
            if followeeId in self.tweetMap:
                idx = len(self.tweetMap[followeeId]) - 1
                count, tweetId = self.tweetMap[followeeId][idx]
                heapq.heappush(min_heap, [count, tweetId, followeeId, idx - 1])
        
        # Extract up to 10 tweets
        while min_heap and len(res) < 10:
            count, tweetId, followeeId, idx = heapq.heappop(min_heap)
            res.append(tweetId)
            
            if idx >= 0:
                count, tweetId = self.tweetMap[followeeId][idx]
                heapq.heappush(min_heap, [count, tweetId, followeeId, idx - 1])
        
        return res
    
    def follow(self, followerId, followeeId):
        self.followMap[followerId].add(followeeId)
    
    def unfollow(self, followerId, followeeId):
        if followeeId in self.followMap[followerId]:
            self.followMap[followerId].remove(followeeId)
```

**Mental Model:**
- Think of it as a multi-way merge
- Each followed user contributes a sorted stream
- Heap efficiently selects next item across all streams
- Classic pattern for "top k from multiple sources"
- This scales well with proper indexing and caching!
