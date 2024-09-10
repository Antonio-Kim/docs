## Python Language

This document are small snippets on Python language itself. It does not deal with external libraries. This has no logical flow and should be treated carefully when reading.

### Data Structures

- List
  - Append(value), extend([values])
  - Pop()
  - del arr[5], del arr, del [4:6]
  - Insert(value, position)
  - Indexing: arr[5] for example. Negative value loops back to the list
  - Initializing an array: arr = [0] \* 5 (initialize array of 5 elements, all 0)
  - index(): find the index of the matching value. First one is returned only
  - Slicing: arr[1:3] (from index 1 to 2)
  - Unpacking: `a,b,c = [1,2,3]`. Left hand side must match right hand side
  - Looping: `for i in range(len(nums))` standard method, `for n in nums:` as foreach, `for i,n in enumerate(nums)` for index and value of list
  - sort()
  - sort(reverse=True)
  - List comprehension: `[i * i for i in range(5)]`
  - You can also initialize 2D array by `[[0] * 4 for i in range(4)]`
- String (Immutable)
  - You can append by += but this creates new string
  - Find ASCII value by ord()
  - Join list of strings by delimiter: `strings = ["ab", "cd", "ef"], print("".join(strings))`
- Queue
  - from collections import queue
  - queue = deque()
  - queue.append(1)
  - queue.popleft()
  - queue.pop()
  - queue.appendleft(1)
- HashSet
  - mySet = set()
  - mySet.add(1)
  - len(mySet)
  - 1 in mySet: check if 1 is in mySet
  - remove(1)
  - Set comprehension: mySet = { i for i in range(5) }
- HashMap

  - myMap = {} (initialize map without values)
  - myMap = { "alice" : 80, "Bob" : 70} (initialize map with values)
  - myMap["Alice"] = 80 (insert key-value to map)
  - "Alice" in myMap (check if key "Alice" is in map)
  - myMap.pop("Alice") - remove "Alice" from map
  - len(myMap) - get number of keys in HashMap
  - Dictionary comprehension: `myMap = { i: i*2 for i in range(3) }`
  - Map Iterations

    ```python
    myMap = { "Alice": 80, "Bob": 70 }
    # Typical method of iteration
    for key in myMap:
    	print(key, myMap[key])

    # Iterating only the values if keys are not needed
    for value in myMap.values():
    	print(value)

    # using both key and value
    for key, value in myMap.items():
    	print(key, value)
    ```

- Tuples (Immutable)

  - You'll most likely use Tuples for keys in HashMap or HashSet

  ```python
  myMap = {(1,2): 3}
  print(myMap[(1,2)])

  mySet = set()
  mySet.add((1,2))
  print((1,2) in mySet)
  ```

- Heaps (minHeap by defaut)

  - import heapq

  ```python
  import heapq

  # under the hood are arrays
  minHeap = []
  heapq.heappush(minHeap, 3)
  heapq.heappush(minHeap, 2)
  heapq.heappush(minHeap, 4)

  # min is always at index 0
  print(minHeap[0])

  while len(minHeap):
  print(heapq.heappop(minHeap))

  # To get maxHeap, we multiply each value by -1, and to access it
  # We do the same thing, but in reverse
  maxHeap = []
  heapq.heappush(maxHeap, -1 * 3)
  heapq.heappush(maxHeap, -1 * 2)
  heapq.heappush(maxHeap, -1 * 4)

  print(maxHeap[0] * -1)

  while len(maxHeap):
  print(-1 * heapq.heappop(maxHeap))
  ```

  - Convert list to heap: `arr = [2, 1, 8, 4, 5], heapq.heapify(arr)`

### Functions

- Common technique in functions is creating inner function. By default, inner functions have all access to outer function's variables

### Class

```python
class MyClass:
	# Constructor
	def __init__(self, nums):
		# Create member variables
		self.nums = nums
		self.size = len(nums)

	# self keyword required as params
	def getLength(self):
		return self.size

	def getdoubleLength(self):
		return 2 * self.getLength()

```
