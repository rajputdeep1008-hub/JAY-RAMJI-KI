# 07. 8-Queens Using Backtracking

Standalone study guide. The full commented program is included at the end. The ZIP contains the original files, observed transcripts, shared reference notes and GitHub folder structure.

## 1. Recognize the slip

**Official syllabus, Part B, assignment 3:** “Implement Backtracking search to solve the 8-Queens constraint satisfaction problem.”

**Say this in the oral:** “I place one queen per row. For each column I check conflicts with queens in earlier rows. A safe placement leads to recursion on the next row. If that branch fails, I remove the queen and try another column. Reaching row 8 means a complete solution.”

Memory cue: **Choose -> Check -> Recurse -> Undo if needed.**

## 2. What to write in the practical journal

### Aim

To solve the 8-Queens constraint satisfaction problem using backtracking search.

### Objectives

1. Represent a chessboard and queen placements.
2. Enforce nonattacking row, column and diagonal constraints.
3. Prune invalid partial assignments with a safety test.
4. Display a valid board and selected intermediate search states.

### Requirements

Python 3; no external packages. An 8 x 8 board using `Q` for a queen and `.` for an empty cell.

### Theory

A queen attacks horizontally, vertically and diagonally. The 8-Queens problem asks for eight queens on an 8 x 8 board with no attacking pair.

A **constraint satisfaction problem (CSP)** has variables, domains and constraints. Let `Q[r]` denote the chosen column for row `r`. Variables are `Q[0]...Q[7]`, each with domain `{0,...,7}`. For distinct rows `r,s`:

- Column constraint: `Q[r] != Q[s]`.
- Diagonal constraint: `abs(Q[r]-Q[s]) != abs(r-s)`.
- One queen per row is guaranteed by assigning one column for each row.

**Backtracking** incrementally builds a partial assignment. Whenever no legal value remains for the next variable, it undoes the previous assignment and tries an alternative. It is depth-first exploration with constraint checks that prune illegal partial boards.

This code checks safety before each placement. It does **not** implement full forward checking, arc consistency, or domain propagation. Do not describe those extra techniques as features of this shortened version.

### Algorithm

1. Initialize an empty 8 x 8 board and start at row 0.
2. If the row index is 8, return success.
3. For each column in the current row, check earlier queens for a column/diagonal conflict.
4. If safe, place a queen and optionally save a display snapshot.
5. Recursively attempt to solve the next row.
6. If the recursive call succeeds, propagate success.
7. Otherwise remove the queen, optionally record the backtrack, and try the next column.
8. If every column fails, return failure to the previous row.
9. Display the first solution found, or report failure if the search exhausts all choices.

### Observation / result

The supplied left-to-right search returns zero-based columns `[0,4,7,5,2,6,1,3]` for rows 0 through 7. All eight queens are nonattacking. The intermediate display records only the first 120 actions; the final solution board is printed separately.

### Conclusion

Backtracking finds a valid 8-Queens arrangement by pruning unsafe placements and undoing choices that lead to dead ends. The program stops after its first complete solution.

## 3. Understand the board and constraints

`board[row][col]` accesses one cell. Rows and columns are indexed 0 through 7. Coordinates shown in the saved-action labels are converted to 1 through 8 by adding one; that conversion does not change the algorithm's indices.

Example: a queen at `(0,0)` attacks `(1,0)` because the column matches and `(1,1)` because both row and column differ by 1. `(1,2)` is safe relative to `(0,0)`.

Only **earlier rows** need checking: later rows are empty, and the current row has no queen before a placement. “No row conflict” is therefore guaranteed by the construction of `solve`, rather than separately scanned in `is_safe`.

## 4. Code in blocks

### Block A: display memory and safety function

Original lines 1-6:

```python
MAX_STEPS = 120
states = []

def is_safe(board, row, col):
    return all(board[r][col] != 'Q' and abs(col - c) != row - r
               for r in range(row) for c in range(8) if board[r][c] == 'Q')
```

`all(...)` returns true only when every generated Boolean is true. It is a built-in function, not a Python keyword. The generator expression is compact; its meaning is:

```python
for r in range(row):
    for c in range(8):
        if board[r][c] == 'Q':
            if board[r][col] == 'Q' or abs(col - c) == row - r:
                return False
return True
```

`r < row`, so `row-r` is positive and already equals `abs(row-r)`. `board[r][col] != 'Q'` rules out an earlier queen in the same column. `abs(col-c) != row-r` rules out a diagonal. Under this solver's one-queen-per-earlier-row invariant, this is correct even though scanning every cell is less direct than a dedicated column list.

When `row=0`, there are no earlier queens. `all` of an empty iterable is `True`, so the first row can try its first column.

### Block B: save independent snapshots

