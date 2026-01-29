

leetcode link : https://leetcode.com/problems/word-ladder/description/

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