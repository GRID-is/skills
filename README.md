GRID skill for spreadsheets
===========================

Official GRID skills for working with spreadsheets in Claude Code, Codex, Cursor, or anywhere
[agent skills](https://agentskills.io/) are supported.

Quick start
-----------

Install this skill:

``` sh
npx skills add GRID-is/skills
```

Alternatively, install the skills from the repo manually:

``` sh
git clone https://github.com/GRID-is/skills.git
cd skills

# Claude Code
mkdir -p ~/.claude/skills
cp -R skills/grid-development ~/.claude/skills/

# Codex
mkdir -p ~/.codex/skills
cp -R skills/grid-development ~/.codex/skills/

# Cursor
mkdir -p ~/.cursor/skills
cp -R skills/grid-development ~/.cursor/skills/
```

Agent Tools plugin
------------------

The skill above teaches agents to *build* with GRID's packages. For agents to
*use* spreadsheets — load, read, edit, and recalculate `.xlsx` files — this repo
also ships an `agent-tools` plugin: the
[`@grid-is/agent-tools`](https://docs.grid.is/agent-tools) MCP server bundled
with a `spreadsheet` skill. The MCP server itself works with any MCP-capable
harness; see the [agent-tools docs](https://docs.grid.is/agent-tools) for
running it standalone.

To install the plugin in Claude Code:

``` sh
claude plugin marketplace add GRID-is/skills
claude plugin install agent-tools@grid
```

To install the plugin in Codex:

``` sh
codex plugin marketplace add GRID-is/skills
codex plugin add agent-tools@grid
```
