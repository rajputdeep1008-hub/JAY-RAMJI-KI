# 05. BFS Planning with the 8-Puzzle

Standalone study guide. The full commented program is included at the end. The ZIP contains the original files, observed transcripts, shared reference notes and GitHub folder structure.

## 1. Recognize the slip

**Official syllabus, Part B, assignment 1:** “Use BFS to solve a planning problem. Examples can be 8 puzzle problem, Robot path planning, Blocks world etc.”

**Read this first:** the uploaded shortened program correctly demonstrates BFS levels, but it **never tests `state == goal` and never returns an action plan**. The goal is merely printed after the search. For a slip asking you to *solve* the problem, use the separate `bfs_goal_solution.py`, which adds a goal test and parent/action links. Your original is preserved.

**Say this in the oral for the complete version:** “I use a FIFO queue to search puzzle states level by level. A visited record prevents repeated states. When the goal is popped, I reconstruct its parent links to return a shortest action sequence, because every move has equal cost.”

Memory cue: **Queue -> Pop oldest -> Goal? -> New neighbors -> Save parent -> Enqueue.**

## 2. What to write in the practical journal

### Aim

To solve an 8-puzzle planning problem using breadth-first search and obtain a shortest sequence of unit-cost moves.

### Objectives

1. Represent an initial state, goal and valid puzzle actions.
2. Explore configurations in increasing depth using a FIFO queue.
3. Avoid repeated states.
4. Identify the goal and recover the action sequence.

### Requirements

Python 3 and `collections.deque`. Zero represents the blank.

### Theory

**Planning** finds an action sequence transforming an initial state into a state satisfying a goal. In this puzzle, an action swaps the blank with an adjacent tile. Preconditions are that the blank's destination is within the board; effects are the exchanged positions of the blank and that tile.

**BFS** is an uninformed search method. A FIFO queue processes all states at depth 0, then depth 1, then depth 2, and so on. It does not calculate a heuristic. A visited/discovered record prevents cycles and duplicate work.

BFS is complete on this finite reachable state space. It returns a shortest path in **number of moves when all step costs are equal**. It is not a general minimum-weight path algorithm for unequal-cost actions. For unequal positive costs use uniform-cost search, or an appropriate A* implementation.

The original code has a display cutoff of depth 3. Such a cutoff would miss any deeper goal even if a goal check were added. The complete companion solver removes that arbitrary display cutoff and returns a goal path; it is still intended for small demonstrations because BFS can use much memory.

### Algorithm: complete goal-solving version

1. Put the initial state in a FIFO queue.
2. Store its parent sentinel in a dictionary, also used as the discovered set.
3. While the queue is nonempty, remove its oldest state.
4. If it is the goal, collect state/action pairs by following parents and reverse them.
5. Otherwise generate every legal blank-move successor.
6. For an undiscovered successor, save its parent and incoming action, then enqueue it.
7. If the queue empties, report no reachable solution.

### Observation / result

The original displays level sizes **1, 3, 5, 10** for depths 0 through 3. The given goal occurs at depth 3, although the original never recognizes it. The companion solver returns the three moves **Up -> Right -> Down**.

### Conclusion

The complete BFS solver obtains a shortest three-move plan for the supplied 8-puzzle. The original shortened script separately demonstrates breadth-first level expansion, but does not itself verify goal achievement.

**If you run only the original:** use the aim “To demonstrate level-wise BFS expansion for an 8-puzzle planning problem,” describe a level-display algorithm, and conclude that the displayed tree contains the supplied goal at depth 3. Do not claim the program detected or reconstructed the solution. This is a narrower demonstration than the official solve requirement.

## 3. Representation

Start: `4 1 3 / 0 2 5 / 6 7 8`.

Goal: `1 2 3 / 4 0 5 / 6 7 8`.

Both are flat nine-element tuples. `state.index(0)` finds the blank. `divmod(pos,3)` maps index to row/column; `row*3+column` maps back. Tuples can be used in a set because their elements are immutable integers.

## 4. Original code in blocks

### Block A: states and import

Original lines 1-8 import the FIFO queue and define start/goal. The multiple display lines inside parentheses still form one flat tuple, not a matrix of lists.

### Block B: legal successors

Original lines 10-19:

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

The direction pairs mean Up, Down, Left, Right. For every direction, calculate the candidate blank position and check bounds. `list(state)` allows a swap; `tuple(new)` freezes the result. `yield` produces one successor per valid direction.

### Block C: search by levels

Original lines 21-34:

```python
def bfs(max_level=3):
    queue = deque([(start, 0)])
    visited = {start}
    levels = {}
    while queue:
        state, level = queue.popleft()
        if level > max_level:
            break
        levels.setdefault(level, []).append(state)
        for new_state in moves(state):
            if new_state not in visited:
                visited.add(new_state)
                queue.append((new_state, level + 1))
    return levels
```

