`@grid-is/spreadsheet-viewer` reference
=======================================

A React component that renders a `Model` from `@grid-is/spreadsheet-engine` in a canvas-based
viewer: grid, sheet tabs, formula bar, keyboard navigation, selection. It is a *viewer*: users
browse and select; cell editing is done programmatically through the engine.

The dist bundle is self-contained ESM (React runtime included; nothing extra to install).

Basic usage
-----------

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

The container **must have an explicit height**: the viewer fills its parent, and a zero-height
parent renders nothing. In real apps, do the async loading inside an effect or a route loader and
keep the model in state; render the viewer only once the model exists.

Props
-----

| Prop                                   | Type                           | Purpose                                                                 |
| -------------------------------------- | ------------------------------ | ----------------------------------------------------------------------- |
| `model`                                | `Model`                        | The engine model to render (required)                                   |
| `initialSelection`                     | `string`                       | A1-style ref selected on mount, e.g. `"A1"`, `"'My Other Sheet'!B2:E5"` |
| `onChange`                             | `(event: ViewerEvent) => void` | Selection and sheet-change notifications                                |
| `theme`                                | object                         | CSS custom properties (see Theming)                                     |
| `fontConfig`                           | object                         | Font configuration                                                      |
| `showErrorTooltips`                    | `boolean`                      | Tooltips on error cells                                                 |
| `showFormulaReferencesOnCellSelection` | `boolean`                      | Highlight a selected formula's referenced ranges                        |

For precise types beyond this list, check `node_modules/@grid-is/spreadsheet-viewer/dist/index.d.ts`
(and note that some prop types come from GRID's commercial docs at
<https://docs.grid.is/mondrian-react/>. Examples there import from `@grid-is/mondrian-react`;
substitute `@grid-is/spreadsheet-viewer`).

Events
------

```tsx
import type { ViewerEvent } from "@grid-is/spreadsheet-viewer";

const handleChange = (event: ViewerEvent) => {
  if (event.type === "selection-change") {
    // event.selection: string (A1-style), event.timestamp: number
  }
  if (event.type === "sheet-change") {
    // event.sheetName, event.previousSheetName, event.timestamp
  }
};

<SpreadsheetViewer model={model} onChange={handleChange} />
```

A common pattern: on `selection-change`, read details from the engine and show them elsewhere in the
UI:

```tsx
if (event.type === "selection-change") {
  const cell = model.readCell(`=${event.selection.split(":")[0]}`);
  setInspector({ value: cell.v, formula: cell.f, format: cell.z });
}
```

Reflecting engine changes
-------------------------

The viewer renders the live model. After programmatic writes (`model.write(...)`), the engine
recalculates and the viewer reflects the new values. There is no separate "refresh" API to call. If
results don't appear after a *formula* edit, the missing step is almost always
`model.recalculate(ALL_FORMULA_CELLS)` (see the engine reference).

Theming
-------

The `theme` prop takes `--mondrian-*` CSS custom properties. Known variables (from the shipped
stylesheet):

```text
--mondrian-bg-white            --mondrian-bg-gray-*
--mondrian-text-black          --mondrian-text-gray-*
--mondrian-border-primary      --mondrian-border-separator
--mondrian-border-radius       --mondrian-border-radius-sm
--mondrian-highlight-stroke    --mondrian-highlight-fill
--mondrian-hover-overlay       --mondrian-resizer-hover
--mondrian-error-red
--mondrian-syntax-number  --mondrian-syntax-string
--mondrian-syntax-range   --mondrian-syntax-prefix
```

```tsx
<SpreadsheetViewer
  model={model}
  theme={{
    "--mondrian-highlight-stroke": "#3b82f6",
    "--mondrian-highlight-fill": "#dbeafe",
    "--mondrian-border-radius": "4px",
  }}
/>
```

Full theme-property documentation: <https://docs.grid.is/mondrian-react/reference/theme-properties/>
