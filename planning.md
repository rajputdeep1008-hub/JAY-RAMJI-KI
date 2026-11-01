# 05. BFS Planning with the 8-Puzzle

## 1) Code

```python
from collections import deque

start = (4, 1, 3,
         0, 2, 5,
         6, 7, 8)
goal = (1, 2, 3,
        4, 0, 5,
        6, 7, 8)

def moves(state):
    pos = state.index(0)
    r, c = divmod(pos, 3)
    for dr, dc in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
        nr, nc = r + dr, c + dc
        if 0 <= nr < 3 and 0 <= nc < 3:
            new = list(state)
            p = nr * 3 + nc
            new[pos], new[p] = new[p], new[pos]
            yield tuple(new)

def bfs():
    queue = deque([(start, 0)])
    parent = {start: None}
    levels = {}
    while queue:
        state, level = queue.popleft()
        levels.setdefault(level, []).append(state)
        if state == goal:
            path = []
            while state is not None:
                path.append(state)
                state = parent[state]
            return levels, path[::-1]
        for new_state in moves(state):
            if new_state not in parent:
                parent[new_state] = state
                queue.append((new_state, level + 1))
    return levels, []

def print_level(level, states):
    print(f"\nLEVEL {level}")
    for row in range(3):
        for state in states:
            values = ["_" if x == 0 else str(x)
                      for x in state[row * 3:row * 3 + 3]]
            print(" ".join(values), end="     ")
        print()

levels, path = bfs()
print("========== 8-PUZZLE BFS ==========")
for level, states in levels.items():
    print_level(level, states)
print("\n========== GOAL STATE ==========")
for row in range(3):
    values = ["_" if x == 0 else str(x)
              for x in goal[row * 3:row * 3 + 3]]
    print(" ".join(values))
print("\nSolution:", " -> ".join(str(state) for state in path) if path else "No solution found")
print("Moves:", len(path) - 1 if path else "N/A")
```

## 2) Theory to write

### Question

**Official syllabus: Part B, assignment 1.** Use BFS to solve a planning problem. Examples can be 8 puzzle problem, Robot path planning, Blocks world etc.

### Aim

To solve an 8-puzzle planning problem using breadth-first search and display a shortest sequence of states from the initial state to the goal.

### Objectives

1. Represent the puzzle as a search state with valid actions.
2. Explore configurations level by level using a FIFO queue.
3. Avoid repeated states by maintaining predecessor records.
4. Detect the goal and reconstruct a shortest solution path.

### Requirements

Python 3 and the standard-library `collections.deque` type. No external packages are needed.

### Theory

**Planning** finds an action sequence that transforms an initial state into a state satisfying a goal. In this practical, the problem is an 8-puzzle: eight numbered tiles and a blank occupy a 3 x 3 board. Each action swaps the blank with an adjacent tile horizontally or vertically. Zero represents the blank.

**Breadth-first search (BFS)** is an uninformed search algorithm. It processes states at depth 0, then depth 1, then depth 2, and so on. A FIFO queue removes the oldest inserted state first. No heuristic or estimate of the remaining distance is used.

The program stores each discovered state's predecessor in a `parent` dictionary. Its keys also act as the visited/discovered record, preventing repeated scheduling. When the goal is removed from the queue, parent links are followed backward and then reversed to obtain the solution path.

For this finite state space, BFS can find any reachable goal given enough memory. Because every move costs one, its returned path has the fewest moves. With unequal action costs, minimum depth need not mean minimum cost.

### Problem formulation

| Element | Representation |
| --- | --- |
| Initial state | `4 1 3 / 0 2 5 / 6 7 8` |
| Goal state | `1 2 3 / 4 0 5 / 6 7 8` |
| State | A flat tuple containing tiles 0 through 8 |
| Actions | Move the blank Up, Down, Left or Right within the board |
| Goal test | Current tuple equals the goal tuple |
| Step cost | One per legal blank move |

### Algorithm

