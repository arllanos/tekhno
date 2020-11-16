## General
### How are programs stored in memory
| TYPICAL MEMORY   |REPRESENTATION OF A PROGRAM| 
|--|--|
| For cmd line arguments | |
| Stack |the stack area contains the program stack, a LIFO structure. Stack **stores automatic and temporary variables for a called function** |
| Heap |the segment where **dynamic memory allocation** usually takes place |
| Uninitialized data |it contains all **global variables and static variables** that are initialized to zero or w/o explicit initialization in src code|
| Initialized data |it contains the **global variables and static variables** that are initialized by the programmer|
| Text segment |object file or in memory, which contains executable instruction like **for** or **if** statements |


### N largest values in dictionary
```python
import heapq
freq = {.....}  # freq = collections.Counter(arr)
k=2
# create heap and build nlargest output O(k log(k))
res = heapq.nlargest(k, freq, key=freq.get)
```
### Custom sort
```python
class Solution(object):
    def reorderLogFiles(self, logs):
        def f(log):
            id_, rest = log.split(" ", 1)
            return (0, rest, id_) if rest[0].isalpha() else (1,)

        return sorted(logs, key = f)
```
### 2D Array (python)
Initialize:
```python
rows, cols = 5,5
arr = [[0] * cols for _ in range(rows)]
```
Get dimensions:
```python
len(arr)  # num rows
len(arr[0])  # num columns
```
### 2D Array (java)
Initialize:
```java
rows = 5; cols = 5;
int[][] arr = new int[][];
for (int i = 0; i < rows; i++) {
	for (int j = 0; j < cols = 5; k++) {
		arr[i][j] = 0;
	}
}
```
Get dimensions:
```java
arr.length;  // num rows
arr[0].length;  // num columns
```
### Reverse string (python)
```python
myStr[::-1]
"".join(reversed(myStr))
```
### Reverse string (java)
```java
String inv = new StringBuilder(myStr).reverse().toString();
```
## Sorting and Searching
### Bubble sort --> O(n^2)
```python
def bubbleSort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
	for j in range(n-i-1):
	    if arr[j] > arr[j+1]:
	        arr[j], arr[j+1] = arr[j+1], arr[j]
            swapped = True
	 if not swapped:
		break
    return arr
```

## Trees
### BinaryTree traversals --> O(E + V) = O(n)
```python
class TreeNode:  
    def __init__(self, val=0, left=None, right=None):  
        self.val = val  
        self.left = left  
        self.right = right  

# dfs using recursion 
def dfs(root: TreeNode):  
    if not root: return  
    print(root.val, end=" ") # preorder dfs
    dfs(root.left)  
    dfs(root.right)  

# dfs using stack
def dfs(root: TreeNode):
	if not root: return
	stack = [root]
	while stack:
		root = stack.pop()
		print(root.val, end=" ")  # preorder dfs
		if root.right: stack.append(root.right)
		if root.left: stack.append(root.left)

# BFS / level order traversal 
def bfs(root: TreeNode):
	if not root: return
	
    q = deque([root])
    while q:  
        node = q.popleft()
        if node.left: q.append(node.left)
        if node.right: q.append(node.right)
        print(node.val, end=" ")
        
# split-level BFS traversal
def split_level_bfs(root: TreeNode) -> list:
	levels = []
	if not root: return levels

	q = deque([root])
	idx = 0
	  
	while q:
		levels.append([])  # initialize a new level
		lenq = len(q)  # num of nodes in the level
		
	    for i in range(lenq):
		    node = q.popleft()
	        levels[idx].append(node.val)
	        if node.left: q.append(node.left)
	        if node.right: q.append(node.right)

	    idx += 1
	  
	return levels
```
### N-ary Tree traversals --> O(E + V) = O(n)
```python
class TreeNode:
    def __init__(self, val=0):
        self.val = val
        self.children = []
  
def dfs(root):
    if not root.children: return
  
	for child in root.children:
        dfs(child)
        print(child.val, end=" ")
```
### Graph DFS
Time complexity: O(n) where n is the number of cells in the grid
Space complexity: O(n), the size of the implicit recursive call stack when calling `dfs`
```python
def dfs(grid: list, i: int, j: int, visited: list):
	is_valid = i >=0 and i < len(grid) and j >= 0 and j < len(grid[0])

	if not is_valid or visited[i][j] == 1:
		return
	visited[i][j] = 1
	dfs(grid, i-1, j, visited)
	dfs(grid, i+1, j, visited)
	dfs(grid, i, j+1, visited)
	dfs(grid, i, j-1, visited)
```
### BST = Binary Search Tree
- Average case:
	- Search --> O(log(n) -the height of the tree
	- Insert --> O(log(n))
- Worst case O(n)
```python
class Node(object):
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

class BST(object):
    def __init__(self, root):
        self.root = Node(root)

    def insert(self, new_val):
        self.insert_helper(self.root, new_val)

    def insert_helper(self, current, new_val):
        if new_val < current.value:
            if current.left is None:
                current.left = Node(new_val)
            else:
                self.insert_helper(current.left, new_val)
        elif new_val > current.value:
            if current.right is None:
                current.right = Node(new_val)
            else:
                self.insert_helper(current.right, new_val)
        else:
            raise ValueError

    def search(self, find_val):
        return self.search_helper(self.root, find_val)

    def search_helper(self, current, find_val):
        if current:
            if find_val == current.value:
                return True
            elif find_val < current.value:
                return self.search_helper(current.left, find_val)
            elif find_val > current.value:
                return self.search_helper(current.right, find_val)

        return False

    def print_tree(self):
        """Print out all tree nodes as they are visited in a pre-order traversal."""
        return self.print_helper(self.root, "")[:-1]

    def print_helper(self, current, traversal):
        """ Recursive print solution."""
        if current:
            traversal += str(current.value) + "-"
            traversal = self.print_helper(current.left, traversal)
            traversal = self.print_helper(current.right, traversal)
        return traversal
```
