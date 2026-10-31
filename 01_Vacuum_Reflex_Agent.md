# 01. Simple Reflex Vacuum Cleaner Agent

Standalone study guide. The full commented program is included at the end. The ZIP contains the original files, observed transcripts, shared reference notes and GitHub folder structure.

## 1. Recognize the slip

**Official syllabus, Part A, assignment 1:** “Design and implement a simple reflex agent for the Vacuum Cleaner world and analyze its rational behavior in a given environment.”

**Say this in the oral:** “My program simulates two rooms and a vacuum cleaner. The agent receives its current location and that room's cleanliness. If dirty, it cleans; otherwise, it moves to the other room. The simulator stops when both rooms are clean.”

Memory cue: **Sense -> Apply rule -> Act -> Update world -> Check goal.**

## 2. What to write in the practical journal

### Aim

To implement a simple reflex agent for a two-room vacuum-cleaner world and analyze its behavior using condition-action rules.

### Objectives

1. Represent a two-room environment and the agent's percept.
2. Implement rules for cleaning and movement.
3. Simulate the agent from different starting locations.
4. Evaluate whether the actions clean both rooms without unnecessary actions in the given examples.

### Requirements

Python 3; no external packages. Two rooms named `Left` and `Right`, each `Dirty` or `Clean`.

### Theory

An **agent** perceives an environment through sensors and acts through actuators. A **percept** is the input received at one instant. Here it is `(location, status)`.

A **simple reflex agent** selects an action from the current percept using condition-action rules. It does not remember previous percepts or maintain an internal model of the other room.

| Condition | Action |
| --- | --- |
| Current room is dirty | Suck |
| Current room is clean and location is Left | Move Right |
| Current room is clean and location is Right | Move Left |

**PEAS description:**

| Component | Meaning in this program |
| --- | --- |
| Performance | Make both rooms clean; prefer fewer cleaning/movement actions |
| Environment | Two rooms with a cleanliness status and vacuum position |
| Actuators | Suck, move Left, move Right |
| Sensors | Current location and current room's status |

The simulator knows both rooms, but the policy sees only the current room. Therefore the environment is **partially observable to this agent**. It is also discrete, deterministic, single-agent and sequential. It is static in the usual sense that no external process changes dirt while the agent deliberates; the agent's own actions still change the world.

**Rationality:** an action is rational relative to a performance measure and available information. Cleaning the dirty current room and then visiting the other room is appropriate for the supplied cases. The policy has no `NoOp`/stop rule and cannot determine from a single clean-room percept that the other room is also clean. The global stopping decision belongs to the simulator. Do not claim the policy is universally optimal.

### Algorithm

1. Initialize the cleanliness of both rooms and the vacuum location.
2. Repeat for at most the specified number of steps.
3. Read the status of the room at the current location.
4. If dirty, return `Suck`; otherwise return movement to the other room.
5. Apply the action: clean the current room or change the location.
6. Display the percept, action and updated environment.
7. If both rooms are clean, stop the simulation.

### Observation / result

With both rooms dirty, starting from either side takes three actions: clean the starting room, move, clean the other room. Both rooms become clean.

### Conclusion

The simple reflex agent successfully cleans the supplied two-room environments using current-percept condition-action rules. The simulation demonstrates agent-environment interaction and the limits of a memoryless policy.

## 3. Understand the representation

`left_room` and `right_room` are strings storing the environment's dirt status. `location` is the vacuum position. `percept` is a tuple for displaying the sensed information. `action` is the string returned by the decision function.

The **simulator's state** is different from **agent memory**. The existence of `left_room` and `right_room` in `run_vacuum_world` does not make `vacuum_agent` a model-based agent.

## 4. Code in blocks

### Block A: choose an action

Original lines 3-10:

```python
def vacuum_agent(location, status):

    if status == "Dirty":
        return "Suck"
    elif location == "Left":
        return "Right"
    else:
        return "Left"
```

- `def` defines a function. `location` and `status` are parameters.
- `if`, `elif`, `else` select one branch. Checking dirt first means the vacuum cleans before moving.
- `==` compares values; it does not assign them.
- `return` sends an action string to the caller and immediately exits the function. This function chooses the action; the simulator carries it out later.
- Example: `vacuum_agent("Left", "Dirty")` returns `"Suck"`; `vacuum_agent("Left", "Clean")` returns `"Right"`.