1. Initialize the queue with the initial state at level 0.
2. Set `parent[start] = None` and create an empty dictionary for displayed levels.
3. Remove the oldest state and its level from the queue.
4. Add that state to the displayed list for its level.
5. If the state equals the goal, follow predecessor links until `None`, reverse the collected states and return the levels and path.
6. Otherwise locate the blank and generate valid neighboring boards by swapping it with an adjacent tile.
7. For each successor absent from `parent`, record its predecessor and enqueue it at the next level.
8. Repeat until the goal is found or the queue is empty.
9. Display the levels, goal, returned solution path and number of moves. If the queue exhausts without a goal, return an empty path and display failure.

### Observation / result

For the given start and goal, BFS returns four states connected by three blank moves: **Up -> Right -> Down**. The output reports `Moves: 3`.

`4 1 3 / 0 2 5 / 6 7 8`

`-> 0 1 3 / 4 2 5 / 6 7 8`

`-> 1 0 3 / 4 2 5 / 6 7 8`

`-> 1 2 3 / 4 0 5 / 6 7 8`

### Conclusion

The BFS implementation successfully detects the goal and reconstructs a shortest three-move solution for the supplied 8-puzzle. It demonstrates planning through state transitions, FIFO exploration and predecessor-based path recovery.


## 3) Explaining the question, code and functioning

### Understanding the question

The examiner is asking you to transform one puzzle arrangement into another using legal moves and BFS. Simply displaying the goal board would not solve the task. This program actually compares states with the goal and returns the path that reaches it.

**Say this in the oral:** “I use a FIFO queue to process puzzle states in increasing depth. My parent dictionary prevents duplicate discovery and remembers how each state was reached. When I pop the goal, I follow parents backward and reverse the sequence. All moves cost one, so the returned path uses the fewest moves.”

Memory cue: **Queue -> Pop oldest -> Goal? -> Discover neighbors -> Store parent -> Enqueue.**

### How the whole program runs

Python imports `deque`, stores the two board tuples and defines three functions. The line `levels, path = bfs()` starts the search. Inside it, `moves(state)` supplies legal successors. After finding the goal, `bfs` returns the level-display dictionary and the recovered path. The remaining code calls `print_level` and prints the final solution/count.

Defining a function does not execute its body. The top-level call to `bfs()` is what starts this search.

### Representation and important variables

A tuple stores the rows consecutively. `(4,1,3,0,2,5,6,7,8)` means rows `(4,1,3)`, `(0,2,5)` and `(6,7,8)`. It is not a tuple of three inner row tuples.

For flat index `pos`, `pos // 3` gives its row and `pos % 3` its column. Conversely, `(row,column)` becomes index `row*3+column`. For example, zero starts at index 3, coordinate `(1,0)`.

| Name | Purpose |
| --- | --- |
| `queue` | FIFO deque of `(state, level)` pairs waiting to be processed |
| `parent` | A state's predecessor, plus a discovered-state record through its keys |
| `levels` | Lists of states popped at each depth for display |
| `path` | Recovered start-to-goal board sequence |
| `new` | Mutable trial board used for a swap |

Integer tuples can be set members and dictionary keys because they are hashable. Lists are temporarily useful for changes but cannot be dictionary keys.

### Block A: legal puzzle moves

```python
def moves(state):
    pos = state.index(0)
    r, c = divmod(pos, 3)
    for dr, dc in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
        nr, nc = r + dr, c + dc
        if 0 <= nr < 3 and 0 <= nc < 3:
            new = list(state)
            p = nr * 3 + nc
            new[pos], new[p] = new[p], new[pos]
            yield tuple(new)
```

1. `state.index(0)` locates the blank's flat position.
2. `divmod(pos,3)` returns quotient and remainder together, giving row and column.
3. The four `(dr,dc)` pairs mean Up, Down, Left and Right.
4. `nr,nc = r+dr,c+dc` calculates a trial blank coordinate.
5. Chained comparisons keep coordinates within 0,1,2.
6. `list(state)` creates an editable board copy.
7. The simultaneous assignment swaps the blank with its neighboring tile.
8. `yield tuple(new)` produces one legal successor, then pauses until the caller asks for another.

