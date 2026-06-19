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
