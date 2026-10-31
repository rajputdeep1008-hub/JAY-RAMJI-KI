# Last-Minute Revision: AI Practical and Oral

## 1. First understand the vocabulary

| Term | Meaning | Example from your code |
| --- | --- | --- |
| Problem | The task to solve | 8-puzzle, water jug, TSP |
| State | One complete configuration | Jug quantities `(3,2)` |
| State space | Configurations considered by the problem | Reachable puzzle boards |
| Action/operator | A permitted state change | Move blank Up; fill a jug |
| Initial state | Configuration before any action | Both jugs empty |
| Goal test | Condition that means success | Either jug contains 4 L |
| Path/plan | Ordered actions or configurations | Up -> Right -> Down |
| Path cost | Total cost of the actions so far | Number of puzzle moves |
| Search algorithm | Method deciding what to explore next | BFS, DFS, A* |
| Heuristic function | Estimate helping rank search choices | Manhattan distance |
| Evaluation/objective | Score for a candidate solution | Full TSP tour distance |
| Constraint | Rule a valid assignment must satisfy | Queens cannot share a diagonal |
| Backtracking | Undo a choice after a failed branch | Remove a queen and try another column |
| Agent | Perceives and acts in an environment | Reflex vacuum cleaner |
| Knowledge base | Stored facts/rules | Movie records or chatbot intents |

**A heuristic is a function, not automatically a separate search algorithm.** A* is an algorithm; Manhattan distance is the heuristic it uses. “State-space search” names an approach; your Hanoi implementation uses BFS within that approach. Backtracking is a search technique that your queens solver implements recursively.

## 2. Compare the algorithms you actually have

| Program | Next choice | Key storage | Success condition | Guarantee to describe |
| --- | --- | --- | --- | --- |
| Vacuum | Condition-action rule | Current percept | Driver sees both rooms clean | Cleans supplied examples; policy lacks global stop memory |
| A* | Minimum `g+h` | Heap, costs, parents, closed set | State equals goal | Shortest path here with consistent Manhattan and unit costs |
| Hanoi | Oldest queued state | FIFO deque, parents, depth | All disks on peg 2 | Shortest move count with unit-cost moves |
| Local beam TSP | Best `k` pooled neighbors | Beam, candidates, visited, best-ever | Budget/exhaustion; no fixed distance target | Best found; no global optimum guarantee |
| BFS companion | Oldest queued state | FIFO deque, parents | State equals goal | Shortest move count for unit-cost moves |
| Water DFS | First next recursive branch | Recursion, visited, path, actions | Either jug equals target | Finds a reachable solution in this finite graph; not necessarily shortest |
| 8-Queens | Next safe column in current row | Board, recursion, snapshots | Eight rows assigned | First valid board; not enumeration of all boards |
| Chatbot | First matching word-pattern rule | Intent dictionary | Respond until exact exit text | Only predefined rule coverage |
| Recommendation | Every movie satisfying all conditions | Movie facts and result list | Return matching list, possibly empty | Exact three-attribute filtering |

“BFS” usually means **breadth-first search** in these files. Do not confuse it with **greedy best-first search**, an informed strategy that chooses minimum `h`.

## 3. Nine short oral cards

### Vacuum

“My agent senses its location and current room status. Dirty means Suck; otherwise it moves to the other room. The simulator updates the world and stops when both rooms are clean.”

Know: percept, PEAS, rationality, simple reflex versus model-based, partially observable to the policy. Example: **Suck, Right, Suck** from Left with two dirty rooms.

### A* 8-puzzle

“I choose the board with minimum `g+h`. `g` is moves made and `h` is Manhattan distance of numbered tiles. Legal blank swaps generate neighbors. Costs and parent records retain cheaper routes and recover the goal path.”

Know: admissible, consistent, heap, tuple, `yield`, `g/h/f`. Example: **Up, Up, Left, Down, Right = 5 moves**, 6 popped states reported explored.

### Hanoi

“I search legal disk configurations using BFS. A successor moves only a source top disk to an empty peg or onto a larger disk. Parent links recover the shortest move sequence.”

Know: tuple-of-tuples, top at `[-1]`, why copy inner pegs, `2^n-1`. Example: **3 disks -> 7 moves**, pegs numbered 0,1,2.

### Local beam TSP

“I maintain two tours and generate pair-swap neighbors from both. I pool candidates, keep the two cheapest and save the historical best over ten iterations. The closing edge back to city zero is included.”

Know: beam width, pool versus independent hill climbs, random initialization, global generated-candidate visited pruning. Example verified best tour: **0,1,5,2,4,3,0; cost 90**. Say best found; the algorithm itself does not certify optimum.

### BFS planning

“I search puzzle states in FIFO order, marking discovered states on enqueue. At the goal, parent links give a shortest sequence because each move costs one.”

