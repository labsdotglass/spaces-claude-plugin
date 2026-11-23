# Spaces for Claude

Spaces holds shared lists, notes, a cheat sheet, and people for groups: a grocery list with a spouse, a family
trip plan, a team's contacts. Each person's own assistant reads and updates the same source of truth.

## What is in this plugin

- **Connector**: the Spaces remote MCP server, `https://mcp.joinspaces.ai/mcp` (in `.mcp.json`). Connect it from the
  plugin's **Connectors** tab and sign in with Google the first time.
- **Skill** `spaces`: tells Claude when to use Spaces and how (look up the cheat sheet before stating values, check
  people before adding them, read a list's fields before changing items, undo or restore changes, report bugs with
  expected and actual).

## Use it

Ask things like "add milk to our grocery list", "what's Noa's bedtime?", "who introduced me to Edo?", or "put the
trip plan back to how it was yesterday".

## Change the skill

Bump `version` in `.claude-plugin/plugin.json` whenever the skill changes. Installed copies update only when it does.
`apps/mcp/test/claude-plugin-skill.test.ts` fails if `skills/spaces/references/tools.md` misses a tool the server
registers, or names one it does not.

## Data

Requests go to your Spaces account at mcp.joinspaces.ai, scoped to the spaces you belong to. The plugin itself
stores nothing.

## Build the zip

From the repo root: `bash scripts/zip-claude-plugin.sh` writes `dist/claude-spaces.zip`. Upload it in claude.ai at
**Customize > Plugins > Add > Upload plugin**.

## Source and licence

The plugin is published at https://github.com/labsdotglass/spaces-claude-plugin, under the MIT licence (`LICENSE`). It is
edited in the Spaces repo (`plugins/claude-spaces`) and copied there with `scripts/publish-claude-plugin.sh`.
