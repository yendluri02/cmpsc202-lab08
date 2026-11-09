# Lab 08: Navigating Weighted Graphs with Dijkstra's Algorithm

Name(s): _______________________  

## Introduction
You are already familiar with Breadth-First Search (BFS) and Depth-First Search (DFS) for exploring graphs. However, what happens when all edges are not created equal? In this lab, we will explore Dijkstra's Algorithm, which allows us to find the shortest path between two vertices in a *weighted* graph.

---

## Part 1: BFS with Weights

BFS is excellent for finding the shortest path in an unweighted graph. Let's see why it fails when weights are introduced. Consider the following weighted graph:

![Part 1 Graph](./part1.png)

**Questions:**
1. If you run a standard BFS starting at Node $A$, looking for Node $D$, which path will the BFS algorithm discover first? Why?
   > BFS will discover the direct path $A \rightarrow D$ first. BFS minimizes the number of edges, not the total weight, and $D$ is one edge away from $A$.

2. What is the total weight of the path BFS found? What is the total weight of the alternative path? 
   > The BFS path $A \rightarrow D$ has weight $100$. The alternative path $A \rightarrow B \rightarrow C \rightarrow D$ has weight $2+3+4=9$.

3. How does the definition of "shortest path" change when moving from unweighted graphs to weighted graphs?
   > In an unweighted graph, the shortest path is the path with the fewest edges. In a weighted graph, it is the path with the smallest sum of edge weights, even if it uses more edges.

---

## Part 2: Dijkstra's Algorithm

Dijkstra's algorithm uses a greedy approach. It maintains a set of "visited" (finalized) nodes and an array of the shortest known distances to all other nodes. Consider the following weighted graph:

![Part 2 Graph](./part2.png)

**Instructions:**
Starting at Node $A$, trace Dijkstra's algorithm. Fill in the tracker table below. 
*   **Current Node:** The unvisited node with the smallest current distance.
*   **Neighbors to Check:** The neighbors of the current node whose distances might need updating.
*   **`dist` Array:** The current known shortest distance from $A$ to each node. Use $\infty$ for unreachable/unknown nodes.
*   **`visited` Set:** The set of nodes that have been finalized and whose shortest distances are known.

| Step | Current Node | Neighbors to Check | `dist` Map | `visited` Set | 
| :--- | :--- | :--- | :--- | :--- | 
| 0 | $A$ | $B$, $C$ | $A$:0, $B$:4, $C$:2, $D$:$\infty$, $E$:$\infty$ | { } | 
| 1 | $C$ | $A$, $B$, $D$, $E$ | $A$:0, $B$:3, $C$:2, $D$:10, $E$:12 | {$C$} |
| 2 | $B$ | $A$, $C$, $D$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:12 | {$B$, $C$} |
| 3 | $D$ | $B$, $C$, $E$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:10 | {$B$, $C$, $D$} |
| 4 | $E$ | $C$, $D$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:10 | {$B$, $C$, $D$, $E$} |

**Conceptual Question:**
When we add a vertex to the "visited" set, why are we 100% certain we have found the absolute shortest path to that node? 
> All edge weights are nonnegative. The algorithm always chooses the unvisited node with the smallest tentative distance, so any other route to that node would have to pass through an unvisited node whose distance is already at least as large. Extending that route cannot make it cheaper. Therefore, the chosen distance is final.

---

## Part 3: Assumptions

Dijkstra's algorithm relies on an assumption about the weights. Let's see what happens when we break it. Consider the following weighted graph, which contains a negative edge:

![Part 3 Graph](./part3.png)

**Instructions:**
Trace Dijkstra's algorithm starting from Node $A$ to find the shortest path to Node $C$.

1. Which node does Dijkstra's algorithm visit first after $A$?
   > $B$, because its tentative distance is $2$, compared with $5$ for $C$.

2. What will Dijkstra's algorithm declare as the shortest path to $C$, and what is its weight?
   > It will update C through B and declare $A \rightarrow B \rightarrow C$ as the shortest path, with weight $2+(-4)=-2$.

3. Look at the graph manually. What is the *actual* shortest path from $A$ to $C$, and what is its total weight? 
   > The actual shortest path in this graph is also $A \rightarrow B \rightarrow C$, with total weight $-2$.

4. **The Trap:** Explain exactly *why* Dijkstra's algorithm failed to find the correct shortest path in this scenario.
   > It did not fail on this particular graph: the negative edge is relaxed before $C$ is finalized, so Dijkstra's algorithm finds the correct result by accident. However, the negative edge violates Dijkstra's nonnegative-weight assumption, so the algorithm is not generally reliable. For example, if a node were finalized before a later negative edge could improve its distance, Dijkstra would not revisit it and would return an incorrect answer. Bellman-Ford should be used when negative edge weights are possible.

---

## Part 4: Implementing Dijkstra's Algorithm

In the real world, problems rarely look like neat mathematical graphs right away. You often have to translate real-world constraints into a standard graph format so that an algorithm like Dijkstra's can process it.

**The Scenario:**
You are building a navigation app for delivery robots on a college campus. 

* Moving between any two adjacent sidewalk intersections takes exactly $2$ minutes.

* Some paths have stairs. The robots *cannot* use stairs.

* Certain intersections are busy crosswalks. The robot must wait an additional $3$ minutes at these specific intersections before it can safely leave them and continue on its path.

**The Problem:**
Standard Dijkstra's algorithm only understands *edges* and *edge weights*. It does not natively understand "node weights" (like waiting at an intersection) or "impassable terrain".

**Questions:**

1. How should you represent the paths with stairs in your graph so the algorithm never tries to use them?

   > Do not add edges for paths containing stairs. If an edge already exists, remove it or mark it unavailable before running Dijkstra's algorithm. The algorithm can then only choose walkable paths.

2. How can you account for the $3$-minute waiting time at busy crosswalk intersections using *only* edge weights? (Hint: Think about how you could modify the weights of edges connecting to that intersection).
   > Use directed edges and add the $3$-minute wait to every edge leaving a busy intersection. Thus, leaving that intersection costs $2+3=5$ minutes instead of $2$. This models the wait before the robot leaves the intersection while keeping the graph represented entirely with edge weights.