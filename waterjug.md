# 06. Water-Jug Problem Using DFS

## 1) Code

```python
def water_jug_dfs(capacity1, capacity2, target):
    visited = set()
    path = []
    actions = []

    def dfs(jug1, jug2):
        if (jug1, jug2) in visited:
            return False
        visited.add((jug1, jug2))
        path.append((jug1, jug2))
        if jug1 == target or jug2 == target:
            return True
        moves = [
            ((capacity1, jug2), "Fill Jug 1"),
            ((jug1, capacity2), "Fill Jug 2"),
            ((0, jug2), "Empty Jug 1"),
            ((jug1, 0), "Empty Jug 2"),
            (
                (max(0, jug1 - (capacity2 - jug2)),
                 min(capacity2, jug1 + jug2)),
                "Pour Jug 1 -> Jug 2"
            ),
            (
                (min(capacity1, jug1 + jug2),
                 max(0, jug2 - (capacity1 - jug1))),
                "Pour Jug 2 -> Jug 1"
            )
        ]
        for next_state, action in moves:
            if next_state == (jug1, jug2):
                continue
            actions.append(action)
            if dfs(next_state[0], next_state[1]):
                return True
            actions.pop()
        path.pop()
        return False
    
    if dfs(0, 0):
        return path, actions
    return [], []

def display_solution(solution, actions, capacity1, capacity2, target):
    print("\n")
    print("=" * 65)
    print("              WATER JUG PROBLEM")
    print("                   DFS SOLUTION")
    print("=" * 65)
    print(f"\nJug 1 Capacity : {capacity1} L")
    print(f"Jug 2 Capacity : {capacity2} L")
    print(f"Target         : {target} L")
    print("\n" + "-" * 65)
    print(f"{'STEP':<8}{'ACTION':<28}{'JUG 1':<12}{'JUG 2':<12}")
    print("-" * 65)
    print(
        f"{0:<8}"
        f"{'Start':<28}"
        f"{solution[0][0]:<12}"
        f"{solution[0][1]:<12}"
    )
    
    for i in range(1, len(solution)):
        jug1, jug2 = solution[i]
        print(
            f"{i:<8}"
            f"{actions[i - 1]:<28}"
            f"{jug1:<12}"
            f"{jug2:<12}"
        )
    print("-" * 65)
    final_jug1, final_jug2 = solution[-1]
    print(f"\nTarget achieved: {target} L")
    print(f"Final State    : ({final_jug1}, {final_jug2})")
    print(f"Total Steps    : {len(solution) - 1}")
    print("\n" + "=" * 65)
    print("                  SOLUTION PATH")
    print("=" * 65)
    for i, state in enumerate(solution):
        if i == 0:
            print(f"  Start  -> {state}")
        else:
            print(f"  Step {i:<2} -> {state}")
    print("=" * 65)
capacity1 = 3
capacity2 = 5
target = 4
solution, actions = water_jug_dfs(
    capacity1,
    capacity2,
    target
)
if solution:
    display_solution(
        solution,
        actions,
        capacity1,
        capacity2,
        target
    )
else:
    print("\n" + "=" * 65)
    print("                 NO SOLUTION FOUND")
    print("=" * 65)
```

## 2) Theory to write

### Question

**Official syllabus: Part B, assignment 2.** Use DFS for problem solving. Examples can be water jug problem , Missionaries and Cannibals ,maze solving, blocks world etc.

### Aim

To solve the water-jug problem using depth-first search with repeated-state detection and path reconstruction.

### Objectives

1. Represent water quantities as a pair of state variables.
2. Define valid fill, empty and pour operations.
3. Explore states recursively using DFS.
4. Prevent cycles and display a valid action sequence reaching the target.

### Requirements

Python 3; no external packages. Assume positive integer capacities, an unlimited water source and the ability to empty water. A pour stops when the source becomes empty or the destination becomes full.

### Theory

