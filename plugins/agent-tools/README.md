# agent-tools plugin

Claude Code and Codex plugin for [GRID's Agent Tools](https://docs.grid.is/agent-tools).
Runs the published [`@grid-is/agent-tools`](https://www.npmjs.com/package/@grid-is/agent-tools)
MCP server via `npx` — no build step, no bundled dependencies — and ships a
`spreadsheet` skill that teaches agents how to use the tools well.

## Install

```sh
claude plugin marketplace add GRID-is/skills
claude plugin install agent-tools@grid
```

Or from a running session: `/plugin marketplace add GRID-is/skills` then
`/plugin install agent-tools@grid`.

## Install in Codex

```sh
codex plugin marketplace add GRID-is/skills
codex plugin add agent-tools@grid
```

## What you get

- The `grid` MCP server: `loadWorkbook`, `viewRange`, `editCells`,
  `runFormula`, `goalSeek`, `whatIf`, `precedents`, `dependents`,
  `saveWorkbook`, and more — see the [tool reference](https://docs.grid.is/agent-tools).
- The `spreadsheet` skill, loaded automatically when a task involves .xlsx
  files, formulas, or financial models.

## Updating

The server is resolved by `npx` at launch, so new `@grid-is/agent-tools`
releases are picked up on the next MCP server start. Pin a version by changing
the `args` in `.claude-plugin/plugin.json` to `["-y", "@grid-is/agent-tools@x.y.z"]`.
