---
name: spreadsheet
description: >
  Work with spreadsheets — analyze, modify, debug, and build financial models.
  Use when the task involves .xlsx files, Excel, spreadsheet formulas, cell
  formatting, financial modeling, or debugging formula errors.
---

# Spreadsheet Tools

The `loadWorkbook` tool loads a `.xlsx` file into memory. All other tools operate on
the currently loaded workbook. Use `saveWorkbook` to persist changes; by default it
saves to a new path to prevent accidental overwrites.

## File Lifecycle

| Tool | Purpose |
|------|---------|
| `loadWorkbook` | Load an `.xlsx` file. Must be called first. |
| `saveWorkbook` | Save to disk. Set `overwrite: true` to update the source file. |
| `listWorkbooks` | List loaded files and their sheets. |

## Reading & Inspection

| Tool | Purpose |
|------|---------|
| `describeStructure` | Map sheets, data regions, headers, labels. Call first to understand layout. |
| `generateWorkbookContext` | Comprehensive natural-language description including auto-labels and formula context. More detailed than describeStructure. |
| `readCalculatedValues` | Read cell values and formula results. Supports cursors for pagination. |
| `inspect` | Per-cell detail: formula, value, number format, style, hyperlink, merge info. |
| `symbols` | Sheet dimensions, named ranges, tables, dependency graph stats. |
| `viewRange` | Render a rectangular range as a compact text grid. |
| `findCells` | Search for cells matching criteria: formula presence, fill color, formula/value substring, numeric range. Returns up to 200 matches. |

## Editing

| Tool | Purpose |
|------|---------|
| `editCells` | Write values or formulas, merge/unmerge cells, apply styles. Primary editing tool. |
| `fillCells` | Extend patterns (series, formulas) across ranges. Like Excel's fill handle. |
| `editCellStyles` | Apply visual formatting without changing values. |
| `manageSheets` | Add, remove, rename, or change visibility of sheets. |
| `manageRowsAndColumns` | Insert, delete, or auto-size rows and columns. |

## Formula Tools

| Tool | Purpose |
|------|---------|
| `runFormula` | Evaluate a formula without writing it to a cell. Use for testing. |
| `whatIf` | Temporarily set values and see which formulas would change. |
| `goalSeek` | Find the input value that produces a target output. |

## Dependency Graph

| Tool | Purpose |
|------|---------|
| `precedents` | Trace upstream — which cells feed into this one? |
| `dependents` | Trace downstream — which cells depend on this one? |
| `listErrors` | List all formula and model errors grouped by type. |

## Style Inspection

| Tool | Purpose |
|------|---------|
| `getStyles` | Read style information by index. |
| `getComment` | Read a single cell comment. |
| `getComments` | Read all comments in a sheet. |

## Conventions

### Cell References

Use A1 notation. Sheet-qualified references: `Sheet1!A1`. Sheets with spaces:
`'Loan Calculator'!A1`. Ranges: `A1:B10`.

Reference types in formulas:
- `$A$1` absolute (both locked)
- `$A1` mixed (column locked)
- `A$1` mixed (row locked)
- `A1` relative

### Values

- Booleans: write JSON `true` / `false` — the strings `"TRUE"` / `"FALSE"`
  produce text cells
- Empty/zero are different: `0` ≠ `""` ≠ `"-"` ≠ `null` ≠ `#N/A`
- Percentages: write as decimals (`1.83` for 183%)

### Formulas

- Test with `runFormula` before writing with `editCells`
- Extend across ranges with `fillCells` after writing the first cell
- Use absolute references (`$A$1`) for lookup tables and constants
- Use relative references for per-row data

## Reference Files

For deeper guidance on specific topics, read these files:

- [Debugging Spreadsheets](references/debugging-spreadsheets.md) — trace errors
  through the dependency graph, assess impact before edits, identify circular
  references
