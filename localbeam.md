# 04. Local Beam Search for the Traveling Salesman Problem

## 1) Code

```python
import random

dist = [
    [0, 10, 15, 20, 25, 30],
    [10, 0, 35, 25, 20, 15],
    [15, 35, 0, 30, 10, 20],
    [20, 25, 30, 0, 15, 25],
    [25, 20, 10, 15, 0, 35],
    [30, 15, 20, 25, 35, 0]
]
n = 6
k = 2

def cost(path):
    return sum(dist[path[i]][path[i + 1]] for i in range(n - 1)) + dist[path[-1]][path[0]]

def neighbors(path):
    result = []
    for i in range(1, n):
        for j in range(i + 1, n):
            new = path[:]
            new[i], new[j] = new[j], new[i]
            result.append(tuple(new))
    return result

states = set()
while len(states) < k:
    tail = list(range(1, n))
    random.shuffle(tail)
    states.add(tuple([0] + tail))

visited = set(states)
best = min(states, key=cost)
for step in range(10):
    candidates = {route for state in states for route in neighbors(list(state)) if route not in visited}
    if not candidates:
        break
    visited.update(candidates)
    states = set(sorted(candidates, key=cost)[:k])
    step_best = min(states, key=cost)
    if cost(step_best) < cost(best):
        best = step_best
    print(f"\nStep {step + 1}")
    print("Route:", list(step_best) + [step_best[0]])
    print("Distance:", cost(step_best))

print("\n-------------------------")
print("Final Best Route:", list(best) + [best[0]])
print("Minimum Distance:", cost(best))
```

## 2) Theory to write

### Question

**Official syllabus: Part A, assignment 7.** Implement Local Beam Search for The Traveling Salesman Problem (TSP).

### Aim

To apply local beam search to find a low-cost tour for a Traveling Salesman Problem instance.

### Objectives

1. Represent a tour and a city-distance matrix.
2. Evaluate the full round-trip cost.
3. Generate neighbors through pairwise swaps.
4. Retain a fixed-width beam of promising tours and record the best tour found.

### Requirements

Python 3 and the standard-library `random` module. The supplied instance has 6 cities and beam width `k=2`.

### Theory

The **Traveling Salesman Problem** asks for a minimum-cost cycle visiting every city exactly once and returning to the start. A tour is an ordering of cities; the final return edge is part of its cost.

The optimization problem is NP-hard. Exact tour enumeration grows factorially, so heuristic methods such as local beam search trade an optimality guarantee for a limited search budget. This does not make the six-city example difficult to check exhaustively; its small size permits the separate verification used here.

**Local search** works with complete candidate solutions and tries to improve them using a neighborhood. It usually emphasizes solution quality instead of constructing a path from an initial world state to a goal world state.

**Local beam search** maintains `k` current states. Each generation considers successors of all current states together, then selects the best `k` successors. This pooling lets one promising tour contribute several members of the next beam. It differs from running `k` independent hill climbs, where each run retains its own separate successor.

Here the evaluation is the actual tour cost, to be minimized. It is not the A* `g+h` formula. The code fixes city 0 first to avoid equivalent rotations of a tour and generates neighbors by swapping two of the remaining positions.

This shortened version also stores a **global visited set containing every generated candidate**, including candidates not selected into the beam. That is an extra pruning rule, not a defining requirement of basic local beam search. It can prevent reconsideration of a previously discarded tour. The beam can move to a worse-cost generation; `best` separately preserves the best result seen.

### Algorithm

1. Define the city-distance matrix, `n=6`, and `k=2`.
2. Create `k` distinct random tours with city 0 fixed first.
3. Evaluate them and save the cheapest initial tour as `best`.
4. Repeat for at most ten generations.
5. For every beam tour, generate pair-swap neighbors of positions 1 through `n-1`.
6. Pool distinct neighbors that are not in `visited`.
7. Stop if no candidates remain; otherwise mark all candidates visited.
8. Sort candidates by tour cost and retain up to the best `k` tours.
9. Update `best` if the current generation has a cheaper tour.
10. Display the best tour found and its round-trip distance.

### Observation / result

An example best tour is `0 -> 1 -> 5 -> 2 -> 4 -> 3 -> 0`, with cost `10+15+20+10+15+20 = 90`. The starting tours are random, so the trace and selected tour can vary. The reported result is the best found in that run.

### Conclusion

The local beam-search variant finds a low-cost tour for the supplied TSP instance using a limited set of current candidates. Its result is the best found during the run, and global optimality is not guaranteed by this search method.

