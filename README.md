# agent-plugins

A multi-agent plugin marketplace (Claude Code, Codex/ChatGPT, Cursor, Grok, OpenCode) for SharpPS framework tooling and knowledge.

## Structure

- **`/plugin`** - plugins developed and maintained here

## Installation

Add this marketplace, then install a plugin from it:

```
/plugin marketplace add sharpps/agent-plugins
/plugin install sharpps-sitecore@sharpps-marketplace
/plugin install sharpps-liferay@sharpps-marketplace
/plugin install sharpps-dotnet@sharpps-marketplace
/plugin install sharpps-work@sharpps-marketplace
/plugin install sharpps-mcp@sharpps-marketplace
/plugin install sharpps-knowledge@sharpps-marketplace
```

or browse for the plugin in `/plugin > Discover`.

### OpenCode

OpenCode has no marketplace/install command - it discovers skills directly from
paths listed in an `opencode.json`. If you're working from inside a clone of
this repo, the root [`opencode.json`](opencode.json) already points at every
plugin's `skills/` tree, so nothing further is needed.

To use these skills from another project, clone this repo somewhere (e.g.
`~/agent-plugins`) and add the paths you want to your own `opencode.json`
(project-local or `~/.config/opencode/opencode.json`):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": [
    "~/agent-plugins/plugin/sharpps-sitecore/skills",
    "~/agent-plugins/plugin/sharpps-liferay/skills",
    "~/agent-plugins/plugin/sharpps-dotnet/skills",
    "~/agent-plugins/plugin/sharpps-work/skills"
  ]
}
```

## Plugins

| Plugin | Description |
|---|---|
| [`sharpps-sitecore`](plugin/sharpps-sitecore) | Sitecore/SXA knowledge for solutions built on the SharpPS framework |
| [`sharpps-liferay`](plugin/sharpps-liferay) | Liferay knowledge and workflow tooling for solutions built on the SharpPS framework |
| [`sharpps-dotnet`](plugin/sharpps-dotnet) | .NET tooling knowledge for solutions built on the SharpPS framework |
| [`sharpps-work`](plugin/sharpps-work) | General office/documentation tooling (screen capture, Word document generation) — not tied to any one platform |
| [`sharpps-mcp`](plugin/sharpps-mcp) | Combined SharpPS remote tools configuration for Replayr, Identity Groups, Chrome DevTools, Shells, Attendance, and Trading. Kept for existing marketplace installs. |
| [`sharpps-knowledge`](plugin/sharpps-knowledge) | Evidence-backed knowledge retrieval, learning drafts, and guarded review over the Knowledge MCP service |


## ChatGPT public-directory packages

Build dedicated ZIPs for the ChatGPT/Codex public plugin directory with:

```bash
./scripts/package-chatgpt.sh
```

The script writes ZIPs to `artifacts/chatgpt/`. It produces four skills-only packages
(Sitecore, Liferay, .NET, and Work) and one package per MCP endpoint (Replayr,
Groups, Browser, Shell, Attendance, Trading, and Knowledge). This split is intentional:
OpenAI currently allows multiple MCP servers to be declared in a package, but only
one MCP server can be connected per plugin in the submission flow. The combined
`sharpps-mcp` configuration remains available for existing marketplace users.

The public-directory packages omit lifecycle hooks. In particular, the Knowledge
package keeps its skills and references but does not include its Codex/Claude hook
configuration or scripts; OpenAI does not accept lifecycle-hook packages in the
public directory. The original source plugin remains unchanged for other hosts.

The generated packages have valid listing metadata and one-server MCP configs, but
they are not submission-complete: each MCP plugin still needs five executed positive
tests, three executed negative tests, a reviewer-accessible demo recording, and any
required reviewer account details entered in the OpenAI dashboard.

## Plugin structure

Each plugin follows the standard Claude Code plugin layout as its canonical source — every other host (Codex, Cursor, Grok, OpenCode) references the same `skills/` tree rather than duplicating it:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata (required)
├── skills/               # Skill definitions
└── README.md             # Documentation
```

## Plugin names are immutable

The `name` field in a marketplace entry is an immutable slug - once a plugin
has been published, its `name` must not change. If a rename is genuinely
unavoidable, add an entry to the top-level `renames` map in
`.claude-plugin/marketplace.json` so existing installs auto-migrate.

## Multi-agent support

Skills are **not** duplicated. Canonical tree:

```text
./plugin/<name>/skills/<skill>/SKILL.md
```

Each host package only references that tree:

```text
./plugin/<name>/
  skills/                       # only copy of SKILL.md
  .claude-plugin/plugin.json
  .codex-plugin/plugin.json     # "skills": "./skills/"
  .cursor-plugin/plugin.json    # "skills": "./skills/"
  .grok-plugin/plugin.json      # "skills": "./skills/"
  .opencode-plugin/plugin.json  # "skills": "./skills/" (reference only, see below)
```

| Host | Entry |
|------|--------|
| Claude Code | `/plugin marketplace add sharpps/agent-plugins` |
| Codex / ChatGPT | `.agents/plugins/marketplace.json` (`path: ./plugin/<name>`) |
| Cursor | `.cursor-plugin/marketplace.json` |
| Grok | per-plugin `.grok-plugin/plugin.json` |
| OpenCode | root [`opencode.json`](opencode.json) `skills` array → `./plugin/<name>/skills` |

Do **not** use `.agents/skills` as a skill base.

Note on OpenCode specifically: OpenCode has no plugin/marketplace-install
concept and never parses a `plugin.json`. Each plugin's `.opencode-plugin/plugin.json`
exists only for documentation/consistency with the other host folders. The
thing OpenCode actually reads is the `skills` array in `opencode.json` (see
[Installation](#installation) above) or its own native skill-discovery paths
(`.opencode/skills/`, `.claude/skills/`, `.agents/skills/`) — we intentionally
don't populate those native paths here, to keep a single canonical skills tree
per the no-duplication rule.

```bash
./scripts/install-agent-skills.sh
```
