---
name: debugging-spreadsheets
description: >
  Trace cell errors and understand formula behavior using dependency-graph tools.
  Covers error propagation, impact assessment before edits, and formula debugging.
---

## Core principle

Errors propagate downstream — the cell showing `#DIV/0!` is rarely the cell that needs fixing. The `precedents` and `dependents` tools let you walk the dependency graph in either direction to find the source.

## Understanding an error

### Step 1: Trace upstream to find the source

```
precedents(erroring_cell, mode: "cells")
```

Walk the tree and look at `errorValue` on each node. The source is the deepest precedent with an `errorValue` — beyond it, precedents are clean.

### Step 2: Handle truncation

If the result is `truncated: { reason: "maxNodes" }`, re-query from the node that was cut:

```
precedents(truncated_node, mode: "cells", maxNodes: 500)
```

If truncated by `reason: "maxDepth"`, re-query the deepest error node as your new root.

### Step 3: Confirm the fix

After fixing the source error:
1. Record which cells had errors before
2. Re-trace from the originally erroring cell
3. Report: `resolved = before − after`, `new = after − before`

## Understanding a specific formula

Use `inspect` to see the formula and calculated value:

```
inspect(cell)
```

The formula string shows all referenced cells directly — there's no need for additional tools to see what inputs a formula uses. To understand *why* the formula produces its result, reason about:
- Function behavior vs. intended behavior
- Whether inputs have expected values
- Whether references point to the intended ranges

If the formula looks correct but the result doesn't, the bug is upstream — use `precedents` to trace back.

## Impact assessment before editing

Before changing a cell, see what depends on it:

```
dependents(cell_to_change, mode: "grouped")
```

`grouped` mode collapses cells with identical formulas, giving a compact view of blast radius. If any dependent looks critical, switch to `mode: "cells"` to verify specific formulas.

## Circular references

`alreadyVisited: true` flags cycle cuts in the returned tree.

Two cases:
- **Intentional cycles** (e.g., LBO interest↔debt loops) — rely on iterative calculation. If results look reasonable, flag the cycle but don't break it.
- **Accidental cycles** — typically produce `0` or errors. Find the edge whose formula looks wrong given nearby labels and cut it.

## When results seem wrong (no error)

If a cell has a value but it's wrong:

1. `inspect(cell)` — read the formula
2. Trace `precedents(cell, mode: "cells")` to verify input values
3. Check for:
   - `SUM` where `SUMIF`/`SUMIFS` was needed
   - Off-by-one ranges
   - VLOOKUP missing exact-match (`FALSE`)
   - Sign errors in cash-flow formulas
