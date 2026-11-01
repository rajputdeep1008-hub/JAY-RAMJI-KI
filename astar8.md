# 02. A* Search for the 8-Puzzle

## 1) Code

```python
import heapq

start = (2, 8, 3, 1, 6, 4, 7, 0, 5)
goal = (1, 2, 3, 8, 0, 4, 7, 6, 5)
moves = {"Up": (-1, 0), "Down": (1, 0), "Left": (0, -1), "Right": (0, 1)}


def show(state):
    for i in range(0, 9, 3):
        print(state[i:i + 3])
    print()


def heuristic(state):
    return sum(abs(i // 3 - goal.index(tile) // 3) + abs(i % 3 - goal.index(tile) % 3)
               for i, tile in enumerate(state) if tile)


def neighbors(state):
    blank = state.index(0)
    row, col = divmod(blank, 3)
    for move, (dr, dc) in moves.items():
        r, c = row + dr, col + dc
        if 0 <= r < 3 and 0 <= c < 3:
            other = r * 3 + c
            new = list(state)
            new[blank], new[other] = new[other], new[blank]
            yield tuple(new), move


def a_star(start, goal):
    queue = [(heuristic(start), 0, start)]
    costs, parents = {start: 0}, {start: (None, None)}
    visited = set()
    while queue:
        _, cost, state = heapq.heappop(queue)
        if state in visited:
            continue
        visited.add(state)
        if state == goal:
            path = []
            while state is not None:
                previous, move = parents[state]
                path.append((state, move))
                state = previous
            return path[::-1], len(visited)
        for nxt, move in neighbors(state):
            new_cost = cost + 1
            if nxt not in costs or new_cost < costs[nxt]:
                costs[nxt] = new_cost
                parents[nxt] = (state, move)
                heapq.heappush(queue, (new_cost + heuristic(nxt), new_cost, nxt))
    return None, len(visited)


print("=" * 50, "\n        A* ALGORITHM - 8 PUZZLE\n", "=" * 50, sep="")
print("\nInitial State:")
show(start)
print("Goal State:")
show(goal)
print("Searching for optimal solution...\n")

solution, explored = a_star(start, goal)
if solution:
    print("=" * 50, "\n             SOLUTION\n", "=" * 50, sep="")
    print(f"\nTotal moves: {len(solution) - 1}\nStates explored: {explored}\n")
    for step, (state, move) in enumerate(solution):
        print(f"Step {step}: " + ("Initial State" if step == 0 else f"Move Blank {move}"))
        show(state)
else:
    print("No solution exists.")
```

## 2) Theory to write

### Question

**Official syllabus: Part A, assignment 3.** Implement A* Search for 8 puzzle problem

### Aim

To solve the 8-puzzle using A* search with the Manhattan-distance heuristic.

### Objectives

1. Model puzzle configurations and legal moves.
2. Calculate `g(n)`, `h(n)` and `f(n)`.
3. Use a priority queue to explore promising configurations.
4. Recover and display a shortest sequence of moves for the given puzzle.

### Requirements

Python 3 and the standard-library `heapq` module. A valid start and goal must each contain exactly the numbers 0 through 8.

### Theory

The **8-puzzle** consists of eight numbered tiles and a blank in a 3 x 3 board. A move swaps the blank with an adjacent tile horizontally or vertically. The search seeks a path from an initial arrangement to a specified goal arrangement.

**A*** is an informed search algorithm. It ranks frontier states by:

`f(n) = g(n) + h(n)`

- `g(n)`: actual cost from start to `n`; here, the number of moves.
- `h(n)`: estimated remaining cost; here, total Manhattan displacement of numbered tiles.
- `f(n)`: estimated total solution cost through `n`.

For a tile, Manhattan distance is `abs(current_row - goal_row) + abs(current_col - goal_col)`. Add this for tiles 1 through 8; exclude the blank. One legal move moves exactly one numbered tile one grid step. Thus Manhattan distance does not overestimate the required moves (**admissible**) and satisfies `h(n) <= 1 + h(neighbor)` (**consistent**).