Original lines 8-10:

```python
def save_state(board, action, row, col):
    if len(states) < MAX_STEPS:
        states.append((action, row + 1, col + 1, [r[:] for r in board]))
```

`MAX_STEPS=120` limits only the stored snapshots, not recursive search steps. Each record contains `(action,row+1,col+1,board_copy)`. `[r[:] for r in board]` copies every row. Without those inner copies, later placements/removals would also alter old saved boards.

The global list object is mutated by `.append`, so no `global states` declaration is required. The variable name is not rebound to a new list.

### Block C: the backtracking solver

Original lines 12-23:

```python
def solve(board, row):
    if row == 8:
        return True
    for col in range(8):
        if is_safe(board, row, col):
            board[row][col] = 'Q'
            save_state(board, "PLACE", row, col)
            if solve(board, row + 1):
                return True
            board[row][col] = '.'
            save_state(board, "BACKTRACK", row, col)
    return False
```

- `row == 8` is the recursion base case; rows 0 through 7 already contain queens.
- `range(8)` tries columns 0 through 7.
- `board[row][col] = 'Q'` is the choose step.
- `solve(board,row+1)` explores the next row using the same mutable board.
- `return True` preserves a complete successful board as recursion unwinds.
- `board[row][col] = '.'` is the undo step after failure.
- Returning `False` means no column worked for this row; it tells the caller to undo its own placement.

`return False` is after the loop, not inside it. If you indent it inside the loop, the program could give up after the first failed column.

### Block D: display saved boards

Original lines 25-37 group snapshots in chunks of four. `states[start:start+4]` slices the next group. `enumerate` produces a local index and the saved record. `_` receives a tuple item that is unused in that loop; `_` is a normal variable by convention, not a special discard keyword.

`" ".join(board[r])` joins the row's characters with spaces. `end="     "` keeps four boards side by side. This entire function is presentation; deleting it would not change the search result.

### Block E: construct and solve

Original lines 39-54:

```python
board = [['.' for _ in range(8)] for _ in range(8)]
print("==============================================")
print("       8-QUEENS BACKTRACKING SEARCH")
print("==============================================")
print("\nQ = Queen")
print(". = Empty Position")
if solve(board, 0):
    print("\nSearch states:")
    display_states()
    print("\n\n==============================================")
    print("               FINAL SOLUTION")
    print("==============================================")
    for row in board:
        print(" ".join(row))
else:
    print("No solution exists.")
```

The nested comprehension makes eight independent row lists. Avoid `[['.'] * 8] * 8`, which would reuse the same inner row object eight times and cause one cell assignment to affect multiple rows.

`solve(board,0)` starts at the first row. On success, the final `board` holds a valid solution and is printed independently of the snapshot log.

## 5. Dry run and verified final board

At row 0, column 0 is safe and gets a queen. At row 1, column 0 conflicts vertically, column 1 conflicts diagonally, and column 2 is safe. The solver continues deeper; if a later row has no safe column, it removes the latest queen and tries another choice. The final solution therefore need not keep these early choices.

| Row, zero-based | Final column, zero-based | Human coordinate |
| --- | --- | --- |
| 0 | 0 | `(1,1)` |
| 1 | 4 | `(2,5)` |
| 2 | 7 | `(3,8)` |
| 3 | 5 | `(4,6)` |
| 4 | 2 | `(5,3)` |
| 5 | 6 | `(6,7)` |
| 6 | 1 | `(7,2)` |
| 7 | 3 | `(8,4)` |

Every row has one queen; the columns are distinct; for every pair the absolute row gap differs from the absolute column gap.

The saved-state list reaches 120 entries. Its final saved snapshot is a backtrack, **not the final solution**. That is expected: the search continues after the recording limit. Use the separately printed “FINAL SOLUTION” board to show success.

## 6. Complexity and limitations

The generalized row-by-row N-Queens search has factorial-scale branching after excluding reused columns; `O(N!)` is a common search-tree shorthand. It is not a precise runtime bound for every implementation. This version scans up to `N` cells in each of `N` earlier rows for a safety check and tries up to `N` columns per partial assignment, so extra polynomial factors apply; a coarse upper bound for this style is `O(N^3 * N!)`.

The board uses `O(N^2)` storage and recursion depth `O(N)`. With at most `L` snapshots, copied-board storage is `O(L*N^2)`. Here `N=8` and `L=120`. The program finds one solution only, not all 92 arrangements of the standard eight-queen problem. Excluding symmetry there are 12 fundamental arrangements, but enumeration is not implemented here.

If asked to adapt to N-Queens, all hardcoded 8s must be generalized together; just changing the board allocation is insufficient. `states` also needs resetting before an independent rerun if you reuse functions in one interpreter.

## 7. Viva questions with short answers

