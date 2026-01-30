
This is the famous "Dutch National Flag" problem. The goal is to sort an array containing 0s (red), 1s (white), and 2s (blue) in-place. While you could just use a standard sort ($O(n \log n)$) or a two-pass counting sort ($O(n)$), the interviewer usually wants to see the one-pass, constant-space solution using Three Pointers.

The Strategy: Partitioning
We maintain three sections of the array using three pointers: left, right, and i (current).

Everything to the left of left is a 0.

Everything to the right of right is a 2.

Everything in between is being processed or contains 1s.

The Logic:

If nums[i] == 0: It belongs at the front. Swap nums[i] with nums[left] and increment both i and left.

If nums[i] == 2: It belongs at the back. Swap nums[i] with nums[right] and decrement right. Crucial: Do not increment i yet, because the value we just swapped from the back needs to be inspected.

If nums[i] == 1: It's already in the middle. Just increment i.


```python

class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """


        left , i = 0,0

        right = len(nums) -1

        while i <= right:
            if nums[i] == 0:

                nums[left], nums[i] = nums[i] , nums[left]
                left +=1
                i+=1
            elif nums[i] == 2:
                nums[i] , nums[right] = nums[right] , nums[i]
                right-=1
                # Note: We don't increment 'i' here because the new nums[i] 
            # might be a 0 that needs to be sent to the front.
            else:
                i+=1
        
        

```


Why don't we increment i when swapping with right?
When we swap nums[i] with nums[left], we know the value coming from left is either a 0 or a 1 (because i has already passed that area). However, when we swap nums[i] with nums[right], the value coming from right is unknown. It could be a 0, 1, or 2. We need to stay at index i for one more iteration to evaluate this new value and decide where it belongs.

