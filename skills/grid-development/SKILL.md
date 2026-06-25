---
name: grid-development
description: Building spreadsheet-based applications using GRID's spreadsheet packages (@grid-is/spreadsheet-engine, @grid-is/spreadsheet-viewer, @grid-is/spreadsheet-editor). Use this skill whenever a task or project involves loading, evaluating, editing, displaying Excel/XLSX workbooks programmatically, running formulas, reading or writing cells, recalculation, what-if analysis, goal seek, exporting XLSX, or embedding a spreadsheet view or editable spreadsheet UI in a React app. Consult it even if the user only says "spreadsheet", "Excel file", "workbook", or "formula".
compatibility: JavaScript and TypeScript projects for use in the browser or in Node, Deno, and Bun.
---

GRID spreadsheet packages
=========================

Three complementary packages from GRID (grid.is):

- **`@grid-is/spreadsheet-engine`**: headless spreadsheet engine. Loads XLSX/CSF/JSF workbooks or
  creates blank ones, evaluates Excel formulas, reads/writes cells with full dependency-graph
  recalculation. Works in Node and browsers. No React required.
- **`@grid-is/spreadsheet-viewer`**: React component (`SpreadsheetViewer`) that renders an engine
  `Model` in a canvas-based viewer with sheet tabs, a formula bar, and cell navigation. Read-only:
  users browse and select; changes are made programmatically through the engine.
- **`@grid-is/spreadsheet-editor`**: React component (`SpreadsheetEditor`) that renders an engine
  `Model` as a fully editable spreadsheet: users type values and formulas, format cells, fill,
  paste, insert/delete/move rows and columns, and manage sheets directly in the UI.

The engine is the foundation; the viewer and editor consume engine models. For calculation-only
tasks (servers, scripts, tests), use the engine alone. For display with programmatic updates, use
the viewer. Reach for the editor only when end users must edit cells themselves.

Which reference to read
-----------------------

- Engine work (loading, formulas, reading/writing cells, recalc, XLSX export, events): read
  `engine.md`
- Viewer work (React integration, props, events, theming): read `viewer.md`
- Editor work (editable UI, edit events, controller, size limits, fonts): read `editor.md`
- Anything not covered there: **grep the bundled type definitions** — they are the authoritative,
  fully documented API surface, already in the consumer's repo:
  - `node_modules/@grid-is/spreadsheet-engine/dist/index.d.ts` (~10,200 lines, rich JSDoc with
    examples)
  - `node_modules/@grid-is/spreadsheet-viewer/dist/index.d.ts` (small)
  - `node_modules/@grid-is/spreadsheet-editor/dist/index.d.ts` (small)

  Example: `grep -n "goalSeek\|insertRows\|describeWorkbook" node_modules/@grid-is/spreadsheet-engine/dist/index.d.ts` then view the surrounding lines. Never
  invent method names — if it's not in the `.d.ts`, it doesn't exist.

Golden rules (the things that break first)
------------------------------------------

1. **Await `Model.preconditions` once before using the engine.** The formula parser loads
   asynchronously; most `Model` entry points throw if it hasn't resolved.

   ```js
   import { Model } from "@grid-is/spreadsheet-engine";
   await Model.preconditions;
   ```

2. **Value writes recalculate automatically; formula writes do not.** `model.write("B2", 42)`
   triggers recalculation of dependents. But writing a *formula* (
   `workbook.editCell("B5", { f: "=SUM(A1:A4)" })`) leaves the cell `null` until you rebuild and
   recalculate:

   ```js
   import { ALL_FORMULA_CELLS } from "@grid-is/spreadsheet-engine";
   wb.editCell("B5", { f: "=SUM(A1:A4)" });
   model.recalculate(ALL_FORMULA_CELLS);   // now B5 has a value
   ```

   After this, subsequent value writes propagate to the new formula normally.

3. **Read expressions start with `=`; write references don't.** `model.readValue("=B2")` but
   `model.write("B2", 42)`. The read methods accept any formula expression, not just references.

