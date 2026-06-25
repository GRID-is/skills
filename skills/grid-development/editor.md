`@grid-is/spreadsheet-editor` reference
=======================================

A React component that renders a `Model` from `@grid-is/spreadsheet-engine` as a fully *editable*
spreadsheet: grid, sheet tabs, formula bar, keyboard navigation, and direct cell editing. Users can
type values and formulas, format cells, fill, paste, clear, insert/delete/move rows and columns,
resize rows and columns, and add/delete/rename sheets. For read-only browsing, use
`@grid-is/spreadsheet-viewer` instead.

Unlike the engine (self-contained) and the viewer (which needs only `react` and `react-dom`), the
editor's dist bundle has **three** peer dependencies your project must provide: `react` and
`react-dom` (version 18 or later), and `@grid-is/spreadsheet-engine`.

Basic usage
-----------

```tsx
import "@grid-is/spreadsheet-editor/style.css";
import { Model } from "@grid-is/spreadsheet-engine";
import { SpreadsheetEditor } from "@grid-is/spreadsheet-editor";

await Model.preconditions;
const res = await fetch("/budget.xlsx");
const model = await Model.fromXLSX(await res.arrayBuffer(), "budget.xlsx");

function App() {
  return (
    <div style={{ height: "600px" }}>
      <SpreadsheetEditor model={model} />
    </div>
  );
}
```

The container **must have an explicit height**: the editor fills its parent, and a zero-height
parent renders nothing. In real apps, do the async loading inside an effect or a route loader and
keep the model in state; render the editor only once the model exists.

Props
-----

| Prop                                   | Type                               | Purpose                                                                 |
| -------------------------------------- | ---------------------------------- | ----------------------------------------------------------------------- |
| `model`                                | `Model`                            | The engine model to render and edit (required)                          |
| `onChange`                             | `(event: EditorEvent) => void`     | Notifications of edits, selection, and sheet changes (see Events)       |
| `initialSelection`                     | `string`                           | A1-style ref selected on mount, e.g. `"A1"`, `"'My Other Sheet'!B2:E5"` |
| `size`                                 | `SpreadsheetSize`                  | Caps the navigable grid (see Grid size limits)                          |
| `controllerRef`                        | `Ref<SpreadsheetEditorController>` | Imperative handle (experimental; see Controller)                        |
| `fontConfig`                           | `FontConfig`                       | Font loading configuration (see Fonts)                                  |
| `showErrorTooltips`                    | `boolean`                          | Tooltips on error cells                                                 |
| `showFormulaReferencesOnCellSelection` | `boolean`                          | Highlight a selected formula's referenced ranges                        |

Note there is **no `theme` prop**: unlike the viewer, the editor does not currently accept CSS
custom properties. For precise types, check
`node_modules/@grid-is/spreadsheet-editor/dist/index.d.ts` (small, fully documented). The
commercial documentation at <https://docs.grid.is/editor/> applies too; examples there import from
`@grid-is/editor-react` and `@grid-is/apiary` — substitute `@grid-is/spreadsheet-editor` and
`@grid-is/spreadsheet-engine` respectively.

Events
------

`onChange` receives an `EditorEvent`, a discriminated union covering the viewer's navigation events
(`selection-change`, `sheet-change` — same shapes as `ViewerEvent`) plus one event per kind of
edit. Every event carries a `timestamp` (Unix-epoch milliseconds). The editing events and their
fields beyond `type` and `timestamp`:

| `type`           | Fields                                                  |
| ---------------- | ------------------------------------------------------- |
| `write-cell`     | `sheetName`, `cellId`, `value`                          |
| `paste`          | `sheetName`, `range`                                    |
| `clear-cells`    | `sheetName`, `range`                                    |
| `format-cells`   | `sheetName`, `range`, `format` (a JSF `Style`)          |
| `fill`           | `sheetName`, `sourceRange`, `targetRange`               |
| `insert-row`     | `sheetName`, `row`, `direction` (`"top"`/`"bottom"`)    |
| `insert-column`  | `sheetName`, `column`, `direction` (`"left"`/`"right"`) |
| `delete-rows`    | `sheetName`, `startRow`, `count`                        |
| `delete-columns` | `sheetName`, `startColumn`, `count`                     |
| `move-rows`      | `sheetName`, `fromRange`, `toRange`                     |
| `move-columns`   | `sheetName`, `fromRange`, `toRange`                     |
| `move-cells`     | `sheetName`, `fromRange`, `toRange`                     |
| `resize-row`     | `sheetName`, `row`, `height`                            |
| `resize-column`  | `sheetName`, `column`, `width`                          |
| `add-sheet`      | `sheetName`                                             |
| `delete-sheet`   | `sheetName`                                             |
| `rename-sheet`   | `oldName`, `newName`                                    |

Events are after-the-fact notifications: by the time `onChange` fires, the editor has already
applied the change to the live model, so don't re-apply it yourself. Use events for autosave,
dirty-state tracking, audit logs, or syncing other UI:

```tsx
import type { EditorEvent } from "@grid-is/spreadsheet-editor";

const handleChange = (event: EditorEvent) => {
  if (event.type === "selection-change" || event.type === "sheet-change") {
    return; // navigation only — nothing changed in the workbook
  }
  scheduleAutosave(); // any other event mutated the workbook
};

<SpreadsheetEditor model={model} onChange={handleChange} />
```

To persist edits, export through the engine as usual (`workbook.toXLSX("arraybuffer")`,
`serializeModel(model)`, …) — see `engine.md`.

Reflecting engine changes
-------------------------

The editor renders the live model, so programmatic engine writes (`model.write(...)`) appear in the
editor just as they do in the viewer, and user edits made in the editor are immediately readable
through the engine (`model.readValue(...)`). If results don't appear after a *programmatic formula*
edit, the missing step is almost always `model.recalculate(ALL_FORMULA_CELLS)` (see the engine
reference) — formulas typed by users into the editor itself don't need this.

Controller (experimental)
-------------------------

`controllerRef` exposes an imperative handle. The interface is marked `@experimental` and may
change without notice:

```tsx
import { useRef } from "react";
import type { SpreadsheetEditorController } from "@grid-is/spreadsheet-editor";

const controller = useRef<SpreadsheetEditorController>(null);

<SpreadsheetEditor model={model} controllerRef={controller} />

controller.current?.selectSheet("Forecast");      // switch the active sheet
controller.current?.selectCells("B2:D5");         // move the selection
controller.current?.highlightRefs(["A1:A10", "C3"], 2000);  // flash ranges (duration in ms)
```

Grid size limits
----------------

The `size` prop caps how far users can navigate. Values are **0-based indices of the last
navigable row/column**, so `{ maxRows: 99, maxCols: 25 }` allows 100 rows and 26 columns (A–Z).
Values beyond the engine's Excel-convention maximums are silently capped; omit a property to leave
it unconstrained.

```tsx
<SpreadsheetEditor model={model} size={{ maxRows: 99, maxCols: 25 }} />
```

Fonts
-----

The editor ships definitions for 20 spreadsheet fonts (Arial, Calibri, Aptos, Times New Roman, …),
each tagged with an `"open"` or `"restricted"` licence. `fontConfig` controls which are loaded and
from where:

```tsx
<SpreadsheetEditor
  model={model}
  fontConfig={{
    fontFilter: "open",            // only open-licence fonts; or "restricted", or an array of
                                   // font ids such as ["arial", "calibri"]
    baseUrl: "https://fonts.example.com/",  // self-host; can also be { open, restricted } URLs
  }}
/>
```

The full font list and the `FontConfig`, `FontFilter`, and `BaseURL` types are in the package's
`dist/index.d.ts`.
