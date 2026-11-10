# Lab 08: Navigating Weighted Graphs with Dijkstra's Algorithm

Name(s): Samuel Roque

## Introduction
You are already familiar with Breadth-First Search (BFS) and Depth-First Search (DFS) for exploring graphs. However, what happens when all edges are not created equal? In this lab, we will explore Dijkstra's Algorithm, which allows us to find the shortest path between two vertices in a *weighted* graph.

---

## Part 1: BFS with Weights

BFS is excellent for finding the shortest path in an unweighted graph. Let's see why it fails when weights are introduced. Consider the following weighted graph:

![Part 1 Graph](./part1.png)

**Questions:**
1. If you run a standard BFS starting at Node $A$, looking for Node $D$, which path will the BFS algorithm discover first? Why?
   > BFS will discover the path $A \rightarrow D$ first. Standard BFS looks at the number of edges in a path instead of the weights of those edges. Since $D$ is directly connected to $A$, BFS reaches it using only one edge before exploring the longer path through $B$ and $C$.

2. What is the total weight of the path BFS found? What is the total weight of the alternative path? 
   > The path found by BFS is $A \rightarrow D$, which has a total weight of $100$. The alternative path is $A \rightarrow B \rightarrow C \rightarrow D$, which has a total weight of $2 + 3 + 4 = 9$. Even though the alternative path uses more edges, its total weight is much smaller.

3. How does the definition of "shortest path" change when moving from unweighted graphs to weighted graphs?
   > In an unweighted graph, the shortest path is the path that uses the fewest edges. In a weighted graph, the shortest path is the path with the smallest total edge weight. This means a path with more edges can still be shorter if those edges have smaller weights.

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
| 1 | $C$ | $B$, $D$, $E$ | $A$:0, $B$:3, $C$:2, $D$:10, $E$:12 | {$A$} |
| 2 | $B$ | $D$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:12 | {$A$, $C$} |
| 3 | $D$ | $E$ | $A$:0, $B$:3, $C$:2, $D$:8, $E$:10 | {$A$, $C$, $B$} |
| 4 | $E$ | None | $A$:0, $B$:3, $C$:2, $D$:8, $E$:10 | {$A$, $C$, $B$, $D$} |

**Conceptual Question:**
When we add a vertex to the "visited" set, why are we 100% certain we have found the absolute shortest path to that node?
> When a vertex is added to the `visited` set, it has the smallest known distance out of every unvisited vertex. Since Dijkstra's algorithm assumes all edge weights are non-negative, taking another path through an unvisited vertex cannot later make this distance smaller. Because of this, the current shortest distance can be finalized when the vertex is visited.

---

## Part 3: Assumptions

Dijkstra's algorithm relies on an assumption about the weights. Let's see what happens when we break it. Consider the following weighted graph, which contains a negative edge:

![Part 3 Graph](./part3.png)

**Instructions:**
Trace Dijkstra's algorithm starting from Node $A$ to find the shortest path to Node $C$.

1. Which node does Dijkstra's algorithm visit first after $A$?
   > Dijkstra's algorithm visits $B$ first after $A$. From $A$, the known distance to $B$ is 2 and the known distance to $C$ is 5. Since 2 is smaller than 5, $B$ is chosen next.

2. What will Dijkstra's algorithm declare as the shortest path to $C$, and what is its weight?
   > Dijkstra's algorithm will declare $A \rightarrow C$ as the shortest path to $C$, with a weight of $5$.

3. Look at the graph manually. What is the *actual* shortest path from $A$ to $C$, and what is its total weight? 
   > The actual shortest path is $A \rightarrow B \rightarrow C$. Its total weight is $2 + (-4) = -2$. Therefore, the actual shortest distance from $A$ to $C$ is $-2$.

4. **The Trap:** Explain exactly *why* Dijkstra's algorithm failed to find the correct shortest path in this scenario.
   > Dijkstra's algorithm fails because it assumes that all edge weights are non-negative. Once a node is finalized, the algorithm assumes that no path discovered later can give it a smaller distance. The negative edge from $B$ to $C$ breaks this assumption because going through $B$ can reduce the cost of reaching $C$. This is why Dijkstra's algorithm should not be used on graphs with negative edge weights.

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

   > I would leave the paths containing stairs out of the graph completely. If there is no edge representing a stair path, Dijkstra's algorithm cannot choose that path when finding a route. This represents the fact that the delivery robot cannot travel across stairs.

2. How can you account for the $3$-minute waiting time at busy crosswalk intersections using *only* edge weights? (Hint: Think about how you could modify the weights of edges connecting to that intersection).

   > I would add the 3-minute waiting time to the weights of the edges leaving each busy crosswalk intersection. A normal sidewalk takes 2 minutes, so traveling out of a busy intersection would have a cost of $2 + 3 = 5$ minutes instead of only 2 minutes. This converts the waiting time at the node into an edge cost that Dijkstra's algorithm can understand.