This implementation permanently closes a state when popped. Its shortest-path guarantee relies on a **consistent** heuristic, which the supplied Manhattan heuristic satisfies. With an inconsistent heuristic, a better implementation may need to reopen closed states. Do not say that every heuristic automatically makes this exact program optimal.

**Problem formulation:** initial state = supplied start tuple; state space = reachable valid boards; operators = legal blank moves; goal test = equality to goal tuple; step cost = 1.

### Algorithm

1. Put the start state in a min-priority queue with priority `h(start)` and cost 0.
2. Set the best known cost of start to 0 and its parent to a sentinel.
3. Remove the entry with minimum `f`.
4. Skip it if its state is already closed; otherwise close it.
5. If it is the goal, follow parent links back to start, reverse the path and return.
6. Generate all legal successors.
7. For each successor, calculate `new_g = g + 1`.
8. If unseen or cheaper than its saved cost, update its cost and parent, then insert it with priority `new_g + h(successor)`.
9. Repeat until success or the queue is empty.

### Observation / result

For the supplied board, `h(start) = 5`. The program finds a 5-move solution and reports 6 states explored, including the goal state that is popped and goal-tested.

### Conclusion

A* solves the given 8-puzzle optimally using actual path cost and a consistent Manhattan heuristic. Parent links recover the move sequence without storing a full path in every frontier entry.

## 3) Explaining the question, code and functioning

### Understanding the question

The question asks for a sequence of legal blank moves that changes the supplied initial board into the goal board. You must use A* priority selection, not just print the target or make greedy moves based only on Manhattan distance.

**Say this in the oral:** “I represent the 8-puzzle as a tuple with zero as the blank. A* expands the state with the smallest `f = g + h`, where `g` is moves made and `h` is Manhattan distance. I generate legal blank moves, save cheaper paths and parent links, then reconstruct the path when the goal is reached.”

Memory cue: **Represent -> Estimate -> Pop minimum f -> Generate -> Improve -> Reconstruct.**

### How the whole program runs

Python imports heapq, stores the puzzle/move data and defines the functions. The driver prints the start/goal with `show`, then calls `a_star`. That function repeatedly calls `neighbors` and `heuristic`. Once it returns a path and explored count, the driver prints the recovered boards in order.

### Representation and coordinates

`(2, 8, 3, 1, 6, 4, 7, 0, 5)` means the rows are `(2,8,3)`, `(1,6,4)`, `(7,0,5)`.

For flat index `i`: `row = i // 3`, `col = i % 3`. For coordinates `(r,c)`: `index = r * 3 + c`. For example, index 7 is row 2, column 1.

Tuples are immutable and hashable, so these integer tuples can be dictionary keys and set members. A list is temporarily used for swapping, then converted back to a tuple.

| Variable | Purpose |
| --- | --- |
| `queue` | Heap of `(f, g, state)` entries |
| `costs[state]` | Best path cost found so far |
| `parents[state]` | `(previous_state, incoming_blank_move)` |
| `visited` | Closed states already popped and processed |
| `path` | Reconstructed `(state, incoming_move)` pairs |

### Code in blocks

#### Block A: setup and display

Relevant code block:

```python
import heapq

start = (2, 8, 3, 1, 6, 4, 7, 0, 5)
goal = (1, 2, 3, 8, 0, 4, 7, 6, 5)
moves = {"Up": (-1, 0), "Down": (1, 0), "Left": (0, -1), "Right": (0, 1)}


def show(state):
    for i in range(0, 9, 3):
        print(state[i:i + 3])
    print()
```

`import heapq` loads heap operations. The dictionary `moves` maps each blank-move name to `(row_change, column_change)`. Negative row means Up. `state[i:i + 3]` slices out one row; `range(0, 9, 3)` gives 0, 3 and 6.

#### Block B: heuristic

Relevant code block:

```python
def heuristic(state):
    return sum(abs(i // 3 - goal.index(tile) // 3) + abs(i % 3 - goal.index(tile) % 3)
               for i, tile in enumerate(state) if tile)
```

`enumerate(state)` produces index-tile pairs. `goal.index(tile)` finds the tile's destination index. `abs` removes the sign of coordinate differences; `sum` adds all distances. `if tile` filters out zero because zero is false in a condition.

