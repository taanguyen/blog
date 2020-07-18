A month ago, I was on a video call with a few friends and we tried to crack the following problem. We flailed our brains around for 40 minutes before giving up. So let's revisit this tough cookie!!! :thumbsup:

Basic Calculator

Implement a basic calculator to evaluate a simple expression string.

The expression string may contain open `(` and closing parentheses `)`, the plus `+` or minus sign `-`, **non-negative** integers and empty spaces ``.

**Example 1:**

```
Input: "1 + 1"
Output: 2
```

**Example 2:**

```
Input: " 2-1 + 2 "
Output: 3
```

**Example 3:**

```
Input: "(1+(4+5+2)-3)+(6+8)"
Output: 23
```

**Note:**

- You may assume that the given expression is always valid.
- **Do not** use the `eval` built-in library function.

What are the input(s)?

- A string representing a mathematical expression consisting of non-negative integers being added or subtracted 

What are the output(s)?

- An integer that is the result of evaluating the expression

What data structures can model this problem?

- stacks 

## Why Stacks?

Stacks process data in a Last In First Out (LIFO) order...Huh?

Think of a stack of plates. When you wash the dishes, you'll remove the top plate. To add a plate, you'll add the plate to the top of the stack. The first plate in the stack is washed last and the last plate is washed first. Last in, first out. 

This LIFO property is useful for situations where we want to **process data and save intermediate results for later use.**

If you're still not sure why we're using a stack, we'll walk through a concrete example right now! 

Input: `(12 - ( 4 + 2 ))`

Expected output: 6

Our grade-school training prompts us to evaluate the innermost parenthetical expression (4 + 2) and then subtract from 12. 

Notice the Last In, First Out order in which we performed those steps:

- `(4 + 2)` is the last part of the string; `(4 + 2)` was evaluated first 
- `12 -` is the first part of the string; `12 -` was evaluated last

Now that we've settled on using a stack, let's see how we can use this stack to do math. 

## When we have to parse a string for expression evaluation, it's time to use a stack

We read our string from left to right, character by character. 

- we see `12 - (`
  - `1`, so our current number is 1
  - `2`, so our current number is 1 * 10 + 2 = 12
  - `-` sign, so we're done building our current number 
  - open `(` indicates the start of a new expression; we'll need to revisit `12 -` later
  - store `12` and `-` on the stack
- we see `4 + 2 )` 
  - `4`, so our current number is 4 
  - `+` sign, so we're about to see a new number; our current result is 4
  - `2`, so our current number is 2 
  - we've seen `4 + 2` so far, so our current result is 6
  - closing `)` indicates that we've reached the end of our current expression 
  - we need to combine our previous result `12 -` with our current result
    - previous result = 12 
    - current result = 6
    - 12 - 6 = 6 (desired output)

From this example, we can tease out how the stack responds to each element. 

The elements of our string are: `(`, `)`, `+`, `-`, and non-negative integers.

- A open `(` indicates the start of a new expression
  - push the current result and the operator (+/-) onto the stack
- A closing `)` indicates the end of our current expression
  - pop the previous operator and previous result from the stack
  - combine (+/-) the previous result with the current result 
- An integer indicates we're building our current number (multiplying by 10 and adding the integer)
- A `+` / `-` indicates the operation to perform between our current number and the current result 
  - ` + `: add the current number to our current result 
  - ` - `: subtract the current number from our current result
  - +/- : indicate we're about to build a new number 

Phew! We're done planning our solution. Onto the code! 

## Building our solution piece by piece 

Instantiate our current result, current number, sign (+/-), and stack. 

```python
res, num, sign, st = 0,0,1,[] 		# current result, current number, (+/-), stack
```

<u>The integer case:</u> 

Consecutive digits like "1" and "2" become 12. 

```python
# process string character by character
for char in s:
    # build number
    if char.isdigit():
        num = 10*num + int(char)	
```

<u>The (+/-) case:</u>

a - b is the same as a + (-b)

With input `12 - ( 4 + 2 )`, we'll eventually store (-1) in our stack. 

- `-` operator sets sign = -1 
- `+` operator sets sign = +1

```python
 # operation is about to happen
    elif char in "+-":
        res += sign * num           # add our current result to our current number 
        num = 0                     # reset num to build upcoming number
        sign = [-1,1][char =="+"]   # a - b = a + (-b)
```

<u>The open ( case:</u> 

Remember to reset our current result and sign back to their defaults of 0 and 1. 

```python
 # a new expression is about to happen
     elif char == "(":               
         st.append(res)             # store result up to this point  	
         st.append(sign)            # store the sign of the current expression
         res = 0                    # reset result for upcoming expression 
         sign = 1                   # reset sign for upcoming expression
```

<u>The closing ) case:</u>

Finish up evaluating the current expression. 

With input `12 - ( 4 + 2 )`, we get a current result of 6. 
Our stack has: | 12 | -1 | 

We pop (-1) and apply it to 6, getting (-6)  

We pop 12 and combine with (-6) to get an output of 6

```python
 # end current expression (expr) and combine with previous result 
    elif char == ")":
        res += sign * num           # result of current expression
        res *= st.pop()             # apply sign; -(expr) subtracts, +(expr) adds 
        res += st.pop()             # add to previous result 
        num = 0                     # reset num to build upcoming number  
                
```

## Solution

```python
def calculate(self, s: str) -> int:
        res, num, sign, st = 0,0,1,[]
        for char in s:
            if char.isdigit():
                num = 10*num + int(char)
            elif char in "+-":
                res += sign * num
                num = 0                      
                sign = [-1,1][char =="+"]          
            elif char == "(":               
                st.append(res)              
                st.append(sign)
                res = 0                    
                sign = 1                    
            elif char == ")":
                res += sign * num           
                res *= st.pop()             
                res += st.pop()             
                num = 0      
        return res + sign*num     # handles cases that don't end in ")", like 1 + 1
```

The time and space complexity are `O(n)`, as we process each character once and store intermediate results  in the stack.  

Now that you've learned how to use stacks for expression evaluation, try [Decode String](https://leetcode.com/problems/decode-string/). You got this! 