- Queue entries are `(state, level)`, beginning with `(start,0)`.
- `visited = {start}` is a one-element set. `levels = {}` is an empty dictionary.
- `popleft()` gives FIFO order; `append()` adds new work at the other end.
- `if level > max_level: break` is valid for this display because BFS queue depths are nondecreasing.
- `levels.setdefault(level, [])` returns this depth's list, creating it if absent. `.append(state)` then records the state for display.
- Marking visited before enqueuing avoids multiple queued copies of a state.
- States at depth 3 still generate depth-4 children, which are queued but not displayed. This is avoidable overhead, not extra displayed levels.
- There is no goal test, parent dictionary, or returned plan in this function.

### Block D: print many boards horizontally

Original lines 36-43:

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

The outer loop chooses a board row. The inner loop prints that row from every board in the level. A comprehension replaces zero with `"_"` and converts other numbers using `str`, because `join` needs strings. `end="     "` keeps boards on the same console line; the final `print()` moves to the next row.

### Block E: driver and independent goal display

Original lines 45-54 call `bfs(3)`, print all returned levels, then print the goal constant separately. The output label “Total Levels Displayed: 3” actually gives the **maximum depth** in this run; depths 0,1,2,3 are **four** displayed levels. These are two different counts.

## 5. Understand the companion fix

Open bfs_goal_solution.py. Its successor function returns `(next_state, blank_move)` instead of just a state. Its core structure is:

```python
frontier = deque([start])
parent = {start: (None, None)}
while frontier:
    state = frontier.popleft()
    if state == goal:
        # Follow parent links, reverse, and return the recovered path.
        ...
    for next_state, action in successors(state):
        if next_state not in parent:
            parent[next_state] = (state, action)
            frontier.append(next_state)
```

This excerpt illustrates the control flow; the linked `.py` file contains the complete runnable implementation. `parent` serves two purposes: detecting previously discovered states and remembering the plan. It marks states on insertion, not only when removed.

## 6. Dry run of the solution

| Depth | Move from previous state | Board, rows separated by `/` |
| --- | --- | --- |
| 0 | Initial | `4 1 3 / 0 2 5 / 6 7 8` |
| 1 | Up | `0 1 3 / 4 2 5 / 6 7 8` |
| 2 | Right | `1 0 3 / 4 2 5 / 6 7 8` |
| 3 | Down | `1 2 3 / 4 0 5 / 6 7 8` |

The original shows this goal as the first board in displayed level 3, but that fact is inferred by inspecting its output. The companion checks it in code.

## 7. Complexity

For an explicit graph with constant-size states, BFS takes `O(V+E)` time and `O(V)` stored states. In a search tree with branching factor `b` and goal depth `d`, resource use grows exponentially. An implementation that generates children before completing a goal depth can generate an additional frontier level, so `O(b^(d+1))` is a safe conventional bound; simpler summaries often say exponential `O(b^d)` when `b` is treated as fixed. Use the explicit-graph bound when explaining these finite-state programs.

The fixed-depth original stores `levels` as well as visited/frontier; it is a visualization, not a memory-optimized solver. BFS can be expensive for deep 8-puzzle instances, so A* is often more practical with an informative heuristic.

## 8. Viva questions with short answers

1. **What is BFS?** A search that processes configurations in increasing depth using a FIFO queue.
2. **What is FIFO?** First in, first out.
3. **Why `deque` rather than list `pop(0)`?** Removing the first list item shifts others; deque supports efficient removal from the left.
4. **Does BFS use a heuristic?** No. It is uninformed search.
5. **Why keep a visited record?** To avoid cycles and duplicate states.
6. **When do you mark a state visited?** When enqueuing/discovering it, to avoid duplicate scheduling.
7. **When is BFS optimal?** For a shortest move count when all actions have equal cost.
8. **What if actions have different costs?** BFS's minimum depth need not be minimum cost; use uniform-cost search or suitable A*.
9. **What is a plan here?** The blank-movement sequence taking start to goal.
10. **Does the original `BFSPLANNING.py` solve the puzzle explicitly?** No. It displays levels and prints the goal separately.
11. **What is missing from the original?** Goal comparison and parent/action-based solution reconstruction.
12. **What does `setdefault` do?** Returns an existing dictionary value, or inserts and returns a default if the key is absent.
13. **Why can the level-limit code use `break`?** After the first depth above the limit, all remaining queue entries are at that depth or deeper.
14. **How many displayed levels are there?** Four, for depths 0 through 3.
15. **What is the solution length for this board?** Three moves: Up, Right, Down.
16. **BFS versus A*?** BFS orders by depth; A* orders by estimated total cost `g+h`.
17. **Why store a parent action?** A state sequence can show boards, but an action sequence explicitly states the plan.
18. **Is the companion a different problem?** No. It uses the same supplied start/goal and legal moves, adding explicit solution detection and recovery.

## 9. Recreate the code from memory

Learn legal blank swaps once; both your BFS and A* practicals use them. For BFS remember **FIFO + discovered record + goal check + parents**. To adapt to a changed slip, edit the top-level start and goal in the complete companion and keep the blank as zero.

## Appendix A. Uploaded program: commented level-display version

Only explanatory comments were added. Executable logic is the same as the uploaded original.