In the water-jug problem, unmarked jugs have fixed capacities. We must obtain a target amount using only filling, emptying and pouring. A state `(a,b)` stores the current quantity in jug 1 and jug 2.

**DFS** explores one branch as deeply as possible before trying another. Recursion provides an implicit stack. A global-to-this-search visited set prevents cycles such as filling and emptying the same jug repeatedly. This makes the finite graph search terminate, subject to available recursion depth and memory.

The goal test is `a == target or b == target`. It does **not** test whether `a+b` equals the target. With capacities 3 and 5, reaching `(3,4)` succeeds because jug 2 contains 4 L.

For positive integer capacities `m,n`, a nonnegative target `t` can be obtained in either individual jug under these assumptions exactly when `t <= max(m,n)` and `t` is divisible by `gcd(m,n)`. Target 0 succeeds immediately. Here `gcd(3,5)=1`, so 4 is feasible. The code searches directly and does not implement this preliminary feasibility check.

### Operators

| Action | Result from `(a,b)` |
| --- | --- |
| Fill jug 1 | `(m,b)` |
| Fill jug 2 | `(a,n)` |
| Empty jug 1 | `(0,b)` |
| Empty jug 2 | `(a,0)` |
| Pour 1 -> 2 | `(a-x,b+x)` where `x=min(a,n-b)` |
| Pour 2 -> 1 | `(a+y,b-y)` where `y=min(b,m-a)` |

### Algorithm

1. Initialize an empty visited set, current path and action list.
2. Start recursive DFS from `(0,0)`.
3. Reject an already visited state; otherwise mark it and append it to the path.
4. If either jug contains the target, return success.
5. Generate the six operations in the given order.
6. Skip operations that leave the state unchanged.
7. Tentatively append an action and recursively search its resulting state.
8. On success, preserve the lists and propagate success.
9. On failure, remove that action; after all children fail, remove the current path state.
10. Display the successful path, or report no solution if DFS exhausts the graph.

### Observation / result

For the supplied action order, DFS returns `(0,0) -> (3,0) -> (3,5) -> (0,5) -> (3,2) -> (0,2) -> (2,0) -> (2,5) -> (3,4)`, using 8 actions.

### Conclusion

DFS obtains a valid 4 L quantity in the 5 L jug while avoiding repeated states. The returned 8-action sequence is the first solution found in its traversal order and is not the shortest possible sequence.

## 3) Explaining the question, code and functioning

### Understanding the question

The question asks how to obtain an exact quantity using two unmarked jugs and legal fill, empty and pour actions. Your chosen method is DFS: search one sequence deeply and try alternatives when it fails.

**Say this in the oral:** “A state is the quantity in each jug. DFS tries fill, empty and pour operations recursively. A visited set prevents cycles. The current path and actions are undone when a branch fails. The code returns the first solution found, not necessarily the shortest.”

Memory cue: **Visit -> Goal? -> Six actions -> Recurse -> Undo on failure.**

### How the whole program runs

Python defines the outer solver and the display function, sets capacities/target, then calls `water_jug_dfs`. That call creates the nested `dfs` and invokes it on `(0,0)`. Recursive calls search until success. The outer solver returns path/actions, and the driver passes them to `display_solution`.

### What the three lists/sets mean

| Variable | Purpose | Removed during backtracking? |
| --- | --- | --- |
| `visited` | All states already explored in this search | No |
| `path` | States on the current recursion branch | Yes, when that state fails |
| `actions` | Actions connecting consecutive path states | Yes, when a child fails |

For a successful path, `len(actions) == len(path) - 1`. Once a state has been completely explored without reaching the goal, there is no benefit to exploring its same outgoing moves again from another branch in this state-based reachability problem. Therefore global visited entries remain.

### Code in blocks

#### Block A: outer function and recursive entry

Relevant code block:

```python
def water_jug_dfs(capacity1, capacity2, target):
    visited = set()
    path = []
    actions = []

    def dfs(jug1, jug2):
        if (jug1, jug2) in visited:
            return False
        visited.add((jug1, jug2))
        path.append((jug1, jug2))
        if jug1 == target or jug2 == target:
            return True
```

`dfs` is a **nested function**. It can read the capacities and target, and mutate the `visited`, `path` and `actions` objects defined in its enclosing function. No `global` or `nonlocal` declaration is needed because these objects are mutated rather than their variable names being rebound.

`(jug1,jug2) in visited` checks membership. `return False` prevents repeated expansion. `.add` stores a state in a set; `.append` adds a state to the list. The goal test is checked after the state is saved, so the successful final state is included in the path.

#### Block B: all possible operations

Relevant code block:

```python
        moves = [
            ((capacity1, jug2), "Fill Jug 1"),
            ((jug1, capacity2), "Fill Jug 2"),
            ((0, jug2), "Empty Jug 1"),
            ((jug1, 0), "Empty Jug 2"),
            (
                (max(0, jug1 - (capacity2 - jug2)),
                 min(capacity2, jug1 + jug2)),
                "Pour Jug 1 -> Jug 2"
            ),
            (
                (min(capacity1, jug1 + jug2),
                 max(0, jug2 - (capacity1 - jug1))),
                "Pour Jug 2 -> Jug 1"
            )
        ]
```

Each list entry is `(next_state, action_description)`. Filling and emptying are straightforward assignments to capacity or zero.

For pouring 1 -> 2, the code writes the equivalent formulas:

`new_jug1 = max(0, jug1 - (capacity2 - jug2))`

`new_jug2 = min(capacity2, jug1 + jug2)`

Example: from `(3,4)` with jug 2 capacity 5, only 1 L fits, giving `(2,5)`. `max` prevents a negative source quantity; `min` prevents overflow. For a pour, the total water quantity is conserved. Fill/empty operations do change the total.

#### Block C: DFS and undo operations

Relevant code block:

```python
        for next_state, action in moves:
            if next_state == (jug1, jug2):
                continue
            actions.append(action)
            if dfs(next_state[0], next_state[1]):
                return True
            actions.pop()
        path.pop()
        return False
    
    if dfs(0, 0):
        return path, actions
    return [], []
```

Tuple unpacking assigns each successor and its action. `continue` skips a no-op. Append the action **before** recursion so it corresponds to the child state if success occurs.

If the child returns `True`, immediately return `True` again; no `pop` runs on that successful branch. A failed child triggers `actions.pop()`. Once every child fails, `path.pop()` removes the current state. The nested `dfs` function uses recursion as its stack; `path` separately records what will be printed.

If a child was already visited, it returns immediately before adding itself to `path`. The caller still removes its tentatively added action, keeping the lengths consistent.

#### Block D: output table

The corresponding program lines define `display_solution`. It prints capacities and the initial state, then one row per action. `solution[0]` is the initial state, `solution[-1]` the final state, and `actions[i-1]` the move leading into `solution[i]`.

`f"{'STEP':<8}"` left-aligns a value in a field of width 8. Adjacent f-strings inside parentheses concatenate automatically; they are not separate rows. These width specifications only affect display, not the search.

#### Block E: supplied problem and outcome check

The corresponding program lines set capacities to 3 and 5, target to 4, and call the solver. `if solution` tests whether the result list is nonempty. `([],[])` means failure. Target 0 instead returns the one-state path `[(0,0)]`, which is nonempty and correctly represents success with zero actions.

### Dry run of the returned path

