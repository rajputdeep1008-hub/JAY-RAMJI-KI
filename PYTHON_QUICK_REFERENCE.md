# Python Used in These Nine Programs

Read this when you recognize an algorithm but cannot explain a Python expression. Examples use the same kinds of values as your practicals.

## 1. Keyword, function, method and variable are different

| Kind | Examples in your code | Meaning |
| --- | --- | --- |
| Keyword | `def`, `return`, `if`, `for`, `yield` | Reserved Python language syntax |
| Built-in function/type | `print`, `len`, `all`, `range`, `list`, `tuple`, `set` | Available without importing a module; these are not keywords |
| Method | `path.append(x)`, `state.index(0)`, `text.lower()` | Operation accessed on an object using a dot |
| Imported function | `heapq.heappop`, `random.choice`, `re.sub` | Function supplied by a module |
| User-defined function | `dfs`, `heuristic`, `cost`, `is_safe` | Function defined by your program using `def` |
| Variable/name | `board`, `row`, `goal`, `_` | A name referring to a value/object |

`key`, `end`, `sep`, `start`, and `steps` used as named arguments are **not automatically keywords**. They are parameter names. `_` is a normal variable conventionally used for an unneeded value.

## 2. Functions and calls

```python
def vacuum_agent(location, status):
    if status == "Dirty":
        return "Suck"
```

`def` defines the function; it does not execute its body yet. `location` and `status` are **parameters**, the names in the definition. In `vacuum_agent("Left","Dirty")`, the supplied strings are **arguments**. Calling the function executes its body.

`return` ends the function and passes a result back to the caller. `print` sends text to the screen and normally returns `None`. Printing an action does not replace returning it to a simulator that needs the action value.

`def bfs(max_level=3)` supplies a **default argument**. `bfs()` uses 3; `bfs(5)` overrides it. `run_vacuum_world(left_room="Dirty", ...)` uses **keyword arguments** to make the parameter-value mapping explicit.

The default integers used in these programs do not have the mutable-default-list problem.

## 3. Essential keywords

| Keyword/expression | Plain meaning | Example use |
| --- | --- | --- |
| `import random` | Load a module name | Random starts/responses |
| `from collections import deque` | Import a specific object | FIFO search queues |
| `if` / `elif` / `else` | Select a branch based on a condition | Vacuum action rules |
| `for x in items` | Visit each element of an iterable | Try each queen column |
| `while condition` | Repeat while condition is true | Process search frontier |
| `while True` | Repeat until explicitly exited | Chatbot conversation |
| `break` | Exit the nearest loop | Stop once both rooms are clean |
| `continue` | Skip to the next loop iteration | Ignore visited/no-op states |
| `return x` | End the function and give its result | Successful path or failure marker |
| `yield x` | Produce an item and pause a generator | Puzzle successors |
| `and` | Require both truth tests; short-circuit on a false first part | Movie attribute conditions |
| `or` | Accept either truth test; short-circuit on a true first part | Either jug contains target |
| `not` | Negate a truth test | `if not candidates` |
| `in` / `not in` | Test membership | State already discovered? |
| `is None` / `is not None` | Check the specific absence sentinel | Parent traversal / failure |
| `True`, `False`, `None` | Boolean true/false and absence sentinel | Recursive search results |

In these conditions, empty lists/sets/dictionaries, zero, empty strings and `None` are false. Nonempty collections/strings are true. Thus `if solution` means “is this result nonempty?”, while `if tile` excludes zero from Manhattan distance.

`and` and `or` can return operands, not just Boolean constants. In the displayed conditions they are used for their truth behavior.

## 4. The four containers you must distinguish

| Container | Example | Ordered? | Mutable? | Main use here |
| --- | --- | --- | --- | --- |
| List | `[0,1,2]` | Yes | Yes | Editable routes, board rows, paths, results |
| Tuple | `(0,1,2)` | Yes | No | Frozen search states and coordinates |
| Set | `{(0,0), (3,0)}` | No sequence-order guarantee | Yes | Visited states and distinct tours |
| Dictionary | `{state: (parent,action)}` | Insertion order in modern Python 3 | Yes | Key-value knowledge and parent records |

