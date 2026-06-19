`@grid-is/spreadsheet-engine` reference
=======================================

The central class is `Model`, a collection of one or more `Workbook`s plus a shared dependency
graph. Workbooks contain `WorkSheet`s, which contain `Cell`s.

Everything below is verified against v16.x. For anything missing here, grep
`node_modules/@grid-is/spreadsheet-engine/dist/index.d.ts`; every public method carries JSDoc, often
with examples.

Setup
-----

```js
import { Model } from "@grid-is/spreadsheet-engine";
await Model.preconditions;   // required once per process before most Model calls
```

Creating a model
----------------

```js
// From an XLSX file on disk (Node only)
const model = await Model.fromXLSXFile("budget.xlsx");

// From XLSX binary data (browser or Node)
const model = await Model.fromXLSX(arrayBufferOrUint8Array, "budget.xlsx");

// Blank workbook with one empty "Sheet1"
const model = Model.empty("workbook.xlsx");

// From pre-converted JSON representations
const model = Model.fromCsf(csf);   // SheetJS Common Spreadsheet Format
const model = Model.fromJSF(jsf);   // JSON Spreadsheet Format (@borgar/xlsx-convert)
```

Loading options (`AddWorkbookOptions`) include `readOnly: true`, which skips formula parsing and
dependency-graph construction for much faster loads, at the cost of writes and recalculation. Use it
for display-only or extract-only workloads.

A model can hold several workbooks: `model.addWorkbookFromXLSXFile(path)`,
`model.addWorkbookFromXLSX(data, filename)`, `model.addWorkbook(jsf)`, `model.removeWorkbook(id)`,
`model.getWorkbooks()`.

Reading
-------

Read expressions are formula strings starting with `=`:

```js
model.readValue("=B2");          // CellValue: string | number | boolean | FormulaError | null
model.readValue("=B2", 0);       // optional fallback for empty results
model.readCell("=B2");           // full Cell object (value .v, formula .f, number format .z, …)
model.readCells("=A1:C10");      // 2-D array of Cell objects (rows of columns)
model.runFormula("=SUM(D:D)");   // evaluate any formula against the model
```

`readCells` accepts `{ cropTo: "cells-with-non-blank-values" }` to trim trailing blank rows/columns.
`model.getCell(cellId, sheetName?, workbookName?)` fetches by address parts instead of an
expression.

Useful `Cell` members: `.v` (computed value), `.f` (formula string or null), `.z` (number format),
`.id`, `.isSpilled()`, `.isSpillAnchor()`.

Writing values
--------------

```js
model.write("B2", 42);                       // single write + automatic recalc of dependents
model.writeMultiple([                        // many writes, ONE recalculation pass
  ["B2", 42],
  ["B3", "Total"],
  ["Sheet2!C1", true],
]);
```

`write` accepts a `WriteOptions` third argument: `skipRecalc`, `forceRecalc`, `skipVolatiles`,
`reset` (drop earlier writes first), `neutralizeFormulaOnSingleCellWrite` (writing a value over a
formula cell suspends the formula until reset). `model.clearCells(ref)` clears a range;
`model.writes()` lists writes made so far; `model.reset()` reverts the model to its loaded state.

Writing a value over a formula cell does not destroy the workbook's formula permanently. The engine
tracks writes as an overlay, which is what makes `reset()` and what-if analysis cheap.

Writing formulas and structural edits
-------------------------------------

These go through the `Workbook`:

```js
import { ALL_FORMULA_CELLS } from "@grid-is/spreadsheet-engine";

const wb = model.getWorkbook("budget.xlsx");
wb.editCell("B5", { f: "=SUM(B1:B4)" });
model.recalculate(ALL_FORMULA_CELLS);   // REQUIRED --- see golden rule 2 in SKILL.md
```

Structural operations (each returns formula-rewrite information and updates references
automatically):

```js
wb.addSheet("Forecast");            wb.removeSheet("Old");
wb.renameSheet("Sheet1", "Data");   wb.copySheet("Data", "Data copy");
wb.insertRows(sheetName, rowIndex, count, below);
wb.insertColumns(sheetName, colIndex, count, toTheRight);
wb.deleteRows(sheetName, rowIndex, count);
wb.moveCells("A1:B4", "D1");
wb.mergeCells(sheetName, "A1:C1");
wb.setColumnWidth(sheetName, col, width);  wb.setRowHeight(sheetName, row, height);
wb.setDefinedName("TaxRate", "=0.24");     // defined names usable in formulas
```

Sheets: `wb.getSheet(name)`, `wb.getSheets()`, `wb.getSheetByIndex(i)`, `sheet.getSize()`,
`sheet.getBounds()`, `sheet.getCells()` (iterator).

Recalculation
-------------

Automatic on value writes. Manual control:

```js
import { ALL_FORMULA_CELLS, CHANGED_ONLY, CHANGED_OR_VOLATILE } from "@grid-is/spreadsheet-engine";
model.recalculate();                      // default pass
model.recalculate(ALL_FORMULA_CELLS);     // everything (after formula edits)
```

`model.iterativeCalculationSettings()` exposes circular-reference settings. Workbooks loaded with
`calcMode: "autoNoTable"` defer expensive data-table cells; recalculate with
`{ includeDeferred: true }` to flush them (see `deferDataTables` JSDoc in the `.d.ts` for the full
semantics).

What-if analysis and goal seek
------------------------------

```js
// What-if: write, read, reset
model.write("Assumptions!B1", 0.05);
const outcome = model.readValue("=Summary!D10");
model.reset();

// Goal seek: find the input value that makes a target cell reach a target value
const requiredGrowth = model.goalSeek("Assumptions!B1", "Summary!D10", 1_000_000);
// returns a number, or a FormulaError if no solution was found
```

Exporting
---------

```js
await wb.toXLSXFile("out.xlsx");                  // Node only
const buf = await wb.toXLSX("arraybuffer");       // browser-friendly ("nodebuffer" in Node)
const jsf = wb.toJSF();                           // JSON representations
const csf = wb.toCSF();
```

Browser download pattern: wrap the ArrayBuffer in a `Blob` with MIME type
`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` and trigger an anchor click.

Fast model persistence without XLSX round-tripping: `serializeModel(model)` /
`deserializeModel(buffer)`.

Errors
------

Formula errors are `FormulaError` values (e.g. `#DIV/0!`, `#NAME?`, `#REF!`, `#VALUE!`, `#N/A`,
`#SPILL!`), not thrown exceptions. They flow through cell values and `runFormula` results. Compare
with the exported constants (`ERROR_DIV0`, `ERROR_NAME`, …) or check `String(value)`. `error.detail`
often carries a specific message. Model-level problems surface in `model.errors` (`ModelError[]`).

Events and introspection
------------------------

```js
model.on("update", () => { /* recalculation happened */ });
model.on("addsheet", (e) => { /* sheet added */ });
```

(Full event map: search `ModelEventArgs` in the `.d.ts`.)

- `functionSignatures()`: machine-readable catalogue of every supported worksheet function (names,
  arguments, descriptions). Useful for building formula editors or validating user input.
- `describeWorkbook(wb)`: structural summary of a workbook (islands of data, etc.).
- `model.analyzeFormula(formula)` / `model.analyzeAndFixFormula(formula)`: parse-level diagnostics
  for user-supplied formulas.