For this start, moving Left is illegal because the blank is already in column 0. Up, Down and Right are legal. The direction labels describe the blank's movement; the adjacent tile moves the opposite way.

### Block B: initialize BFS

```python
def bfs():
    queue = deque([(start, 0)])
    parent = {start: None}
    levels = {}
```

`deque([(start,0)])` holds one queue entry. The outer list supplies the initial deque contents; `(start,0)` is a state-depth pair.

`parent = {start: None}` says the initial state has no predecessor. The special value `None` is the stopping marker for reconstruction. Dictionary membership checks keys, so `new_state not in parent` also checks whether that board has been discovered.

`levels = {}` creates an empty dictionary, not a set. An empty set would be `set()`.

### Block C: remove a state, record its level and detect the goal

```python
    while queue:
        state, level = queue.popleft()
        levels.setdefault(level, []).append(state)
        if state == goal:
            path = []
            while state is not None:
                path.append(state)
                state = parent[state]
            return levels, path[::-1]
```

`while queue` runs while the deque is nonempty. `popleft()` removes the oldest entry, giving FIFO order. A list's `pop()` would remove the newest item and change this behavior toward DFS.

`levels.setdefault(level,[])` gets the list for this level, creating it if missing. `.append(state)` records the popped board.

The equality check `state == goal` actually verifies success. At the goal, append each state and assign `state = parent[state]` to go backward. The start is appended too, then its parent is `None` and the loop stops. `path[::-1]` returns the states in forward order.

The reconstruction loop changes the local `state` variable; it does not alter the immutable board tuples or overwrite the `goal` constant.

### Block D: discover and enqueue new boards

```python
        for new_state in moves(state):
            if new_state not in parent:
                parent[new_state] = state
                queue.append((new_state, level + 1))
    return levels, []
```

For each legal successor, test whether it has a parent record already. New states receive the current board as predecessor and are appended at `level+1`. Marking discovery at enqueue time avoids having several queued copies of the same board.

If the queue becomes empty without reaching the goal, return the displayed levels and `[]`. This has the same two-value return shape as success, but an empty solution path means failure.

### Block E: display levels

```python
def print_level(level, states):
    print(f"\nLEVEL {level}")
    for row in range(3):
        for state in states:
            values = ["_" if x == 0 else str(x)
                      for x in state[row * 3:row * 3 + 3]]
            print(" ".join(values), end="     ")
        print()
```

The outer loop selects one row of a 3 x 3 board. The inner loop prints that row from every board in the level, placing boards side by side. A list comprehension converts zero to `"_"` and other values to strings. `join` requires string items. `end="     "` keeps the next board on the same console line; the final `print()` moves to the next board row.

The function's parameter called `states` is the list of boards for this one level, not the set of every possible puzzle configuration.

### Block F: call search and print its returned solution

```python
levels, path = bfs()
print("========== 8-PUZZLE BFS ==========")
for level, states in levels.items():
    print_level(level, states)
print("\n========== GOAL STATE ==========")
for row in range(3):
    values = ["_" if x == 0 else str(x)
              for x in goal[row * 3:row * 3 + 3]]
    print(" ".join(values))
print("\nSolution:", " -> ".join(str(state) for state in path) if path else "No solution found")
print("Moves:", len(path) - 1 if path else "N/A")
```

`levels, path = bfs()` unpacks the pair returned by the function. `.items()` iterates over level/list pairs. The separate goal display is followed by the **actual returned solution**, which is joined as tuple strings with arrows.

`X if condition else Y` is a conditional expression. For a nonempty path, print the board sequence and `len(path)-1`; otherwise print the no-solution message and `N/A`. The subtraction is necessary because the initial state is included before the first move.

### Dry run of the solution

| Depth | Blank move | Board, with `/` separating rows |
| --- | --- | --- |
| 0 | Initial | `4 1 3 / 0 2 5 / 6 7 8` |
| 1 | Up | `0 1 3 / 4 2 5 / 6 7 8` |
| 2 | Right | `1 0 3 / 4 2 5 / 6 7 8` |
| 3 | Down | `1 2 3 / 4 0 5 / 6 7 8` |

This run displays 1 popped board at depth 0, 3 at depth 1, 5 at depth 2 and **1 at depth 3**. The goal is the first popped board at depth 3, so the function returns immediately. That last list is only the processed part of the level; it is not the complete breadth of depth 3. Other boards may already be queued when the goal is found.

Four returned states mean three moves. The program returns states, not text direction labels; Up, Right, Down is the movement sequence inferred from consecutive blank positions.

### Complexity and limitations

For an explicit graph with constant-sized states, BFS takes `O(V+E)` time and `O(V)` memory, where V and E count the explored/discovered graph. For the fixed 3 x 3 puzzle, a valid start has 181,440 reachable configurations. BFS can therefore use substantial memory on difficult boards.

It is an uninformed algorithm: no Manhattan heuristic appears here. Every action costs one, which is why minimum depth equals minimum move cost. There is no input validation or inversion-parity precheck. Invalid boards can raise errors, and an unreachable valid goal can require exhausting the reachable component.

The function uses global `start` and `goal`. For a changed numerical slip, edit those two top-level tuples, retaining each integer 0 through 8 exactly once. Do not restore the older depth-limited display version.

### Viva questions with short answers

1. **What is BFS?** Search in increasing depth using a FIFO frontier.
2. **Why is the queue FIFO?** States discovered earlier must be processed before later, deeper states.
3. **Why use deque?** It supports efficient append and removal from the left.
4. **Does BFS use a heuristic?** No. It is uninformed search.
5. **What is a planning problem here?** Find actions that transform the start board into the goal board.
6. **Where does this code test the goal?** At `if state == goal` after removing the state from the queue.
7. **Why no separate visited set?** The keys of `parent` already record discovered states.
8. **Why mark discovery when enqueuing?** To avoid scheduling the same board more than once.
9. **Why store a parent?** To reconstruct the path once the goal is reached.
10. **What does the start's `None` parent mean?** It has no predecessor and ends reconstruction.
11. **Why reverse the collected path?** Parents are followed from goal backward to start.
12. **Why are moves one less than path length?** The path includes the initial board before any move.
13. **What solution does this input produce?** Up, Right, Down: three blank moves.
14. **Why only one board displayed at level 3?** The goal is popped first there and causes immediate return.
15. **Is BFS always minimum-cost?** Only when equal action costs make minimum depth correspond to minimum cost.
16. **What if actions have unequal costs?** Use uniform-cost search or a suitable A* implementation for minimum weighted cost.
17. **What does `yield` do in moves?** Produces a successor and pauses the generator until iteration continues.
18. **What signals failure?** An empty returned path; the driver prints no solution and `N/A` moves.

### Recreate the logic from memory

First write blank-neighbor generation. Then initialize `deque([(start,0)])` and `parent={start:None}`. Pop, test the goal, and enqueue only new boards with their parents. Reconstruct by following parents until `None` and reversing. Add level-print formatting after the search works.

### Keywords, functions and expressions to recognize

A keyword is reserved language syntax. Names such as `all`, `min`, `set`, `range` and `enumerate` are built-ins; they are not Python keywords.

| Expression | What it means here |
| --- | --- |
| `yield` | Produce a legal successor board without constructing one combined result list. |
| `while queue` | Continue while the deque is nonempty. |
| `state == goal` | Compare board values to detect success. |
| `state is not None` | Follow predecessors until the start sentinel, not until an empty board. |
| `not in parent` | Discover a board only once; dictionary membership checks keys. |
| `.setdefault(level, [])` | Create the depth list if missing, then append the popped state. |