### Block B: show the world

Original lines 12-22:

```python
def display_environment(left_room, right_room, vacuum_location):

    left = f"[{left_room}]"
    right = f"[{right_room}]"

    if vacuum_location == "Left":
        left = f"[Vacuum, {left_room}]"
    else:
        right = f"[Vacuum, {right_room}]"

    print(f"{left}  {right}")
```

`f"[{left_room}]"` is an f-string: Python inserts the variable value inside `{}`. The function labels whichever room contains the vacuum, then prints both strings. It does not change room cleanliness or choose actions.

### Block C: initialize, sense, and decide

Original lines 24-40:

```python
def run_vacuum_world(left_room, right_room, start_location, steps=10):

    location = start_location

    print("\nINITIAL STATE")
    display_environment(left_room, right_room, location)

    for step in range(1, steps + 1):

        if location == "Left":
            status = left_room
        else:
            status = right_room

        percept = (location, status)

        action = vacuum_agent(location, status)
```

- `steps=10` is a default argument. Calling the function without `steps` allows at most 10 iterations.
- `range(1, steps + 1)` produces step numbers 1 through 10, because the upper bound is excluded.
- The `if` chooses the status of the **current** room.
- `(location, status)` groups two values in a tuple.
- `vacuum_agent(location, status)` calls Block A. `percept` itself is displayed, while its two components are passed as separate arguments.

### Block D: carry out the action and stop

Original lines 42-62:

```python
        print(f"\nStep {step}")
        print("Percept:", percept)
        print("Action :", action)

        if action == "Suck":
            if location == "Left":
                left_room = "Clean"
            else:
                right_room = "Clean"

        elif action == "Right":
            location = "Right"

        elif action == "Left":
            location = "Left"

        display_environment(left_room, right_room, location)

        if left_room == "Clean" and right_room == "Clean":
            print("\n✔ Goal Achieved: Both rooms are clean!")
            break
```

`Suck` assigns `"Clean"` to the current room. Moving assigns a new value to `location`. `and` requires both room comparisons to be true. `break` exits the simulation's `for` loop. It does not return a value from the agent function.

### Block E: run two examples

Original lines 64-78:

```python
print("===== PERCEPT SEQUENCE 1 =====")

run_vacuum_world(
    left_room="Dirty",
    right_room="Dirty",
    start_location="Left"
)

print("\n\n===== PERCEPT SEQUENCE 2 =====")

run_vacuum_world(
    left_room="Dirty",
    right_room="Dirty",
    start_location="Right"
)
```

These are **keyword arguments**: the names show which parameter gets each value. Every call has its own local room state, so the second run starts with two dirty rooms again.

## 5. Dry run to understand the output

Starting state: Left dirty, Right dirty, vacuum at Left.

| Step | Percept before action | Returned action | World after action |
| --- | --- | --- | --- |
| 1 | `(Left, Dirty)` | Suck | Left clean, Right dirty, vacuum Left |
| 2 | `(Left, Clean)` | Right | Left clean, Right dirty, vacuum Right |
| 3 | `(Right, Dirty)` | Suck | Both clean, vacuum Right; stop |

Starting on Right produces `Suck -> Left -> Suck`. These are three **actions**, including two cleaning actions and one movement.

## 6. Complexity and limitations

Each decision takes constant time and constant auxiliary space. For a requested step limit `s`, simulation time is `O(s)` and auxiliary memory is `O(1)` apart from output. The supplied examples stop after three steps.

If both rooms start clean, this program still performs one movement before its first goal check. Without the simulator's global stop condition, the reflex policy would keep moving between clean rooms. If new dirt appears unpredictably, the current fixed simulation does not model it. String comparisons are case-sensitive: `"dirty"` is not `"Dirty"`.

On an old Windows terminal, the check-mark character in the success message may cause an encoding error. Replace only `✔` with `Goal achieved` or run a UTF-8 terminal if that occurs; it has no effect on the algorithm.

## 7. Viva questions with short answers