Know: the **original only displays levels**. Use the added complete solver for an actual solution. Example: **Up, Right, Down = 3 moves**. Original level counts: **1,3,5,10**.

### Water-jug DFS

“I recursively explore fill, empty and pour actions. Visited prevents cycles, while path and action lists record the current branch. Failed branches undo their path entries. Success is the target in either jug.”

Know: six operators, pour amount `min(source, free space)`, DFS not shortest. Example: **3 L and 5 L jugs -> 4 L target**, code returns **8 actions**; a shortest sequence has 6.

### 8-Queens

“I place one queen per row and check earlier queens for column or diagonal conflicts. On failure I remove the last queen and try another column. Row 8 means all queens were placed.”

Know: CSP variables/domains/constraints, diagonal formula, base case, undo step. Final zero-based columns: **0,4,7,5,2,6,1,3**. Snapshot cap 120 does not cap search.

### Chatbot

“I normalize text, split it into tokens and match the first intent whose pattern words all appear in the input. Then I choose one stored response randomly. Unknown messages use a fallback.”

Know: `re` cleans punctuation; token membership does intent matching. Word order ignored, no learning. Exact **bye/exit** stops; **bye!** only matches a farewell and continues.

### Movie recommendation

“I filter movie facts using the user's genre, language and mood. All three must match case-insensitively. I return every match and display a message if there are none.”

Know: knowledge base versus rule evaluator versus interface; `and`, `lower`, `set`, `sorted`. Example: **Sci-Fi + English + Exciting -> Inception**. It is a Part C mini-project and a fixed-rule system.

## 4. Fifteen common viva answers

1. **What is AI?** The study/design of systems that perceive, reason, learn or act to perform tasks associated with intelligent behavior. These practicals mainly demonstrate agents, search, constraints and rules.
2. **Is all AI machine learning?** No. Search and rule-based systems can work without learning from data.
3. **What is uninformed search?** Search without a problem-specific remaining-cost estimate; BFS and DFS are examples.
4. **What is informed search?** Search using additional estimates to guide exploration; A* uses `h`.
5. **What is completeness?** Whether the search is guaranteed to find a solution when one exists, under stated assumptions.
6. **What is optimality?** Whether the returned solution has minimum cost under the defined cost measure.
7. **Why are those different?** Finding some solution does not imply finding the cheapest solution; your DFS run demonstrates this.
8. **What is branching factor?** Number of successors from a state. In 8-puzzle it is at most 4; edges/corners have fewer.
9. **What is depth?** Number of actions from the initial state to a state.
10. **What is a frontier?** States discovered but not yet processed, stored as queue/heap in these searches.
11. **Why prevent repeated states?** To avoid cycles, redundant work and unnecessary memory use.
12. **Why use tuples in search?** The integer tuples are immutable and hashable, so they can be set members or dictionary keys.
13. **Why store parents?** To recover the chosen route after reaching the goal.
14. **Why count states separately from moves?** Search explores alternative candidates that do not appear in the final plan.
15. **Why explain assumptions?** Claims such as “BFS is optimal” require equal-cost moves; algorithm guarantees depend on the problem and implementation.

## 5. Mistakes to catch before writing

| Mistake | Correct statement |
| --- | --- |
| Heuristic is another search algorithm | It is an estimate function that can guide algorithms |
| Every BFS problem means best-first | Your queue programs use breadth-first; greedy best-first is different |
| DFS returns the shortest route | It returns the first route found in its traversal order |
| Goal displayed means goal found in code | Original BFS prints a separate goal constant without testing it |
| `MAX_STEPS` makes queens stop searching | It only stops recording snapshots |
| `return` and `print` are interchangeable | Return supplies a caller value; print displays text |
| `new = old` copies a list | It aliases the same object; use the right copying operation |
| All tuples are hashable | Only tuples whose elements are hashable are hashable |
| All AI learns | Your search and rule programs do not train a model |
| Chatbot regex detects intents | Regex cleans; token membership detects intents here |
| TSP's output label proves minimum | The beam search records best found; optimum was checked separately |
| Board/path formatting is the algorithm | Formatting can change while the state transitions/search remain the same |

## 6. Exam execution routine

1. Read the slip and identify the exact task, algorithm and numerical inputs.
2. Write the aim, representation and algorithm before decorative output.
3. Implement the core functions and check indentation/base cases.
4. Run the given example; verify the goal and legal transitions.
5. Record the actual output and a conclusion consistent with your program.
6. For the oral, explain state, actions, goal, data structure, stopping condition and one limitation.

If you have very little time, prioritize each oral card, the five warnings in the main index, and each algorithm's core loop. Then review the block explanations for any line you cannot explain without reading it aloud.