`[]` is an empty list, `()` an empty tuple, `set()` an empty set, and `{}` an empty dictionary. A one-item tuple needs a comma: `(3,)`.

Sets and dictionary keys require **hashable** objects. A tuple containing only integers is hashable; a list is not. A tuple containing a list is also not hashable. “Tuple means always hashable” would be an incorrect generalization.

Dictionary `x in data` checks **keys**, not all values. `for key,value in data.items()` supplies both. Set iteration has no defined index order, so `states[0]` is invalid for a set.

## 5. Indexing, slicing and board coordinates

| Expression | Meaning |
| --- | --- |
| `path[0]` | First item |
| `path[-1]` | Last item |
| `board[r][c]` | Cell in row r, column c |
| `path[:]` | Shallow copy of the full list |
| `path[::-1]` | Reversed sequence copy |
| `states[start:start+4]` | Up to four items, excluding the ending index |
| `state[i:i+3]` | One length-three row of a flat puzzle board |
| `a,b = b,a` | Exchange two values |
| `r,c = divmod(index,3)` | Row and column of a flat 3 x 3 board index |

`//` is floor division and `%` is remainder. For a nonnegative puzzle index 7, `7//3 == 2`, `7%3 == 1`. So index 7 is coordinate `(2,1)`. Converting back gives `2*3+1 == 7`.

`=` assigns a value; `==` compares values; `!=` means unequal. `+= 1` increments a number. `0 <= r < 3` means both `0 <= r` and `r < 3`.

## 6. Copies: why several programs use `[:]`

```python
new = path[:]                 # Good for a flat list of integer city labels.
copy = [row[:] for row in board]  # Copies outer collection AND each row.
```

`new = path` creates another name for the same list. Modifying `new` then modifies the same object seen through `path`.

For a nested board, `board[:]` alone copies only the outer list; inner row objects remain shared. Copy each row to preserve independent snapshots. Since cells are immutable strings here, another deeper copying step is unnecessary.

Similarly, `[['.'] * 8] * 8` repeats references to one row. Your queens program correctly uses nested comprehensions to create distinct rows.

## 7. Lists, sets, dictionaries and strings: methods used

| Operation | Effect |
| --- | --- |
| `items.append(x)` | Add x to a list's end; mutates the list |
| `items.pop()` | Remove and return the last list item |
| `visited.add(state)` | Add one member to a set |
| `visited.update(candidates)` | Add all members from an iterable to a set |
| `levels.setdefault(level, [])` | Return existing value; otherwise insert/return a new default list |
| `moves.items()` | Iterate over dictionary key-value pairs |
| `state.index(0)` | Index of the first occurrence of zero |
| `text.lower()` | Return a lowercased string |
| `text.strip()` | Return a string without outside whitespace |
| `text.split()` | Split on whitespace into tokens |
| `" ".join(values)` | Join a sequence of strings with spaces |

Mutating methods like `append`, `add`, `update`, and `random.shuffle` do not return the updated collection. Avoid `path = path.append(x)`: it sets `path` to `None`. `sorted(items)` returns a new list; it is different from an in-place mutation.

## 8. Comprehensions and generator expressions

```python
numbers = [x * x for x in range(3)]  # [0,1,4]
```

Read it as “for each x in 0,1,2, calculate x*x and collect it.”

The TSP candidate comprehension is equivalent to this idea:

```python
candidates = set()
for state in states:
    for route in neighbors(list(state)):
        if route not in visited:
            candidates.add(route)
```

Order is left-to-right for the `for` clauses: choose a beam state, then visit all its neighbor routes, then move to the next state.

`sum(expression for x in items)` uses a **generator expression**, producing one value at a time for `sum`. `all(test for x in items)` checks the tests and stops at the first false one. `all` over no values returns true.