## 3) Explaining the question, code and functioning

### Understanding the question

The question asks for a short tour that visits every city once and returns to its start. Local beam search keeps several current candidates and chooses the next beam from their pooled successors.

**Say this in the oral:** “I keep two candidate tours. I generate neighbors by swapping two non-start cities, pool neighbors from both tours, and keep the two cheapest candidates. I record the best tour seen over ten iterations. This is a local beam-search variant and does not guarantee a global optimum.”

Memory cue: **k tours -> All their neighbors -> Pool -> Best k -> Repeat -> Best ever.**

### How the whole program runs

Python stores the matrix and parameters, defines `cost` and `neighbors`, then immediately generates random initial tours. The top-level generation loop calls `neighbors` to build candidates and calls `cost` to rank them. It prints each generation and finally the best tour seen.

### Representation

`dist[a][b]` is the cost from city `a` to city `b`. The supplied matrix is symmetric and has zero diagonal. A tour such as `(0,1,5,2,4,3)` contains each city once. The closing 0 is added only when printing; `cost` already includes the closing edge.

| Name | Meaning |
| --- | --- |
| `states` | Current beam; initially two distinct random tours |
| `candidates` | Pooled eligible neighbors of all beam tours |
| `visited` | All generated/initial tours marked by this variant |
| `step_best` | Cheapest member of the current beam |
| `best` | Cheapest tour encountered across the search |

### Code in blocks

#### Block A: matrix and parameters

The corresponding program lines import `random`, define the 6 x 6 distance matrix, and set `n=6`, `k=2`. Indexing is zero-based. If the number of cities changes, the matrix and `n` must change together.

#### Block B: evaluate a tour

Relevant code block:

```python
def cost(path):
    return sum(dist[path[i]][path[i + 1]] for i in range(n - 1)) + dist[path[-1]][path[0]]
```

The generator adds consecutive edges for indices 0 through `n-2`. `path[-1]` is the last city; `path[0]` is the start. The final addition closes the cycle. Omitting it would solve a different problem, a visiting path rather than a tour.

#### Block C: generate a neighborhood

Relevant code block:

```python
def neighbors(path):
    result = []
    for i in range(1, n):
        for j in range(i + 1, n):
            new = path[:]
            new[i], new[j] = new[j], new[i]
            result.append(tuple(new))
    return result
```

The outer loop chooses one non-start position. The inner loop chooses a later position, so each pair is considered once. For `n=6`, there are `5 choose 2 = 10` swap-neighbors per tour.

`path[:]` makes a separate list; a swap must not change the current route. The simultaneous assignment exchanges the two cities. Converting to `tuple` makes each new route usable as a set member. These are **pair-swap neighbors**, not the segment reversal operation commonly called 2-opt.

#### Block D: choose starting tours

Relevant code block:

```python
states = set()
while len(states) < k:
    tail = list(range(1, n))
    random.shuffle(tail)
    states.add(tuple([0] + tail))

visited = set(states)
best = min(states, key=cost)
```

`set()` creates an empty set; `{}` would create an empty dictionary. `random.shuffle(tail)` changes the existing list in place. `[0] + tail` concatenates lists. The set removes duplicate tours, and the `while` loop continues until two distinct starts exist.

`min(states, key=cost)` compares states by the result of the `cost` function. `key=cost` passes the function itself; writing `key=cost()` would try to call it immediately without a tour argument.

#### Block E: pool and select the beam

Relevant code block:

```python
for step in range(10):
    candidates = {route for state in states for route in neighbors(list(state)) if route not in visited}
    if not candidates:
        break
    visited.update(candidates)
    states = set(sorted(candidates, key=cost)[:k])
    step_best = min(states, key=cost)
    if cost(step_best) < cost(best):
        best = step_best
    print(f"\nStep {step + 1}")
    print("Route:", list(step_best) + [step_best[0]])
    print("Distance:", cost(step_best))
```

The set comprehension means:

1. For each current beam state, generate its neighbors.
2. Keep only routes not already marked visited.
3. Collect distinct candidates in one shared set.

`visited.update(candidates)` adds every candidate, even those that the next line discards. `sorted(..., key=cost)[:k]` sorts by increasing distance and slices the first `k` entries. Converting them back to a set drops their order but not their selected membership. A tie can be resolved differently across implementations/set iteration details; do not promise one exact tour.

`if not candidates` tests whether the set is empty. `break` exits the generation loop. `best` is updated only for a strict cost improvement. A later, worse `step_best` never overwrites a better historical `best`.