This is a generator expression. Expanded idea: loop over every numbered tile, calculate its row and column distances from the goal, add the distances to a total, and return the total.

**Heuristic is a function**, not a search algorithm by itself. Manhattan is one choice of heuristic; A* is the search algorithm that uses it.

#### Block C: generate legal successor boards

Relevant code block:

```python
def neighbors(state):
    blank = state.index(0)
    row, col = divmod(blank, 3)
    for move, (dr, dc) in moves.items():
        r, c = row + dr, col + dc
        if 0 <= r < 3 and 0 <= c < 3:
            other = r * 3 + c
            new = list(state)
            new[blank], new[other] = new[other], new[blank]
            yield tuple(new), move
```

`state.index(0)` locates the blank. `divmod(blank, 3)` returns quotient and remainder, so it gives row and column together. `.items()` yields dictionary key-value pairs. Tuple unpacking assigns the move name and `(dr,dc)`.

`0 <= r < 3` is a chained comparison that rejects out-of-board moves. The simultaneous assignment swaps two list entries. `yield` returns one successor at a time and pauses the generator; it does not finish the function like `return`.

#### Block D: frontier, costs, closing, and goal test

Relevant code block:

```python
def a_star(start, goal):
    queue = [(heuristic(start), 0, start)]
    costs, parents = {start: 0}, {start: (None, None)}
    visited = set()
    while queue:
        _, cost, state = heapq.heappop(queue)
        if state in visited:
            continue
        visited.add(state)
        if state == goal:
```

The queue starts as a one-element list, already a valid heap. `heappop` removes the smallest tuple: first compare `f`; ties compare `g`; remaining ties compare the state tuple. `_` receives the unused `f` value.

`continue` skips an already closed state. A state may have duplicate heap entries because Python's `heapq` is not using a decrease-key operation here. `visited.add` happens after popping, whereas the BFS programs mark states when enqueuing.

#### Block E: reconstruct the solution

Relevant code block:

```python
            path = []
            while state is not None:
                previous, move = parents[state]
                path.append((state, move))
                state = previous
            return path[::-1], len(visited)
```

At the goal, follow `parents[state]` backward. The start parent is `(None, None)`, so the loop ends after recording the start. `path[::-1]` reverses the list. Both the initial board and goal board are included; therefore moves = `len(path) - 1`.

#### Block F: relax edges and schedule successors

Relevant code block:

```python
        for nxt, move in neighbors(state):
            new_cost = cost + 1
            if nxt not in costs or new_cost < costs[nxt]:
                costs[nxt] = new_cost
                parents[nxt] = (state, move)
                heapq.heappush(queue, (new_cost + heuristic(nxt), new_cost, nxt))
    return None, len(visited)
```

`nxt not in costs or ...` uses short-circuiting: if the state is new, Python does not evaluate `costs[nxt]`, avoiding a missing-key error. If a cheaper route is found, record both its new cost and its parent. `heappush` restores heap order after insertion. Exhausting the queue returns `None` with the number of closed states.

#### Block G: driver and output

The corresponding program lines display start/goal, call `a_star`, and print each recovered state. `sep=""` controls separators between `print` arguments. `enumerate(solution)` numbers states from 0. The conditional expression prints “Initial State” for step 0 and a blank-movement label afterward.

### Dry run

Initial blank position is index 7, coordinate `(2,1)`. Three moves are legal:

| First move | New blank position | `g` | `h` | `f` |
| --- | --- | --- | --- | --- |
| Up | `(1,1)` | 1 | 4 | 5 |
| Left | `(2,0)` | 1 | 6 | 7 |
| Right | `(2,2)` | 1 | 6 | 7 |

A* chooses Up first. For this example the solution is:

| Step | Blank move | State, rows separated by `/` | `g` | `h` | `f` |
| --- | --- | --- | --- | --- | --- |
| 0 | Initial | `2 8 3 / 1 6 4 / 7 0 5` | 0 | 5 | 5 |
| 1 | Up | `2 8 3 / 1 0 4 / 7 6 5` | 1 | 4 | 5 |
| 2 | Up | `2 0 3 / 1 8 4 / 7 6 5` | 2 | 3 | 5 |
| 3 | Left | `0 2 3 / 1 8 4 / 7 6 5` | 3 | 2 | 5 |
| 4 | Down | `1 2 3 / 0 8 4 / 7 6 5` | 4 | 1 | 5 |
| 5 | Right | `1 2 3 / 8 0 4 / 7 6 5` | 5 | 0 | 5 |

