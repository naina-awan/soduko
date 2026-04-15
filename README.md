# Sudoku CSP Solver

A Python implementation of a 9×9 Sudoku solver using **Constraint Satisfaction Problem (CSP)** techniques, combining **AC-3 arc consistency**, **backtracking search**, **forward checking**, and the **MRV heuristic** for efficient puzzle solving.

---

## Table of Contents

- [Overview](#overview)
- [Algorithms & Techniques](#algorithms--techniques)
  - [AC-3 (Arc Consistency)](#ac-3-arc-consistency)
  - [Backtracking Search](#backtracking-search)
  - [Forward Checking](#forward-checking)
  - [MRV Heuristic](#mrv-heuristic)
- [Project Structure](#project-structure)
- [Input Format](#input-format)
- [How to Run](#how-to-run)
- [Expected Output](#expected-output)
- [Function Reference](#function-reference)
- [Performance Statistics](#performance-statistics)

---

## Overview

Sudoku is modeled as a **CSP** where:

- **Variables** — each of the 81 cells `(row, col)` in the 9×9 grid
- **Domains** — digits `1–9` for empty cells; a fixed singleton `{digit}` for pre-filled cells
- **Constraints** — no two cells in the same row, column, or 3×3 box may share the same value

The solver first applies **AC-3** as a preprocessing step to prune domains, then uses **backtracking with forward checking** to find the solution.

---

## Algorithms & Techniques

### AC-3 (Arc Consistency)

AC-3 enforces _arc consistency_ across all variable pairs. For each arc `(Xi, Xj)`, it removes any value from `Xi`'s domain that has no compatible value in `Xj`'s domain.

**Key behavior:**

- Initializes a queue with every arc in the constraint graph
- Calls `revise()` to prune inconsistent values
- If any domain becomes empty → no solution exists
- Re-enqueues affected neighbors after a domain is revised

AC-3 is run **twice**: once as an upfront preprocessing pass on the initial board, and again inside the backtracking loop after each value assignment.

```
ac3(domains) → True if consistent, False if unsolvable
revise(domains, xi, xj) → True if xi's domain was reduced
```

### Backtracking Search

A standard recursive backtracking algorithm that:

1. Selects an unassigned variable (via MRV)
2. Tries each value in the domain (in ascending numeric order)
3. Applies forward checking and AC-3 to propagate constraints
4. Recurses; backtracks if no valid assignment is found

The search operates on **deep copies** of the domain dictionary at each node, so backtracking is simply a matter of discarding the failed copy.

### Forward Checking

After assigning a value to a variable, forward checking immediately removes that value from the domains of all neighboring (constrained) variables. If any neighbor's domain becomes empty, the assignment is abandoned before recursing.

This catches failures earlier than plain backtracking, dramatically reducing the search tree.

### MRV Heuristic

**Minimum Remaining Values (MRV)** — also called the _fail-first_ heuristic — selects the unassigned variable with the **fewest remaining legal values**. This focuses the search on the most constrained cells first, pruning large branches of the search tree early.

```python
select_unassigned_variable(domains)
# Returns the cell (row, col) with the smallest domain size > 1
```

---

## Project Structure

```
.
├── AI_Assignment5_23F-0777.py  # Main solver implementation
├── easy.txt                # Easy puzzle input
├── medium.txt              # Medium puzzle input
├── hard.txt                # Hard puzzle input
├── veryhard.txt            # Very hard puzzle input
└── README.md               # This file
```

> If running from a Jupyter notebook, replace `AI_Assignment5_23F-0777.py` with `AI_Assignment5_23F-0777.ipynb`.

---

## Input Format

Each puzzle file contains exactly **9 lines**, each with exactly **9 digits** and **no spaces or separators**.

- `0` represents an **empty cell**
- `1`–`9` represents a **pre-filled digit**

**Example (`easy.txt`):**

```
530070000
600195000
098000060
800060003
400803001
700020006
060000280
000419005
000080079
```

---

## How to Run

### Prerequisites

- Python 3.x (no external libraries required — uses only `copy` and `collections`)

### Running the Script

```bash
python AI_Assignment5_23F-0777.py
```

This will automatically solve all four puzzles in sequence: `easy.txt`, `medium.txt`, `hard.txt`, and `veryhard.txt`.

### Solving a Single Puzzle

Call `solve()` directly with a filename:

```python
from AI_Assignment5_23F-0777 import solve

solve("hard.txt")
```

### Running in Jupyter Notebook

1. Open `AI_Assignment5_23F-0777.ipynb` in Jupyter or VS Code
2. Select **Python 3** as the kernel
3. Run all cells top to bottom (`Kernel → Restart & Run All`)

---

## Expected Output

For each puzzle, the solver prints:

```
Solving: easy.txt

Solved Board:
5 3 4 6 7 8 9 1 2
6 7 2 1 9 5 3 4 8
1 9 8 3 4 2 5 6 7
8 5 9 7 6 1 4 2 3
4 2 6 8 5 3 7 9 1
7 1 3 9 2 4 8 5 6
9 6 1 5 3 7 2 8 4
2 8 7 4 1 9 6 3 5
3 4 5 2 8 6 1 7 9

Backtrack calls: 1
Failures: 0
```

If no solution exists, the output will be:

```
No solution exists.
```

---

## Function Reference

| Function                              | Description                                                                         |
| ------------------------------------- | ----------------------------------------------------------------------------------- |
| `get_neighbors(cell)`                 | Returns all 20 cells that share a row, column, or 3×3 box with `cell`               |
| `ac3(domains)`                        | Runs the AC-3 algorithm; returns `False` if any domain becomes empty                |
| `revise(domains, xi, xj)`             | Removes values from `xi`'s domain inconsistent with `xj`; returns `True` if revised |
| `select_unassigned_variable(domains)` | Returns the unassigned cell with the fewest remaining values (MRV)                  |
| `is_complete(domains)`                | Returns `True` when every cell has exactly one value in its domain                  |
| `backtrack(domains)`                  | Recursive backtracking search with forward checking and AC-3                        |
| `read_board(filename)`                | Reads a puzzle file and returns a 9×9 list of integers                              |
| `init_domains(board)`                 | Builds the initial domain dictionary from the board                                 |
| `print_board(domains)`                | Prints the solved grid to stdout                                                    |
| `solve(filename)`                     | Top-level function: loads, solves, and prints a puzzle with statistics              |

---

## Performance Statistics

Two global counters track solver efficiency:

- **`backtrack_calls`** — total number of times `backtrack()` was invoked (including the initial call)
- **`failures`** — number of times no value in a variable's domain led to a solution (i.e., dead ends)

Lower values indicate that AC-3 and forward checking pruned the search space more effectively. Easy puzzles typically show 1 backtrack call and 0 failures; very hard puzzles may require significantly more.

---

> **Author:** 23F-0777 | BCS-6E  
> **Course:** Artificial Intelligence 

