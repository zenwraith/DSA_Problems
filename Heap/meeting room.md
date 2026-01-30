
```python

def canAttendMeetings(intervals: list[list[int]]) -> bool:
    # Sort by start time
    intervals.sort(key=lambda x: x[0])
    
    for i in range(1, len(intervals)):
        # If the current meeting starts before the last one ends
        if intervals[i][0] < intervals[i-1][1]:
            return False
            
    return True

```