#### Block F: final display

Relevant code block:

```python
print("\n-------------------------")
print("Final Best Route:", list(best) + [best[0]])
print("Minimum Distance:", cost(best))
```

`list(best) + [best[0]]` shows the return to the starting city. **“Minimum Distance” is the program's label for best found**, not evidence that the algorithm proved the global minimum.

### Tiny worked example

For `(0,1,5,2,4,3)`, the edges have costs 10, 15, 20, 10, 15 and 20, totaling 90.

For `(0,1,2,3,4,5)`, the cost is `10+35+30+15+35+30 = 155`. Swapping positions 2 and 5 gives `(0,1,5,3,4,2)`, cost `10+15+25+15+10+15 = 90`. This illustrates how one pair swap changes multiple edges while preserving one visit per city.

For a hypothetical candidate pool with costs `[120,95,110,90]` and `k=2`, retain the candidates with 90 and 95. The best tour seen so far may stay at 90 even if a later generation's best costs 100.

### Complexity and limitations

One tour has `O(n^2)` pair-swap neighbors. Copying and evaluating a tour cost `O(n)` each. For `k` beam members and `C=O(k*n^2)` distinct candidates, candidate generation is about `O(k*n^3)`, and ranking costs `O(C*n + C log C)` because Python evaluates each sort key once.

This exact code is **not** an `O(k)`-memory implementation: it materializes candidate tours and accumulates visited tours. If `G` tours have been generated, storing them takes `O(G*n)` representation space. With a fixed start there are at most `(n-1)!` tour orderings; symmetric reverse tours are still stored separately. Ten generations limit the actual run.

Random starting tours can change the trace and outcome. A fixed seed helps reproduce a demonstration but does not create an optimality guarantee. If `k` exceeds the number of possible distinct fixed-start tours, initialization cannot finish. The supplied `n=6, k=2` is valid. Reducing beam diversity can also limit exploration.

### Viva questions with short answers

1. **What is TSP?** Find a minimum-cost cycle visiting each city once and returning to the start.
2. **What is beam width?** The maximum number of current candidates retained; here 2.
3. **How are neighbors generated?** By swapping two non-start positions in a tour.
4. **Why keep city 0 fixed?** Equivalent rotations represent the same cycle, so fixing a start reduces redundancy.
5. **Does fixing city 0 remove the need to return to it?** No; the closing edge must still be counted.
6. **Why is `cost` added for last-to-first?** A TSP solution is a closed tour.
7. **Why not one independent successor per beam tour?** Local beam search pools all successors and globally chooses the best `k`.
8. **Beam search versus hill climbing?** Beam search keeps several candidates and shares their successor pool; hill climbing typically keeps one.
9. **Is this guaranteed optimal?** No. It may miss the globally shortest tour.
10. **Is the cost always improving each iteration?** No; only the historical `best` is guaranteed not to worsen.
11. **What does `key=cost` mean?** Compare/sort tours by their evaluated travel distance.
12. **Why store tours as tuples?** They can be members of a set.
13. **What is special about this visited set?** It marks all generated candidates, including ones discarded from the beam.
14. **What is the difference between `step_best` and `best`?** Best this generation versus best over the whole run.
15. **Are these 2-opt neighbors?** No; the code swaps positions, rather than reversing a route segment.
16. **Why can output vary?** Initial tours are random, and ties need not choose one unique tour.
17. **What is an evaluation function?** A function that scores a candidate; here lower tour cost is better.
18. **What is the verified optimum of this tiny instance?** 90, checked separately by enumerating all fixed-start tours.

### Recreate the code from memory

Write a cost function including return-to-start. Write pair-swap neighbors with a copy. Initialize two random tours, then repeatedly pool, rank and keep two. Keep a separate historical best. In the oral, say **best found** unless you also explain the independent exhaustive verification.

### Keywords, functions and expressions to recognize

A keyword is reserved language syntax. Names such as `all`, `min`, `set`, `range` and `enumerate` are built-ins; they are not Python keywords.

| Expression | What it means here |
| --- | --- |
| `while`, `for` | Keywords for distinct-tour initialization and generation/neighbor loops. |
| `set`, `tuple` | Built-in types used for distinct, hashable routes. |
| `key=cost` | Passes the cost function to min/sorted; key is a parameter name. |
| `[:k]` | Keep the first k entries of the sorted candidate list. |
| `path[:]` | Copy a flat route list before swapping. |
| `.update(candidates)` | Adds every candidate to the visited set, not only retained tours. |