```python
# BFS Planning with the 8-Puzzle
# Study copy: only explanatory comments were added; algorithm is unchanged.
# See README.md for journal theory, block explanations, dry run and viva.

# deque provides the FIFO frontier required for BFS.
from collections import deque

# State is a flat tuple; 0 is the blank. The display prints it as an underscore.
start = (4, 1, 3,
         0, 2, 5,
         6, 7, 8)
goal = (1, 2, 3,
        4, 0, 5,
        6, 7, 8)

# Yield legal blank-swapping successors.
def moves(state):
    pos = state.index(0)
    # divmod(pos, 3) returns (row, column).
    r, c = divmod(pos, 3)
    for dr, dc in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
        nr, nc = r + dr, c + dc
        # Keep only board coordinates from 0 to 2.
        if 0 <= nr < 3 and 0 <= nc < 3:
            # Copy into a list to swap; freeze back into a tuple for the visited set.
            new = list(state)
            p = nr * 3 + nc
            new[pos], new[p] = new[p], new[pos]
            yield tuple(new)

# IMPORTANT: this original function displays levels; it does not test for a goal.
def bfs(max_level=3):
    # Queue entries contain both a board and its depth.
    queue = deque([(start, 0)])
    # Mark on enqueue to avoid scheduling the same state repeatedly.
    visited = {start}
    levels = {}
    while queue:
        state, level = queue.popleft()
        # The break is safe for level display because FIFO depths are nondecreasing.
        if level > max_level:
            break
        # Get/create the list for this depth, then append the current state.
        levels.setdefault(level, []).append(state)
        # This also generates depth max_level + 1 states, which are not displayed.
        for new_state in moves(state):
            if new_state not in visited:
                visited.add(new_state)
                queue.append((new_state, level + 1))
    return levels

# Display all boards of one depth horizontally, one board row at a time.
def print_level(level, states):
    print(f"\nLEVEL {level}")
    for row in range(3):
        for state in states:
            # Convert integers to strings before join; show 0 as an underscore.
            values = ["_" if x == 0 else str(x)
                      for x in state[row * 3:row * 3 + 3]]
            print(" ".join(values), end="     ")
        print()

# Call the level demonstration with depths 0, 1, 2, 3.
levels = bfs(3)
print("========== 8-PUZZLE BFS ==========")
for level, states in levels.items():
    print_level(level, states)
# Printing goal is separate: no comparison to goal occurs anywhere in the search.
print("\n========== GOAL STATE ==========")
for row in range(3):
    values = ["_" if x == 0 else str(x)
              for x in goal[row * 3:row * 3 + 3]]
    print(" ".join(values))
# For this run the label is maximum displayed depth; four levels include depth 0.
print("\nTotal Levels Displayed:", len(levels) - 1)
```

## Appendix B. Complete BFS solution version

This separate companion explicitly detects the goal and reconstructs a shortest plan for the same puzzle. Use this version when the slip asks you to solve the planning problem.

```python
"""Complete BFS solver for the uploaded 8-puzzle planning example.

The uploaded BFSPLANNING.py remains unchanged. This companion adds an explicit
goal test and a shortest path with blank-movement labels. Python standard library
only. Run: python bfs_goal_solution.py
"""

from collections import deque

START = (4, 1, 3, 0, 2, 5, 6, 7, 8)
GOAL = (1, 2, 3, 4, 0, 5, 6, 7, 8)


def successors(state):
    """Yield a legal next board and the blank move used to reach it."""
    blank = state.index(0)
    row, col = divmod(blank, 3)
    for action, dr, dc in [("Up", -1, 0), ("Down", 1, 0),
                           ("Left", 0, -1), ("Right", 0, 1)]:
        r, c = row + dr, col + dc
        if 0 <= r < 3 and 0 <= c < 3:
            other = r * 3 + c
            new = list(state)
            new[blank], new[other] = new[other], new[blank]
            yield tuple(new), action


def bfs(start, goal):
    """Return [(state, incoming_action), ...], or None if no path exists.

    The first entry is (start, None). All moves cost one, so the returned
    plan has the fewest moves. Integer tuples must be valid 8-puzzle boards.
    """
    frontier = deque([start])
    parent = {start: (None, None)}  # Also serves as the discovered set.

    while frontier:
        state = frontier.popleft()
        if state == goal:
            path = []
            while state is not None:
                previous, action = parent[state]
                path.append((state, action))
                state = previous
            return path[::-1]

        for next_state, action in successors(state):
            if next_state not in parent:
                parent[next_state] = (state, action)
                frontier.append(next_state)

    return None


def show(state):
    for row in range(0, 9, 3):
        print(" ".join("_" if value == 0 else str(value)
                       for value in state[row:row + 3]))


if __name__ == "__main__":
    solution = bfs(START, GOAL)
    if solution is None:
        print("No solution exists.")
    else:
        print("Total moves:", len(solution) - 1)
        for step, (state, action) in enumerate(solution):
            print(f"\nStep {step}: {action or 'Initial State'}")
            show(state)
```