4. **`Model.fromXLSXFile` and `toXLSXFile` are Node-only.** In browsers use
   `Model.fromXLSX(arrayBuffer, filename)` and `workbook.toXLSX("arraybuffer")`.

5. **All three packages are ESM-only** (`"type": "module"`). Use `import`, not `require`. The
   **engine** dist bundle is self-contained (no runtime dependencies to install alongside). The
   **viewer and editor are React components** and need `react` and `react-dom` (version 18 or
   later) as peer dependencies your project must provide; the **editor additionally peer-depends on
   `@grid-is/spreadsheet-engine`**.

6. **Import the component stylesheet.** Without it the app works but the formula bar is unstyled
   and there are other visual bugs. The correct import path (using the package's declared `exports`
   map) is:

   ```js
   import "@grid-is/spreadsheet-viewer/style.css";
   // or, for the editor:
   import "@grid-is/spreadsheet-editor/style.css";
   ```

   Add this once, at your entry point, before the component is rendered. `dist/index.css` is the
   physical file but it is not in the `exports` map and bundlers such as Vite will reject it with a
   500 error.

7. **The viewer's and editor's container needs an explicit height.** `SpreadsheetViewer` and
   `SpreadsheetEditor` fill their parent; a parent with no height renders nothing visible.

8. **Sheet references**: unprefixed refs (`A1`, `D:F`) hit the first sheet. Use `Sheet2!A1`, and
   single-quote names containing spaces: `'My Sheet'!A1`.

Minimal end-to-end example
--------------------------

```tsx
import "@grid-is/spreadsheet-viewer/style.css";
import { Model } from "@grid-is/spreadsheet-engine";
import { SpreadsheetViewer } from "@grid-is/spreadsheet-viewer";

await Model.preconditions;
const res = await fetch("/budget.xlsx");
const model = await Model.fromXLSX(await res.arrayBuffer(), "budget.xlsx");

function App() {
  return (
    <div style={{ height: "600px" }}>
      <SpreadsheetViewer model={model} />
    </div>
  );
}
```

For an editable spreadsheet, swap `SpreadsheetViewer` for `SpreadsheetEditor` (and the stylesheet
import for `@grid-is/spreadsheet-editor/style.css`) — the rest is identical. See `editor.md`.

Other useful packages
---------------------

These packages complement the core engine/viewer/editor stack and can be installed from npmjs.com.

- **numfmt**: formats a raw cell value using an Excel number-format specifier string (e.g.
  `"#,##0.00"`, `"dd/mm/yyyy"`). Use it when you read a value from the engine and need to display
  it exactly as Excel would, outside of the viewer/editor UI.

  ```js
  import numfmt from "numfmt";
  numfmt(1234.5, "#,##0.00"); // → "1,234.50"
  ```

- **@borgar/xlsx-convert**: converts an Excel XLSX file to JSF (a JSON spreadsheet format). Useful
  when you want to inspect or manipulate workbook structure as plain JSON before loading it into the
  engine, or when you need a serialisable, human-readable snapshot of the file.

- **@jsfkit/types**: TypeScript types for JSF. Only needed if your code constructs or inspects JSF
  objects directly (e.g. after converting with `@borgar/xlsx-convert`, or when building workbooks
  from JSON rather than from an XLSX file).

Licensing and telemetry (tell the user when relevant)
-----------------------------------------------------

The publicly installable packages are **evaluation versions** under the
[GRID Evaluation Licence](https://docs.grid.is/evaluation-license/): free for internal evaluation,
prototypes, and proofs of concept in non-production environments. Production or commercial use
requires a commercial licence (<https://grid.is/license>). The evaluation builds print a licence
notice on load and send anonymous telemetry (package/runtime metadata only — never spreadsheet data)
to PostHog; telemetry must not be removed or suppressed, and it is absent from licensed builds. If
the user is building something destined for production, remind them of this — don't attempt to
disable the notice or telemetry.