| Step | Action | Jug 1 | Jug 2 |
| --- | --- | --- | --- |
| 0 | Start | 0 | 0 |
| 1 | Fill Jug 1 | 3 | 0 |
| 2 | Fill Jug 2 | 3 | 5 |
| 3 | Empty Jug 1 | 0 | 5 |
| 4 | Pour Jug 2 -> Jug 1 | 3 | 2 |
| 5 | Empty Jug 1 | 0 | 2 |
| 6 | Pour Jug 2 -> Jug 1 | 2 | 0 |
| 7 | Fill Jug 2 | 2 | 5 |
| 8 | Pour Jug 2 -> Jug 1 | 3 | 4 |

This is the returned successful branch, not a list of every state examined on failed branches. DFS chooses the first available fill action first, which leads to extra actions. A separately verified shortest solution uses **6** actions:

`(0,0) -> (0,5) -> (3,2) -> (0,2) -> (2,0) -> (2,5) -> (3,4)`.

Do not change an 8-step DFS result to 6 in your journal unless you actually change the algorithm/order and observe that output. “DFS is not guaranteed shortest” is the correct explanation.

### Complexity and limitations

For integer capacities `m,n`, at most `(m+1)(n+1)` quantity pairs exist; many may be unreachable. Each visited state has at most six generated successors. Thus time is `O(V+E)`, bounded by `O(mn)` for positive capacities, and this implementation uses `O(V)` space for visited/path/recursion.

Do not quote only the smaller space bound of DFS tree traversal: this exact graph-search implementation also stores a visited set. Very large capacities may exceed Python's recursion limit. Negative capacities/targets and noninteger inputs are outside its intended model. The original contains no input-validation layer.

### Viva questions with short answers

1. **What is a state?** A pair giving the current water quantity in each jug.
2. **What is the initial state?** `(0,0)`.
3. **What is the goal test?** Either jug contains the target amount.
4. **What operations are allowed?** Fill either jug, empty either jug, pour in either direction.
5. **Why DFS?** It explores one sequence recursively before backtracking to alternatives.
6. **Where is the stack?** In Python's recursive function calls.
7. **Why a visited set?** To prevent cyclic repeated exploration of the same quantities.
8. **Does backtracking remove visited entries?** No; it removes current-path states/actions, not the explored-state record.
9. **Why `min` and `max` in pours?** To respect capacity and nonnegative-quantity limits.
10. **Why can the code return `(3,4)` for target 4?** Jug 2 contains 4; jug 1 does not have to be empty.
11. **Is the DFS result shortest?** Not guaranteed; the supplied run takes 8 actions while a shortest path takes 6.
12. **How would you guarantee fewest actions?** Use BFS with unit-cost actions.
13. **What changes the returned DFS path?** The order in which the six operations are explored.
14. **What is the feasibility condition?** For positive integer capacities, target no larger than the bigger jug and divisible by the capacities' gcd; 0 is immediate success.
15. **Why are `actions` one shorter than `path`?** The first state has no incoming action.
16. **Why skip unchanged successors?** They are no-op actions and do not advance the search.
17. **Is `dfs` the same as backtracking?** DFS is the traversal strategy; this recursive implementation uses undo operations when a branch fails.
18. **Why no `nonlocal path`?** The function mutates the existing list; it does not assign a new list to that name.

### Recreate the code from memory

Memorize the six operations and the order **append action -> recurse -> pop action on failure**. Keep global visited separate from current path. Change only the capacity/target constants for another numerical slip, and explain any changed feasibility assumptions.

### Keywords, functions and expressions to recognize

A keyword is reserved language syntax. Names such as `all`, `min`, `set`, `range` and `enumerate` are built-ins; they are not Python keywords.

| Expression | What it means here |
| --- | --- |
| `def dfs(...)` inside another function | A nested function that can use the outer solver variables. |
| `or` | Either jug may contain the target; it is not a sum-of-jugs test. |
| `continue` | Skip an action that leaves quantities unchanged. |
| `return True`, `return False` | Propagate success or failure through recursive calls. |
| `min`, `max` | Built-ins that clamp pour results to capacity and nonnegative quantities. |
| `.pop()` | Undo the current tentative action/state when that branch fails. |