The original output calls all 6 popped distinct states “explored”; only the 5 non-goal states have their successors generated.

### Complexity and changes to watch

In a general search tree A* may require exponential time and memory in solution depth. On a finite graph, a heap-based implementation with a consistent heuristic has the familiar `O((V + E) log V)` search bound when degree is bounded, excluding heuristic/representation overhead; memory includes costs, parents, closed states and heap entries. For the fixed 8-puzzle there are `9! / 2 = 181,440` reachable configurations from a valid start. Memory use, rather than one heuristic calculation, often becomes the practical limit.

`heuristic(state)` uses the **global** `goal`, even though `a_star(start, goal)` has a parameter of the same name. For your exam, edit the top-level `start` and `goal` constants together and call the existing function. If turning it into reusable code, pass the goal into the heuristic explicitly. Using a different function argument while leaving the global goal unchanged can invalidate its heuristic guarantees.

There is no input validation or inversion-parity precheck. For a 3 x 3 board, start and goal must have the same inversion parity when zero is ignored. A different but valid unsolvable arrangement can make the search explore the entire reachable component before failing. The supplied goal is not the standard `1..8,0` arrangement, so do not compare only with the standard goal's parity.

### Viva questions with short answers

1. **What is A*?** Informed best-first search using minimum `g + h`.
2. **What is `g` here?** Number of moves from the initial board.
3. **What is `h`?** Estimated moves remaining, calculated by Manhattan distance.
4. **Why exclude the blank?** A move shifts one numbered tile; counting the blank too can overestimate the required moves.
5. **What is admissibility?** The heuristic never exceeds the true cheapest remaining cost.
6. **What is consistency?** `h(n) <= cost(n,n') + h(n')` for every legal edge.
7. **Why is consistency relevant here?** Closed states are not reopened; consistent Manhattan distance makes that safe.
8. **A* versus greedy best-first?** A* uses `g+h`; greedy uses only `h`.
9. **What happens if `h=0`?** A* becomes uniform-cost search; with unit move costs it has BFS-like depth ordering.
10. **Why use `heapq`?** To efficiently remove the smallest-priority frontier entry.
11. **Why use tuples?** They are hashable, so these states can be stored in dictionaries and sets.
12. **Why use a list inside `neighbors`?** Lists can be modified for the tile swap; tuples cannot.
13. **What does `yield` do?** Produces one result and pauses the generator until iteration continues.
14. **Why store `parents`?** To reconstruct a solution path after finding the goal.
15. **Is every 8-puzzle solvable?** No; valid arrangements split into two parity classes.
16. **Why `len(solution)-1`?** The path contains the initial state before any move.
17. **Why might the same state appear in the heap twice?** A cheaper path inserts another entry instead of editing the old entry.
18. **Is Manhattan a search type?** No. It is a heuristic function used to guide a search type.

### Recreate the code from memory

Write state constants, Manhattan calculation, legal blank swaps, then the A* loop. Remember the three records: **heap for choice, costs for improvement, parents for path**. Add board printing last. To explain a heap entry, point to its three fields and say `f`, `g`, state.

### Keywords, functions and expressions to recognize

A keyword is reserved language syntax. Names such as `all`, `min`, `set`, `range` and `enumerate` are built-ins; they are not Python keywords.

| Expression | What it means here |
| --- | --- |
| `yield` | Keyword producing one neighbor and pausing the generator. |
| `continue` | Keyword skipping an already closed heap entry. |
| `not in`, `or` | Accept an unseen state or a cheaper known route; the test short-circuits. |
| `enumerate`, `sum`, `abs` | Built-ins used to calculate Manhattan distance; not keywords. |
| `//`, `%`, `divmod` | Convert a flat index to row and column. |
| `path[::-1]` | A reversed copy: parent traversal collected the path backward. |
