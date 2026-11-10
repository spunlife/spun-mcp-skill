# Spun MCP skill

An agent skill for AI clients that use the Spun MCP server (`https://api.spun.com/api/mcp`). It tells the
agent how to sign in, which access level allows what, how a send waits for a person to
approve it, how to answer "did it send" from the delivery status instead of guessing, and
how to stay inside the read allowance and the Free MCP limits.

Spun is a hosted MCP server on your own WhatsApp number: not a browser extension, not a
customer-facing bot, no Business API, no API key for ChatGPT or claude.ai.

Spun is an independent product and is not affiliated with or endorsed by WhatsApp or Meta.

## What is in this repository

| Path | What it is |
| :- | :- |
| `spun-mcp/SKILL.md` | The skill itself, the same file Spun serves at https://spun.com/mcp/SKILL.md. |
| `spun-mcp/mcp.json` | The Spun MCP server for Amp, which reads an `mcp.json` next to a skill's `SKILL.md`. |
| `spun-mcp/.claude-plugin/plugin.json` | Claude Code plugin manifest: the skill plus the Spun MCP server. |
| `.claude-plugin/marketplace.json` | Claude Code marketplace listing that one plugin. |

Every file is text or JSON. There are no hooks, scripts or other executable code, and no API
key or token: you sign in to Spun in your browser when your client first connects.

## Access levels

On the consent screen an organization manager picks one of three access levels for the
connection. Read-only analyst reads and can never send. Assistant (no sends) is an
assistant that can never send. Assistant with sends can propose messages, and each one waits
for a person to approve it in the Spun inbox unless the organization has opted into
autonomous sending for that app. How each level works, and what Free MCP includes, is on
https://spun.com/mcp/ (Free MCP limits: https://spun.com/mcp/#free-mcp).

## Install

You need a Spun account with a WhatsApp number linked. Each install line below is taken from
the client's own documentation, read on 10 November 2026.

### Claude Code (plugin)

From Claude Code documentation, read 2026-11-10 (https://code.claude.com/docs/en/plugin-marketplaces):
`claude plugin marketplace add <owner>/<repo>` adds a GitHub repository as a marketplace,
and `/plugin install my-first-plugin@my-marketplace` installs a plugin from it. For this
repository:

```
/plugin marketplace add spunlife/spun-mcp-skill
/plugin install spun-mcp@spun
```

The plugin brings the skill and the Spun MCP server. To sign in, run `/mcp`, choose
`spun` and Authenticate: Claude Code opens the Spun sign-in page in your browser.

### Claude Code (personal skill only)

From Claude Code documentation, read 2026-11-10 (https://code.claude.com/docs/en/skills): a personal
skill lives at `~/.claude/skills/<skill-name>/SKILL.md`. Copy `spun-mcp/SKILL.md` from this
repository to `~/.claude/skills/spun-mcp/SKILL.md`. This adds the guidance only; connect the server
separately with `claude mcp add --transport http spun https://api.spun.com/api/mcp`.

### Amp

From Amp documentation, read 2026-11-10 (https://ampcode.com/docs/customize/skills): "For direct shell
installation from a GitHub repository, Git URL, or local path, run `amp skill add <source>`."
For this repository:

```
amp skill add https://github.com/spunlife/spun-mcp-skill
```

The skill's `mcp.json` names the Spun MCP server. The Amp documentation says a skill's MCP
tools stay hidden until the skill is loaded, and that for a server address Amp opens the
server's sign-in page in your browser (https://ampcode.com/docs/customize/mcp).

### claude.ai

From the Claude Help Center, read 2026-11-10 (https://support.claude.com/en/articles/12512180-using-skills-in-claude): "Package your
skill folder as a ZIP file", then upload it under Customize > Skills. This repository does
not publish a ZIP; the folder to package is `spun-mcp/`.

### ChatGPT

ChatGPT does not load skill files. The same guidance reaches it, and every other client,
through the instructions the Spun MCP server sends when a client connects.

## Update and remove

- Claude Code plugin: from Claude Code documentation, read 2026-11-10
  (https://code.claude.com/docs/en/skills), uninstall with `/plugin uninstall <plugin-name>@<marketplace-name>`,
  here `/plugin uninstall spun-mcp@spun`. `claude plugin marketplace remove spun` "removes the
  marketplace and uninstalls its plugins" (https://code.claude.com/docs/en/plugin-marketplaces).
- Claude Code personal skill: "delete the skill's directory, `~/.claude/skills/<skill-name>/`"
  (https://code.claude.com/docs/en/skills). To update, copy the new `spun-mcp/SKILL.md` over it.
- Amp: the Amp documentation read on 2026-11-10 (https://ampcode.com/docs/customize/skills) says "The
  `amp skill import` and `amp skill update` commands manage shared imports" without giving
  their syntax, and names no remove command. `amp skills list` shows what is installed.
- Claude Code plugin, update: the pages read do not document an update command. The remove
  and install commands above can be run again.
- claude.ai: from the Claude Help Center, read 2026-11-10 (https://support.claude.com/en/articles/12512180-using-skills-in-claude),
  "To remove a custom skill you've uploaded": "Navigate to Customize > Skills.", "Find the
  skill in your skills list and click on it to view the details.", "Use the toggle in the
  upper right corner to disable the skill.", then "click the "..." button next to the toggle,
  then select "Delete"" and "Click "Delete" in the confirmation prompt." The article adds:
  "If you change your mind, you can add the skill again by re-uploading the file." It does
  not describe updating an uploaded skill in place.

The skill file changes when Spun's tools change. The current copy is always at
https://spun.com/mcp/SKILL.md.

## More

- Install guide: https://spun.com/guides/install-spun-skill/
- Spun MCP: https://spun.com/mcp/
- Repository: https://github.com/spunlife/spun-mcp-skill (owner `spunlife`, repository `spun-mcp-skill`)

Licensed under the MIT License, see `LICENSE`.
