---
layout: post
title:  "Writing Recursive Functions: Using Arguments and the Return Statement"
date:   2020-07-24 16:00:00 -0400
categories: leetcode recursion trees
---
Let's consider a simple count function. 

```python
def count_down(num):
	if num < 1: return 
	print(str(num))
	count_down(num - 1)
```

If you call the function, say count_down(5), you'll get the printout:

```python
5
4
3
2
1
None
```

Liftoff! We have liftoff!! :rocket: 

Now that we're up and running, let's try and understand this code better. 

The function above is an example of a recursive function, a function that calls itself. Once called, a recursive function can run an indefinite number of times. In the code above, the number passed in to ` count_down` is printed and then called again with (number - 1). 

`count_down(5)` prints 5 then calls `count_down(4)`, which prints 4 and then calls `count_down(3)`, and so on. 

The `count_down` finishes once `num` has been whittled down to `0`. Conditions that terminate recursive calls are known as base cases.  

We could have achieved the same output with a while loop. In fact, a while loop would have saved stack space and probably executed faster too. So why and when is recursion useful?

## Why Use Recursion?

Recursion offers a natural way of breaking up a problem into smaller and smaller subproblems. 

Tree and graph algorithms often leverage recursion. As an example, consider the following function to count the number of nodes in a binary tree:

```python
def size(root):
	if not root: return 0	# base case: no root means no nodes in tree
	return 1 + size(root.left) + size(root.right)	# nodes in tree = nodes of left subtree + nodes of right subtree
```

Boom, two lines of code! Life doesn't get much better than that. haha :P 

Compare this clean and simple implementation with the iterative version:

```python
def size(root):
	count = 0
	queue = deque()
	queue.append(root)
    # level-order traversal 
	while queue:
		node = queue.popleft()
		count += 1 
		if node.left:
			queue.append(node.left)
		if node.right:
			queue.append(node.right)
	return count 
```

That's a mouthful in comparison. And you have to do several things manually- such as push, pop, and count each node. 

The trouble is, we're taught how to code iteratively first, with for-loops and while loops, so recursive thinking takes time getting used to. 

## Recipe for Recursion

Recursion behaves like a while loop. Recursive functions say "while you haven't reached the base case, repeat the same process with smaller and smaller input". 

Note: you can have bigger and bigger input, but I haven't personally come across a recursive function that grows the input

To write a recursive function, you must have two things:

- base case(s)
- recursive call(s) on smaller input 

A tricky thing about writing a recursive function is determining where information needs to go- as an argument or in the return statement. 

- <u>arguments</u> = stuff that gets **sent down to successive calls**
  - e.g. halving the input, a left/right subtree, a list to store paths from root to leaf, etc. 
- <u>return statement</u> = stuff that gets **passed up to preceding calls**
  - example uses: 
    - True/False - for determining if a value exists in a tree
    - Integer - for determining the size of a tree 

What information do you want the next call to see? Put that information in arguments. 

What information do you want the previous call to see? Put that information in the return statement.

## Example: Sending Stuff as Arguments

If you want to return all paths from root to leaf in a binary tree, you would need something to store nodes as you visit them (e.g. a list). You could pass in a second list as an argument to store all root to leaf paths. Whenever you reach a leaf, you'll add the current `path ` to an `all_paths` list. 

```python
 def binaryTreePaths(self, root):
        def find_paths(root, path, all_paths):
            if not root: return 
            # add node to current path 
            path.append(str(root.val))
        	# reached a leaf, add current path to all_paths
            if not root.left and not root.right:
                all_paths.append("->".join(path))
            # keep drilling down tree
            find_paths(root.left, path, all_paths)
            find_paths(root.right, path, all_paths)
            # remove node from current path to store nodes in next path
            path.pop()
        
        path = []
        all_paths = []
        find_paths(root, path, all_paths)
        return all_paths
```

Breaking down this example, we see the base case happens when our root doesn't exist (an empty tree). 

```python
if not root: return		
```

Once we visit a node, we add the node to our current path.

```python
# add node to current path 
path.append(str(root.val))
```

Then we check if that node is a leaf. If it is, we add it to our list all_paths

```python
# reached a leaf, add current path to all_paths
if not root.left and not root.right:
	all_paths.append("->".join(path))
```

We're done processing the node, so we make recursive calls to its left and right subtrees. 

We want successive calls to:

- add nodes to our current `path` and 
- add root-to-leaf paths to `all_paths`

so `path` and `all_paths` are **arguments**

```python
 # keep drilling down tree
 find_paths(root.left, path, all_paths)		# successive calls "see" path and all_paths
 find_paths(root.right, path, all_paths)
```

We're done processing left and right subtrees, so we've visited all paths that include this node. 

We'll remove this node from our current path to set up for the next path. 

```python
path.pop()
```

## Example: Sending Stuff in the Return Statement

If you want to check whether or not a binary tree is height-balanced, you need to find the height of the left and right subtrees and compare them. 

Skim the code below. 

```python
def isBalanced(self, root: TreeNode) -> bool:
    def height(root):
        # base case: empty tree
        if root == None: return -1
        # find heights of subtrees
        leftHeight = height(root.left)
        rightHeight = height(root.right)
        # check if tree is height-balanced 
        if abs(leftHeight - rightHeight) > 1: 
        # record imbalanced tree
            self.balanced = False
        # report back height of tree (to preceding calls)
    	return max(leftHeight, rightHeight) + 1

self.balanced = True
height(root)
return self.balanced
```

Each time we visit a node, we determine the height of the tree rooted at that node by finding `max(leftHeight, rightHeight) + 1`. 

Each `root` node relies on its subtrees - `root.left` and `root.right` - to determine its height. So each recursive call - `height(root.left)` and `height(root.right)`- must "report back" a.k.a. **return** their respective heights. 

To remember if a tree is balanced, we have a flag `self.balanced`. When you want to maintain a record of something across all recursive calls, use a variable (that's not local to the recursive function). 

## Bonus Challenge

Recap: 

- <u>S</u>end stuff down to <u>S</u>uccessive calls using arguments
- <u>P</u>ass stuff up to <u>P</u>receding calls in the return statement

Now that we're familiar with this "send-successive, pass-preceding" idea, try to crack [Binary Tree Pruning](https://leetcode.com/problems/binary-tree-pruning/)

You got this! :muscle:





