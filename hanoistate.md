# 03. Tower of Hanoi by State-Space Search

## 1) Code

```python
from collections import deque

def successors(state):
    result = []
    pegs = [list(peg) for peg in state]
    for source in range(3):
        if not pegs[source]:
            continue
        disk = pegs[source][-1]
        for destination in range(3):
            if source == destination or (pegs[destination] and pegs[destination][-1] < disk):
                continue
            next_pegs = [peg[:] for peg in pegs]
            next_pegs[source].pop()
            next_pegs[destination].append(disk)
            result.append((tuple(map(tuple, next_pegs)), (source, destination)))
    return result

def state_space_search(disks):
    start = (tuple(range(disks, 0, -1)), (), ())
    goal = ((), (), tuple(range(disks, 0, -1)))
    frontier = deque([start])
    parent = {start: (None, None)}
    depth = {start: 0}
    explored = 0
    current_level = -1
    level_count = 0

    while frontier:
        state = frontier.popleft()
        explored += 1
        level = depth[state]
        if level != current_level:
            if current_level != -1:
                print(f"Total states at Level {current_level}: {level_count}\n")
            current_level = level
            level_count = 0
            print(f"{'=' * 15} LEVEL {level} {'=' * 15}")
        print(state)
        level_count += 1
        if state == goal:
            break
        for next_state, action in successors(state):
            if next_state not in parent:
                parent[next_state] = (state, action)
                depth[next_state] = depth[state] + 1
                frontier.append(next_state)

    if current_level != -1:
        print(f"Total states at Level {current_level}: {level_count}")

    if goal not in parent:
        return None, explored

    path = []
    state = goal
    while parent[state][0] is not None:
        previous, action = parent[state]
        path.append(action)
        state = previous
    return path[::-1], explored

disks = int(input("Enter number of disks: "))
moves, explored = state_space_search(disks)
print("\nStates explored:", explored)
print("\n========== SHORTEST SOLUTION ==========")
if moves is None:
    print("No solution found.")
else:
    for number, (source, destination) in enumerate(moves, 1):
        print(f"Move {number}: Peg {source} -> Peg {destination}")
    print("Total moves:", len(moves))
```

## 2) Theory to write

### Question

**Official syllabus: Part A, assignment 5.** Implement Tower of Hanoi by State Space Search

### Aim

To solve the Tower of Hanoi problem using state-space representation and breadth-first search.

### Objectives

1. Define initial and goal configurations of the three pegs.
2. Generate legal successor configurations.
3. Explore the state space with BFS without revisiting states.
4. Recover a shortest sequence of valid disk moves.

### Requirements

Python 3 and `collections.deque`. Use a small positive disk count, such as 3, for demonstration.

### Theory

Tower of Hanoi has three pegs and disks of different sizes. Initially all disks are on the source peg, largest at the bottom. The goal is to transfer all disks to the destination peg.

Rules: move one disk at a time; take only the top disk of a peg; never place a larger disk on a smaller disk.

A **state space** is a set of configurations connected by legal actions. A **state-space search** is an approach to solving a problem; it is not one single algorithm. This program chooses **BFS** as its search strategy. Each BFS level contains configurations whose shortest distance from the start is that level number.

For `n` disks on three pegs, there are `3^n` legal assignments of disks to pegs, since the order on each peg is forced by size. For standard three-peg Hanoi, the optimal number of moves is `2^n - 1`. The recurrence is `T(n) = 2T(n-1) + 1`: move `n-1` disks aside, move the largest disk, then move the smaller disks onto it.

The recurrence explains the minimum but **this code does not implement the classic recursive Hanoi routine**. It searches explicit configurations with a queue.

**Problem formulation:** initial state = all disks on peg 0; goal = all disks on peg 2; operators = legal top-disk transfers; unit step cost = one disk move.

### Algorithm

1. Form start `((n,...,1), (), ())` and goal `((), (), (n,...,1))`.
2. Enqueue start, record its parent sentinel and depth 0.
3. Remove the oldest state and display its level.
4. If it is the goal, stop.
5. For each nonempty source peg, select its top disk.
6. For each other peg, allow the move if it is empty or has a larger top disk.
7. Copy the pegs, apply the move, and convert the result to an immutable state.
8. For each undiscovered successor, save parent/action and depth, then enqueue it.
9. At the goal, follow parent links to collect moves; reverse and display them.

### Observation / result

For 3 disks, the program produces a legal solution of 7 moves. In the supplied iteration order it reports 25 popped states explored before stopping on the goal.

### Conclusion

The BFS state-space implementation solves the supplied Tower of Hanoi problem with a shortest sequence of legal unit-cost moves, demonstrating successor generation and path reconstruction.

## 3) Explaining the question, code and functioning

### Understanding the question

The question asks you to model disk configurations and search between them. Your code implements explicit BFS through the state space; it is different from printing the standard recursive Hanoi move sequence.

**Say this in the oral:** “I represent the three pegs as a tuple of tuples. A successor moves one top disk to a legal destination. BFS searches configurations level by level, and parent links reconstruct the shortest move sequence.”

Memory cue: **Top disk only -> Legal destination -> Copy -> BFS -> Backtrack parents.**

### How the whole program runs

Python defines `successors` and `state_space_search`. It reads a disk count, calls `state_space_search`, and processes its queue. Each expansion calls `successors`. The search reconstructs and returns moves; the final loop displays them. Defining these functions does not run their bodies until a call occurs.

### Representation: understand before memorizing

`((3,2,1), (), ())` has all disks on peg 0. Each inner tuple is **bottom to top**. Thus `peg[-1]` is the top disk, and disk 1 is smallest.

Example: `((3,2), (), (1,))` means disk 1 moved from peg 0 to peg 2. `(source,destination) = (0,2)` records that action. The program names pegs **0, 1, 2**, not 1, 2, 3.

| Variable | Purpose |
| --- | --- |
| `frontier` | FIFO queue of states waiting to be processed |
| `parent` | Discovered-state record and backward parent/action links |
| `depth` | Number of moves from start to a state |
| `explored` | Count of popped states, including goal |
| `current_level`, `level_count` | Display bookkeeping |

### Code in blocks

#### Block A: generate legal moves

Relevant code block:

```python
from collections import deque

def successors(state):
    result = []
    pegs = [list(peg) for peg in state]
    for source in range(3):
        if not pegs[source]:
            continue
        disk = pegs[source][-1]
        for destination in range(3):
            if source == destination or (pegs[destination] and pegs[destination][-1] < disk):
                continue
            next_pegs = [peg[:] for peg in pegs]
            next_pegs[source].pop()
            next_pegs[destination].append(disk)
            result.append((tuple(map(tuple, next_pegs)), (source, destination)))
    return result
```

- `[list(peg) for peg in state]` makes each peg mutable.
- `if not pegs[source]` detects an empty list; `continue` skips this source.
- `[-1]` selects the last, topmost disk.
- The rejection condition uses `or`: skip a same-peg move **or** a destination with a smaller top disk.
- `pegs[destination] and ...` short-circuits. Python checks the top disk only if the destination is nonempty, avoiding an index error.
- `[peg[:] for peg in pegs]` copies **every inner list**, so creating one successor does not damage the current state or another successor.
- `.pop()` removes the source top. `.append(disk)` pushes it onto the destination.
- `map(tuple, next_pegs)` converts each peg to a tuple; the outer `tuple(...)` freezes the three-peg collection. The result can be a dictionary key.
- Return entries contain `(new_state, action)`, so the search knows both where it can go and which move gets there.

#### Block B: initialize the search

Relevant code block:

```python
def state_space_search(disks):
    start = (tuple(range(disks, 0, -1)), (), ())
    goal = ((), (), tuple(range(disks, 0, -1)))
    frontier = deque([start])
    parent = {start: (None, None)}
    depth = {start: 0}
    explored = 0
    current_level = -1
    level_count = 0
```

`range(disks, 0, -1)` counts down from `disks` to 1. `()` is an empty tuple. `deque([start])` initializes the FIFO frontier. The start's parent `(None,None)` is a sentinel, meaning it has no predecessor or incoming action.

`parent` also replaces a separate visited set: if a state is a key, it has already been discovered.

#### Block C: BFS and level display

Relevant code block:

```python
    while frontier:
        state = frontier.popleft()
        explored += 1
        level = depth[state]
        if level != current_level:
            if current_level != -1:
                print(f"Total states at Level {current_level}: {level_count}\n")
            current_level = level
            level_count = 0
            print(f"{'=' * 15} LEVEL {level} {'=' * 15}")
        print(state)
        level_count += 1
        if state == goal:
            break
        for next_state, action in successors(state):
            if next_state not in parent:
                parent[next_state] = (state, action)
                depth[next_state] = depth[state] + 1
                frontier.append(next_state)
```

`popleft()` gives FIFO order. `explored += 1` increments the number of removed states. Changing `current_level` prints a heading and the previous level's count. These formatting counters do not decide which state to search next.

The goal test occurs before successor generation. `if next_state not in parent` marks states **when they are enqueued**, so duplicate configurations are not scheduled repeatedly. Every child's depth is its parent's depth plus one.

The program stops immediately when the goal is popped. Therefore the final printed “Total states at Level ...” may be a **partial count of that level**, not the mathematical total of all states at that depth.

#### Block D: recover the shortest moves

Relevant code block:

```python
    if current_level != -1:
        print(f"Total states at Level {current_level}: {level_count}")

    if goal not in parent:
        return None, explored

    path = []
    state = goal
    while parent[state][0] is not None:
        previous, action = parent[state]
        path.append(action)
        state = previous
    return path[::-1], explored
```

If goal was never discovered, return `None`. Otherwise, retrieve `(previous,action)` repeatedly until the start sentinel. Each collected action points forward from previous to current, but they are collected in reverse chronological order. `path[::-1]` restores start-to-goal order.

#### Block E: accept input and print the result

Relevant code block:

```python
disks = int(input("Enter number of disks: "))
moves, explored = state_space_search(disks)
print("\nStates explored:", explored)
print("\n========== SHORTEST SOLUTION ==========")
if moves is None:
    print("No solution found.")
else:
    for number, (source, destination) in enumerate(moves, 1):
        print(f"Move {number}: Peg {source} -> Peg {destination}")
    print("Total moves:", len(moves))
```

`input` returns a string; `int` converts it to a disk count. `enumerate(moves,1)` numbers the actions from 1. `is None` checks the explicit failure marker. An empty action list for 0 disks is different from failure.

### Dry run for 3 disks

The first successors of `((3,2,1),(),())` move disk 1 to peg 1 or peg 2. BFS keeps both possibilities, then explores their successors.

One returned shortest plan is:

| Move | Action | State afterward |
| --- | --- | --- |
| 0 | Initial | `((3,2,1), (), ())` |
| 1 | Peg 0 -> Peg 2 | `((3,2), (), (1,))` |
| 2 | Peg 0 -> Peg 1 | `((3,), (2,), (1,))` |
| 3 | Peg 2 -> Peg 1 | `((3,), (2,1), ())` |
| 4 | Peg 0 -> Peg 2 | `((), (2,1), (3,))` |
| 5 | Peg 1 -> Peg 0 | `((1,), (2,), (3,))` |
| 6 | Peg 1 -> Peg 2 | `((1,), (), (3,2))` |
| 7 | Peg 0 -> Peg 2 | `((), (), (3,2,1))` |

`2^3 - 1 = 7` verifies the minimum move count. **Moves = 7** and **explored states = 25** measure different things.

### Complexity and limitations

BFS on an explicit graph takes `O(V+E)` traversal work when state operations are treated as constant. Here `V = 3^n` and the branching factor is bounded by six source-destination pairs. This Python code copies and hashes states containing `n` disk labels, so `O(n * 3^n)` is a useful worst-case implementation estimate for both work and stored representation size, excluding printed text.

The classic recursive solver uses only `O(n)` recursion space, while this BFS stores many configurations to demonstrate state-space search. Keep the disk count small. The original accepts zero or negative input without validation; negative values misleadingly create empty start/goal states. Use a positive integer in the exam. A nonnumeric input raises `ValueError`.

### Viva questions with short answers

1. **What is a state here?** A complete arrangement of disks on the three pegs.
2. **What is state-space search?** Searching configurations linked by valid actions to reach a goal.
3. **Which search strategy does this code use?** Breadth-first search.
4. **Why is its solution shortest?** Every move costs one, and BFS finds the goal at minimum depth.
5. **Why not use the classic recursive algorithm?** This syllabus practical demonstrates explicit state-space search; the recursive routine is a different implementation.
6. **What is the minimum for `n` disks?** `2^n - 1`.
7. **What is the minimum for 4 disks?** 15 moves.
8. **Why only move `peg[-1]`?** Only the top disk is accessible.
9. **Why reject `top_destination < disk`?** It would put a larger disk on a smaller disk.
10. **Why copy each peg?** Successor states must be independent; one trial must not modify another.
11. **Why tuple of tuples?** It is immutable and hashable for dictionary keys.
12. **Where is the visited set?** The keys of `parent` act as the discovered set.
13. **What is stored in `parent`?** The preceding state and action used to reach the state.
14. **Why reverse the path?** Parent traversal goes from goal back to start.
15. **What is `deque`?** A double-ended queue; used here with append/popleft for FIFO behavior.
16. **Are 25 explored states the minimum moves?** No; the solution has 7 moves. Exploration includes other candidate states.
17. **Are the final level counts complete?** Not necessarily, because the search stops as soon as the goal is popped.

### Recreate the code from memory

Write `successors` carefully: pick top disk, check legality, copy, pop, append, freeze. Then write a normal BFS with a `parent` dictionary. The most important explanation is why the queue gives a shortest solution; decorative level printing can be added afterward.

### Keywords, functions and expressions to recognize

A keyword is reserved language syntax. Names such as `all`, `min`, `set`, `range` and `enumerate` are built-ins; they are not Python keywords.

| Expression | What it means here |
| --- | --- |
| `from ... import ...` | Keywords loading the deque queue type from collections. |
| `continue` | Skip an empty source or an illegal destination. |
| `or`, `and` | Combine legality tests; short-circuiting avoids indexing an empty peg. |
| `[-1]`, `.pop()`, `.append()` | Read, remove and add a top disk. |
| `map`, `tuple` | Built-ins freezing the mutable peg lists into a hashable state. |
| `is None` | Identity test for the specific no-solution marker, distinct from an empty successful path. |