`sorted(set(movie["genre"] for movie in movies))` means extract genres, remove duplicates, then sort. `tuple(map(tuple,next_pegs))` means convert every peg into a tuple, then make the outer collection a tuple.

## 9. Useful built-ins

| Name | Meaning in the pack |
| --- | --- |
| `len(x)` | Number of items |
| `range(8)` | Integers 0 through 7 during iteration |
| `range(1,steps+1)` | Integers 1 through steps |
| `range(n,0,-1)` | Count n down to 1 |
| `enumerate(items)` | Index-item pairs, starting with index 0 |
| `enumerate(items,1)` | Number-item pairs, starting with 1 |
| `sum(values)` | Add numeric values |
| `abs(x)` | Absolute value |
| `min(...)`, `max(...)` | Smallest/largest value |
| `min(states,key=cost)` | State with the smallest evaluated `cost` |
| `sorted(candidates,key=cost)` | New list ordered by evaluated tour cost |
| `int(input(...))` | Read a string and convert to integer |
| `str(x)` | Convert a value to a string for display |

`range` does not include its stop value. `key=cost` passes a function to be called for each candidate; `key=cost()` would immediately call it without a required route.

## 10. Queue, stack and heap

| Structure | Add | Remove | Search use |
| --- | --- | --- | --- |
| FIFO deque | `append(x)` | `popleft()` | BFS and Hanoi |
| LIFO list | `append(x)` | `pop()` | Stack idea and undo/path operations |
| Recursive call stack | Call function | Return from function | Water DFS and queens |
| Min-heap | `heapq.heappush` | `heapq.heappop` | A* priority selection |

A heap is not a fully sorted list. It maintains enough order to remove a minimum efficiently. An A* entry `(f,g,state)` is compared lexicographically: f first, then g, then the integer state tuple. `deque` means double-ended queue, but these BFS implementations use it specifically as FIFO.

## 11. Recursion, backtracking and generators

**Recursion:** a function calls itself on a smaller or next-stage problem. It needs a stopping/base case. Water DFS stops on a goal or exhausted successors; queens stops when row 8 is reached or a row has no safe column.

**Backtracking:** undo a tentative choice when it fails. `actions.pop()` undoes a trial water-jug action; `board[row][col]='.'` undoes a queen placement. Recursion does not automatically undo changes to shared mutable lists or boards; your code must do that explicitly.

**Generator:** a function containing `yield` returns an iterable generator object when called. A `for` loop asks it for successive items. `yield` pauses local execution; the next iteration resumes after that point. `return` ends the function.

## 12. Scope and the main guard

The nested water-jug `dfs` sees variables from `water_jug_dfs`. Appending to `path` mutates the existing list. It does not require `nonlocal`, which would be needed to rebind that enclosing variable name.

The chatbot and recommendation programs use:

```python
if __name__ == "__main__":
    main()
```

For the chatbot the called function is `chatbot()` rather than `main()`. The condition is true on direct script execution and false on ordinary import. The other supplied originals have top-level driver code, so importing them also starts their demonstrations.

## 13. Printing, f-strings and regular expressions

`f"Step {i}"` inserts i's value. `f"{value:<8}"` left-aligns it in a field at least eight characters wide. `print(...,end=" ")` avoids the default newline. `print(a,b,sep="")` removes the usual separating space. `"=" * 50` repeats a separator character. `"\n"` is a newline.

In the chatbot, `re.sub(r'[^\w\s]', '', text)` removes characters outside word/whitespace classes. The raw-string prefix is Python syntax; `\w`, `\s`, brackets and negation are regex syntax. `random.choice` chooses a stored response, while `random.shuffle` reorders a tour list in place.

## 14. If the examiner points at an unfamiliar line

Explain it in this order: **What data goes in? Which operation is applied? What value/object changes or comes out? Why does the algorithm need it?** For example: “`parent[next_state]=(state,action)` saves the predecessor and move for a newly discovered state, so I can reconstruct the goal plan later.”
