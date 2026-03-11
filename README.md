# Mobiik Agent Skills

Agent Plugin for VS Code and GitHub Copilot CLI that provides a curated
collection of agent skills for AI-assisted development workflows.

## Available skills

| Skill | Description |
|---|---|
| **agent-blueprints** | Reference specification with templates for the 5-agent + 12-skill + 3-rule framework |

## Installation

### Option 1: VS Code marketplace (recommended for teams)

Add the marketplace source to your VS Code `settings.json`:

```json
{
  "chat.plugins.enabled": true,
  "chat.plugins.marketplaces": ["mobiik/docs-aidd-agent-skills"]
}
```

Then open the Extensions view (`Ctrl+Shift+X`), search for `@agentPlugins`, and
install **mobiik-agent-skills**.

### Option 2: GitHub Copilot CLI

```bash
copilot plugin marketplace add mobiik/docs-aidd-agent-skills
copilot plugin install mobiik-agent-skills@mobiik-copilot-marketplace
```

Or install directly from the repository:

```bash
copilot plugin install mobiik/docs-aidd-agent-skills
```

### Option 3: Local path

Clone the repo and register the plugin locally:

```json
{
  "chat.plugins.paths": {
    "/path/to/docs-aidd-agent-skills": true
  }
}
```

## Usage

Skills load automatically based on context. Each skill activates when the agent
detects relevant keywords or tasks described in the skill's trigger conditions.

## Plugin structure

```
docs-aidd-agent-skills/
├── plugin.json                        # Plugin manifest
├── .github/
│   └── plugin/
│       └── marketplace.json           # Marketplace definition
├── skills/
│   └── agent-blueprints/
│       ├── SKILL.md                   # Skill: framework specification
│       └── references/
│           └── stacks.md              # Stack-specific adaptations
├── CHANGELOG.md
└── LICENSE
```

## Compatibility

| Platform | Support |
|---|---|
| VS Code + GitHub Copilot (v1.110+) | Native via Agent Plugin system |
| GitHub Copilot CLI | `copilot plugin install` |
| Cursor | Reads `.claude/` output at project level |
| Claude Code | Native `.claude/` format |
| Codex, Amp, OpenCode | `.claude/` compatible |

## License

[MIT](LICENSE)
