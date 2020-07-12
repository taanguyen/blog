---
layout: post
title:  "Hashmap Hopping? Think Graphs"
date:   2020-07-12 13:30:47 -0400
categories: leetcode graphs tech interviews
---
Let's have some coding fun, courtesy of LeetCode Challenge! 

Reconstruct Itinerary 

Given a list of airline tickets represented by pairs of departure and arrival airports `[from, to]`, reconstruct the itinerary in order. All of the tickets belong to a man who departs from `JFK`. Thus, the itinerary must begin with `JFK`.

**Note:**

1. If there are multiple valid itineraries, you should return the itinerary that has the smallest lexical order when read as a single string. For example, the itinerary `["JFK", "LGA"]` has a smaller lexical order than `["JFK", "LGB"]`.
2. All airports are represented by three capital letters (IATA code).
3. You may assume all tickets form at least one valid itinerary.
4. One must use all the tickets once and only once.

**Example 1:**

```
Input: [["MUC", "LHR"], ["JFK", "MUC"], ["SFO", "SJC"], ["LHR", "SFO"]]
Output: ["JFK", "MUC", "LHR", "SFO", "SJC"]
```

**Example 2:**

```
Input: [["JFK","SFO"],["JFK","ATL"],["SFO","ATL"],["ATL","JFK"],["ATL","SFO"]]
Output: ["JFK","ATL","JFK","SFO","ATL","SFO"]
Explanation: Another possible reconstruction is ["JFK","SFO","ATL","JFK","ATL","SFO"].
             But it is larger in lexical order.
```

What are the input(s)?

- a list of flights in the form of [from, to] 

What are the output(s)?

- a list of airports representing a valid itinerary using each ticket exactly once 
- a valid itinerary:
  - begins with JFK
  - uses each ticket exactly once
  - if multiple itineraries are possible, then return the itinerary with smaller lexical order

What data structures can model this problem?

- a hashmap
- a graph

## Hashmap approach

If we use a hashmap, we'll link each **from** airport with its corresponding **to** airports. 

For input

[["JFK","SFO"],["JFK","ATL"],["SFO","ATL"],["ATL","JFK"],["ATL","SFO"]]

We'd make a hashmap:

{"JFK": ["SFO", "ATL"], "SFO": ["ATL"], "ATL": ["SFO", "JFK"]}. 

We can then start at "JFK" and travel to "ATL". Then fly from "ATL" to "JFK" and so on. Seems like a reasonable approach, but what could go wrong?

<u>Accidental cycling:</u> The problem states we use each ticket once and only once. If we're not careful as we are hashmap hopping, we could get stuck in a loop, bouncing from one city to another and back again.

In the example above, we could fly 

"JFK" to "ATL" to "JFK" to "ATL" ... oops. 

So to prevent revisiting the same city, we could keep a visited set. But ooph! We do want to revisit cities since a valid itinerary can have repeats, ie: ["JFK","SFO","ATL","JFK","ATL","SFO"]. 

<u>Early termination</u>: Another issue is that we must use each ticket for a valid itinerary. But hashmap hopping can lead to early stops. 

Consider the input: [["JFK","KUL"],["JFK","NRT"],["NRT","JFK"]]

We'll have "JFK", "KUL" ...and then...nothing. Welppp. T_T

## When we're trying to do hashmap hopping, it's a good sign we're looking at a graph problem

Let's model this problem as a graph. We treat: 

- each airport as a vertex
- each ticket as a directed edge connecting two airports           

```python
# build an adjacency list, storing each [from, to]
adj = defaultdict(list)
for src,dest in tickets:
    adj[src].append(dest)
```

Now that we've built out our adjacency list, we can lexically sort the airports. I sorted **to** airports in reverse lexical order, you'll see why in a bit :)

```python
for airport in adj: 
	adj[airport] = sorted(adj[airport], reverse=True)
```

We need a list to store our itinerary. 

```python
itinerary = []
```

We're asked to perform a traversal in which we travel each edge exactly once. This can be done using a classic DFS approach, with one small tweak. Instead of maintaining a visited set of airports, we remove an edge (a flight) once we have traveled that edge. That way, we use each ticket exactly once. 

A vanilla DFS template looks like:

```
def dfs(vertex):
    visited.add(vertex)
    for neighbor in adj_list[vertex]:
        if neighbor not in visited:
            dfs(neighbor)
```

For our problem, once we visit a "to" airport , we remove the edge [from, to]

```python
def dfs(airport):
    while adj_list[airport]:
        destination = adj_list[airport].pop()	# airports are sorted in reverse order	
        dfs(destination)
```

Since we need to return an itinerary, we'll need to store the airports being visited. But *when* should we store the airport? Our first instinct is to store an airport as soon as we've visited it. Let's give it a try. 

```python
def dfs(airport):
    itinerary.append(airport)
    while adj_list[airport]:
        destination = adj.list[airport].pop()		
        dfs(destination)
```

Seems good! Except that we won't get the correct ordering. :sweat_smile: 

Again, consider the input: [["JFK","KUL"],["JFK","NRT"],["NRT","JFK"]]

If we append an airport as soon as we visit it, we'll get the itinerary ["JFK","KUL","NRT","JFK"]  when it should be ["JFK","NRT","JFK","KUL"].

Notice that "KUL" has no adjacent airports (a sink). It must be last on the itinerary. 

To figure out when we should append an airport, let's check out the call stack of after dfs is called. 

dfs (vertex) | dfs (neighbor of vertex) | dfs (neighbor of neighbor of vertex) | dfs (some sink vertex)

So the dfs calls are returned in the opposite order in which they are called:

dfs (some sink vertex) --------------------------------------- KUL     

dfs (neighbor of neighbor of vertex) ------------------- JFK       

dfs (neighbor of vertex) ------------------------------------ NRT      

dfs (vertex) ------------------------------------------------------ JFK        

Knowing this, we can append an airport after visiting its neighbors and reverse the itinerary for our final result. 

Our dfs function is: 	

```python
def dfs(airport):
	while adj_list[airport]:
        destination = adj_list[airport].pop()		
        dfs(destination)
    itinerary.append(airport)
```

And our final solution is: 

```python
class Solution:
    def findItinerary(self, tickets: List[List[str]]) -> List[str]:
        def dfs(airport):					
            while adj_list[airport]:	
                destination = adj_list[airport].pop()	# use each ticket exactly once
                dfs(destination)					
            itinerary.append(airport)				# add airport to itinerary
            
        itinerary = []
        adj_list = defaultdict(list)
        for src,dest in tickets:					# build a graph of airports
            adj_list[src].append(dest)
        for airport in adj: 
            adj_list[airport] = sorted(adj_list[airport], reverse = True)
            
        dfs("JFK")						# start at JFK and fly to neighboring airports 
        return reversed(itinerary)
```







