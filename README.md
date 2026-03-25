# agentskills-context-injection

[![Blackwell Systems™](https://raw.githubusercontent.com/blackwell-systems/blackwell-docs-theme/main/badge-trademark.svg)](https://github.com/blackwell-systems)
[![Agent Skills](assets/badge-agentskills.svg)](https://agentskills.io)

The [Agent Skills](https://agentskills.io) spec tells skills to put reference material in `references/` and let the model load it "when needed." In practice, the model forgets, loads the wrong file, or loads everything upfront. There's no enforcement.

This repo fixes that. Skills declare which references to load for which prompts, and a script handles the rest.

> Uses existing Agent Skills conventions (`scripts/`, `references/`, `metadata:`). No spec changes. Works with Claude Code, Cursor, GitHub Copilot, and anything else that supports the spec.

## How it works

Skills add `triggers:` to their YAML frontmatter. Each trigger maps a regex to a reference file. When the user's prompt matches, the reference loads automatically.

Two layers, same trigger definitions:

**Layer 1 -- Script (any agent).** A `scripts/inject-context` script ships with the skill. The model runs it before executing:

```bash
bash scripts/inject-context "/saw program execute add caching"
# outputs contents of references/program-flow.md
```

One line in `SKILL.md`: "run `scripts/inject-context` with the user's prompt before proceeding." Simpler than a routing table, harder to skip.

**Layer 2 -- Hook (Claude Code).** A `UserPromptSubmit` hook runs the same script before the model sees the prompt. No model decision. The reference is in context when the model starts. Deterministic.

## Trigger Definitions

Skills declare triggers in YAML frontmatter using the `triggers:` field (via the spec's `metadata:` extension point):

```yaml
---
name: my-skill
description: Does things
triggers:
  - match: "^/my-skill subcommand"
    inject: references/subcommand-flow.md
  - match: "error|failed|blocked"
    inject: references/troubleshooting.md
---
```

- `match`: regex pattern tested against the full prompt text
- `inject`: path relative to the skill directory
- Multiple matches -> all matching references injected (concatenated)
- No match -> no injection, zero overhead

**Important: dispatch-time triggers only.** The Claude Code `UserPromptSubmit` hook receives the prompt *after* skill body expansion -- the full `SKILL.md` content is in the prompt, not just what the user typed. This means keyword triggers like `failure|blocked` will match against the skill's own instructions and fire on every invocation. Use anchored patterns that can't appear in the skill body (e.g. `^/saw program`, `^/saw amend`). Mid-execution references that depend on runtime state (like failure routing after agents report back) should stay convention-based -- the hook fires too early for them.

## Installation

### The injection script (any skill)

Copy `scripts/inject-context` into your skill's `scripts/` directory:

```bash
cp scripts/inject-context ~/.agents/skills/my-skill/scripts/
chmod +x ~/.agents/skills/my-skill/scripts/inject-context
```

Add `triggers:` to your skill's frontmatter, and add this to your `SKILL.md` instructions:

```markdown
Before executing any subcommand, run:
  bash scripts/inject-context "<user prompt>"
and incorporate the output as context.
```

### The Claude Code hook (optional, Claude Code only)

```bash
# Install the hook script
cp hooks/inject_skill_context ~/.local/bin/
chmod +x ~/.local/bin/inject_skill_context
```

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "inject_skill_context"
          }
        ]
      }
    ]
  }
}
```

The hook iterates all skill directories (`~/.claude/skills/`, `~/.agents/skills/`) and delegates to each skill's `scripts/inject-context`. Adding a new skill requires zero hook changes.

## Redundancy Model

All three layers are active simultaneously:

| Layer | Mechanism | Vendor-neutral | Enforcement |
|-------|-----------|----------------|-------------|
| Hook | `UserPromptSubmit` | Claude Code only | Deterministic (pre-model) |
| Script | `scripts/inject-context` | Any agent with Bash | Model-initiated |
| Fallback | Routing table in SKILL.md | Any agent | Convention-based |

Users get the best available layer. No regression at any level.

## Spec Alignment

This project uses only conventions the Agent Skills spec already defines:
- `scripts/` directory for executable code ([spec](https://agentskills.io/skill-creation/using-scripts))
- `references/` directory for on-demand content ([spec](https://agentskills.io/specification#references))
- Frontmatter extensibility for custom fields ([spec](https://agentskills.io/specification#metadata-field))

The `triggers:` field is skill-specific metadata using the spec's existing extensibility. Agents that don't understand `triggers:` simply ignore it.

## The Four-Tier Model

The Agent Skills spec defines three tiers. This project documents a fourth — **Tier 0 (Discovery)** — that sits outside the skill in project config files (`CLAUDE.md`, `.cursorrules`, etc.):

| Tier | What | When loaded |
|------|------|-------------|
| 0 — Discovery | Skill index in project config | Session start (always in context) |
| 1 — Metadata | Name + description from frontmatter | Session start (catalog) |
| 2 — Instructions | Full SKILL.md body | Skill activation |
| 3 — Resources | Reference files via triggers | **Context injection** (this project) |

See [`docs/tier-0-discovery.md`](docs/tier-0-discovery.md) for the full Tier 0 pattern.

## Example

See [`examples/saw/`](examples/saw/) for the Scout-and-Wave skill's trigger configuration.

## License

MIT