1. **What is a CSP?** A problem defined by variables, domains and constraints.
2. **What are the variables here?** One chosen queen column for each row.
3. **What is each domain?** Columns 0 through 7.
4. **What constraints must hold?** No shared column or diagonal; one queen per row is enforced by construction.
5. **What is backtracking?** Build a partial solution, explore, and undo a choice when it cannot lead to completion.
6. **Why is no same-row test needed?** The recursive procedure places exactly one queen per row.
7. **How is a diagonal detected?** Absolute row difference equals absolute column difference.
8. **Why check only earlier rows?** Later rows contain no queens yet.
9. **What does `all` do?** It returns true if every generated test is true; an empty iterable also gives true.
10. **What is the base case?** `row == 8`.
11. **What exactly is undone?** The trial queen is replaced by `'.'` when deeper recursion fails.
12. **Does `MAX_STEPS` stop the search after 120 moves?** No. It only caps saved display snapshots.
13. **Why copy rows for snapshots?** To preserve old boards independently from later mutations.
14. **Does it find all solutions?** No. It returns immediately on the first solution.
15. **Is it full constraint propagation?** No. It checks consistency of proposed placements but does not propagate future domains.
16. **How are DFS and backtracking related here?** Recursion explores depth first, and backtracking adds pruning and undo steps.
17. **Why avoid repeated inner lists in board initialization?** Shared row objects would corrupt multiple rows when one cell is assigned.
18. **Why is the final snapshot not necessarily solved?** Recording stops at 120 entries while the search continues.

## 8. Recreate the code from memory

Start with the solver skeleton: base case, loop columns, safety check, place, recurse, remove. Then write column and diagonal conflict tests. Save/display states only after the core logic works. Be able to explain **why undo happens only after a failed recursive call**.

## Appendix A. Uploaded program: commented version

Only explanatory comments were added. Executable logic is the same as the uploaded original.

```python
# 8-Queens Using Backtracking
# Study copy: only explanatory comments were added; algorithm is unchanged.
# See README.md for journal theory, block explanations, dry run and viva.

# Limit ONLY the number of saved display snapshots, not the search itself.
MAX_STEPS = 120
states = []

# Check a new position against queens in PREVIOUS rows.
def is_safe(board, row, col):
    # Reject same column or same diagonal. Same-row conflicts are prevented by construction.
    return all(board[r][col] != 'Q' and abs(col - c) != row - r
               # Read this as nested loops over earlier rows, retaining cells that contain a queen.
               for r in range(row) for c in range(8) if board[r][c] == 'Q')

# Store an independent board snapshot and human-readable 1-based coordinates.
def save_state(board, action, row, col):
    if len(states) < MAX_STEPS:
        # [r[:] for r in board] copies each row so later edits do not change old snapshots.
        states.append((action, row + 1, col + 1, [r[:] for r in board]))

# Recursive backtracking: choose a column for this row.
def solve(board, row):
    # Rows 0..7 are filled; row == 8 means all eight queens are placed.
    if row == 8:
        return True
    for col in range(8):
        if is_safe(board, row, col):
            # Choose: tentatively place one queen.
            board[row][col] = 'Q'
            save_state(board, "PLACE", row, col)
            # Explore the next row. Propagate success immediately.
            if solve(board, row + 1):
                return True
            # Unchoose: the next row failed, so remove this queen and try another column.
            board[row][col] = '.'
            save_state(board, "BACKTRACK", row, col)
    # No column worked in this row: report failure to the previous recursive call.
    return False

# Display snapshots four boards at a time; this is presentation, not search logic.
def display_states():
    for start in range(0, len(states), 4):
        group = states[start:start + 4]
        print("\n" + "=" * 100)
        # _ receives an unused tuple field. enumerate adds an index to each snapshot.
        for i, (action, row, col, _) in enumerate(group):
            print(f"STEP {start + i + 1}: {action} ({row},{col})", end="       ")
        print()
        print("-" * 100)
        for r in range(8):
            for _, _, _, board in group:
                print(" ".join(board[r]), end="     ")
            print()
        print("=" * 100)

# Nested comprehension creates eight independent row lists.
board = [['.' for _ in range(8)] for _ in range(8)]
print("==============================================")
print("       8-QUEENS BACKTRACKING SEARCH")
print("==============================================")
print("\nQ = Queen")
print(". = Empty Position")
# Search begins at row 0 and stops at the first solution.
if solve(board, 0):
    print("\nSearch states:")
    display_states()
    print("\n\n==============================================")
    print("               FINAL SOLUTION")
    print("==============================================")
    # Print the actual final board separately; it can be beyond the saved snapshot limit.
    for row in board:
        print(" ".join(row))
else:
    print("No solution exists.")
```