1. **What is a simple reflex agent?** An agent that selects an action using only the current percept and condition-action rules.
2. **What is the percept here?** Current location and the current room's cleanliness.
3. **What are the actions?** Suck, move Left and move Right.
4. **Why check dirt first?** To clean the current room before leaving it.
5. **Does the agent remember the other room?** No. The simulator stores the environment; the policy has no memory.
6. **Is the world fully observable?** Not to this policy: its percept does not reveal the other room's status.
7. **What is PEAS?** Performance measure, Environment, Actuators and Sensors.
8. **What makes an agent rational?** Choosing an action expected to maximize its performance measure using available information.
9. **Does the function `vacuum_agent` physically clean the room?** No. It returns an action; `run_vacuum_world` updates the room.
10. **Why use a step limit?** To prevent an uncontrolled simulation and set a finite budget.
11. **What does `break` do?** It exits the nearest enclosing loop.
12. **What does `return` do?** It exits a function and passes a value to its caller.
13. **How many states are possible?** `2 locations x 2 left statuses x 2 right statuses = 8` world states.
14. **How many percept combinations are possible?** Four: two locations times two current-room statuses.
15. **How is a model-based agent different?** It maintains an internal state/model to infer information not present in the current percept.
16. **Can the policy decide to stop when both rooms are clean?** Not from its current percept alone; the simulator handles that condition.

## 8. Recreate the code from memory

Write the three-rule `vacuum_agent` first. Then write a loop that senses the current room, calls the agent, applies the returned action, and checks both rooms. Add display formatting last.

If the examiner changes the starting conditions, change the three arguments in the example calls. Preserve the exact status/location strings. Explain the new action sequence before running it.

## Appendix A. Uploaded program: commented version

Only explanatory comments were added. Executable logic is the same as the uploaded original.

```python
# Simple Reflex Vacuum Cleaner Agent
# Study copy: only explanatory comments were added; algorithm is unchanged.
# See README.md for journal theory, block explanations, dry run and viva.

# Simple Reflex Vacuum Cleaner Agent

# Decision function: look only at the current location and current room status.
def vacuum_agent(location, status):

    # Condition-action rule 1: if the current room is dirty, clean it.
    if status == "Dirty":
        return "Suck"
    # Rules 2 and 3: from a clean room, move to the other room.
    elif location == "Left":
        return "Right"
    else:
        return "Left"

# Display function: build two strings; put the Vacuum label in its current room.
def display_environment(left_room, right_room, vacuum_location):

    left = f"[{left_room}]"
    right = f"[{right_room}]"

    if vacuum_location == "Left":
        left = f"[Vacuum, {left_room}]"
    else:
        right = f"[Vacuum, {right_room}]"

    print(f"{left}  {right}")

# Simulator: it stores the real world, repeatedly senses, acts, updates and displays.
def run_vacuum_world(left_room, right_room, start_location, steps=10):

    location = start_location

    print("\nINITIAL STATE")
    display_environment(left_room, right_room, location)

    # At most steps iterations. range excludes its upper bound, hence steps + 1.
    for step in range(1, steps + 1):

        # Sense ONLY the current room for the agent decision.
        if location == "Left":
            status = left_room
        else:
            status = right_room

        # A percept is the agent input: (location, status). This tuple is for display.
        percept = (location, status)

        # The simulator passes the same percept information as two arguments.
        action = vacuum_agent(location, status)

        print(f"\nStep {step}")
        print("Percept:", percept)
        print("Action :", action)

        # Apply the returned action to the simulated world.
        if action == "Suck":
            if location == "Left":
                left_room = "Clean"
            else:
                right_room = "Clean"

        elif action == "Right":
            location = "Right"

        elif action == "Left":
            location = "Left"

        display_environment(left_room, right_room, location)

        # The SIMULATOR checks both rooms. The simple reflex policy has no such memory.
        if left_room == "Clean" and right_room == "Clean":
            print("\n✔ Goal Achieved: Both rooms are clean!")
            break

# Run two separate simulations, once from each starting location.
print("===== PERCEPT SEQUENCE 1 =====")

run_vacuum_world(
    left_room="Dirty",
    right_room="Dirty",
    start_location="Left"
)

print("\n\n===== PERCEPT SEQUENCE 2 =====")

run_vacuum_world(
    left_room="Dirty",
    right_room="Dirty",
    start_location="Right"
)
```
