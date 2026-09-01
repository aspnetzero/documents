# Research: Consolidating the AI Assistant Configuration

Follow-up on **TODO 1** of [DOCS-SYNC-TODO.md](DOCS-SYNC-TODO.md).

**Question:** the template ships a separate configuration tree for every AI coding assistant. Can these be reduced to a single, vendor-neutral structure that every editor understands?

**Short answer:** yes, for the two layers that carry almost all of the weight — **skills** and **repository instructions**. A vendor-neutral `.agents/skills/` directory is now read natively by Cursor, VS Code / GitHub Copilot, Windsurf and Gemini CLI, and `AGENTS.md` is read natively by all of them too. Claude Code is the one tool that still needs a bridge for both, but the bridge is one symlink layer and one three-line `CLAUDE.md`. **Subagents** have no cross-vendor standard and have to stay duplicated or be dropped.

Rough scale of the win: **206 tracked files / ~680 KB today → around 60 files**, with a single copy of every skill and one instruction file.

> Everything below was checked against vendor documentation in September 2026. Sources are listed at the end. This ecosystem moves fast — re-verify before acting on it.

---

## 1. What is in the repository today

206 tracked files, ~680 KB, across eight directories.

| Layer | Locations | Files |
|---|---|---|
| **Skills** | `.agent/skills/`, `.claude/skills/`, `.cursor/skills/`, `.windsurf/skills/` | 20 skills × 4 = **80** |
| **Commands** | `.agent/commands/`, `.claude/commands/`, `.cursor/commands/`, `.windsurf/workflows/`, `.github/prompts/` | 15 commands × 5 = **75** |
| **Subagents** | `.agent/agents/`, `.claude/agents/`, `.cursor/agents/` | 7 agents × 3 = **21** |
| **Instructions / rules** | `AGENTS.md`, `.agent/rules/`, `.claude/rules/`, `.cursor/rules/`, `.windsurf/rules/`, `.github/copilot-instructions.md` | **6** |
| **MCP config** | `mcp.json`, `.cursor/mcp.json`, `.vscode/mcp.json`, `.gemini/settings.json` | **4** |
| **Claude-only extras** | `.claude/` indexes, flows, guidelines, knowledge | **20** |

### The copies have already drifted

They are not byte-identical, and the differences are not only path self-references:

- Of the 20 skills, only **15–16 are identical** between `.claude/skills/` and each of the other three trees. Four or five have diverged.
- `.agent/commands/add-feature.md` documents which files are created versus modified with `[+]` / `[~]` markers. The `.cursor/` and `.windsurf/` copies of the same command lost those lines.
- Agent definitions differ structurally: `.claude/agents/` uses `tools:`, `skills:` and `keywords:` frontmatter and a "Does / Does NOT" voice, while `.agent/` and `.cursor/` use `model: inherit` and a "You MUST / You MUST NOT" voice.

Five copies of every command is five places to update and five places to forget. That is already happening.

---

## 2. What each tool actually reads

### Repository instructions

| Tool | Reads `AGENTS.md`? | Native file |
|---|---|---|
| Cursor | **Yes**, root and nested | `.cursor/rules/*.mdc` |
| GitHub Copilot / VS Code | **Yes** | `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md` |
| Windsurf / Devin | **Yes**, fed into the same rules engine | `.windsurf/rules/`, `.devin/rules/` |
| Gemini CLI | Not documented | `GEMINI.md` |
| Claude Code | **No** | `CLAUDE.md`, `./.claude/CLAUDE.md`, `.claude/rules/*.md` |

Claude Code's documentation is explicit: *"Claude Code reads `CLAUDE.md`, not `AGENTS.md`. If your repository already uses `AGENTS.md` for other coding agents, create a `CLAUDE.md` that imports it."* The supported bridge is a one-line `@AGENTS.md` import, or a symlink — and the same docs note that on Windows a symlink needs Administrator privileges or Developer Mode, so **the import is the right choice for this template**.

### Skills

The Agent Skills format is an open standard (`agentskills.io`), originally from Anthropic, now supported by roughly 40 products. A skill is a directory with a `SKILL.md` whose frontmatter needs only `name` and `description`. **The specification does not define a discovery directory** — that is left to each client. In practice the clients have converged anyway:

| Tool | Project skill directories |
|---|---|
| Cursor | `.agents/skills/`, `.cursor/skills/`, plus `.claude/skills/` and `.codex/skills/` for compatibility |
| VS Code / Copilot | `.github/skills/`, `.claude/skills/`, `.agents/skills/` (extendable via `chat.agentSkillsLocations`) |
| Windsurf / Cascade | `.windsurf/skills/`, **and** `.agents/skills/` for cross-agent discovery |
| Gemini CLI | `.gemini/skills/` or `.agents/skills/` (the `.agents/` alias takes precedence in the same tier) |
| Claude Code | **`.claude/skills/` only** — but each `<skill-name>` entry may be a symlink to a directory elsewhere on disk |

**`.agents/skills/` is the common denominator for four of the five tools.** The repository's `SKILL.md` files already use the correct `name` + `description` frontmatter, so they are portable as they stand.

### Commands

Both Claude Code and Cursor have **merged commands into skills**:

- Claude Code: *"Custom commands have been merged into skills. A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy`."*
- Cursor: commands and skills now create the same `/slash-command` interface; new ones should go in the skills directory. `disable-model-invocation: true` makes a skill behave like a traditional, explicitly-invoked command.

Windsurf keeps workflows and skills as complementary features — workflows are manual-only, skills can be picked up automatically — and ships a migration wizard from workflows to skills. Copilot prompt files (`.prompt.md`) remain a separate, explicitly-invoked mechanism.

So commands **can** collapse into the skills tree, at the cost of Windsurf losing `/workflow` invocation and Copilot losing its prompt-file palette entry unless thin per-vendor stubs are kept.

### Subagents

There is **no cross-vendor standard**. Claude Code subagents, Cursor agents and Copilot custom agents (`.agent.md`) each have their own frontmatter and semantics. This layer cannot be consolidated today.

### MCP servers

Four copies of the same single `angular-cli` server, in three different schema shapes (`mcpServers` for the root and Cursor, `servers` for VS Code, nested in `settings.json` for Gemini). No universal standard exists yet. These files are 6–8 lines each, so the duplication is cheap — but they should at least be kept in sync deliberately.

---

## 3. Defects found in the current setup

These are worth fixing regardless of whether the consolidation goes ahead.

### 3.1 `AGENTS.md` never reaches Claude Code

There is **no `CLAUDE.md` and no `.claude/CLAUDE.md`** in the repository. Claude Code does not read `AGENTS.md`, so the root instruction file — the one with the `UI-FILTER` markers that the download system uses — is invisible to it. Claude Code sees only `.claude/rules/claude-instructions.md`, which is a separate, hand-maintained copy of the same guidance.

**Fix:** add a `CLAUDE.md` at the root whose first line is `@AGENTS.md`.

### 3.2 `.windsurf/rules/aspnet-zero-abp.md` is too large

The file is **17,236 characters**. Windsurf's documented workflow limit is 12,000 characters per file, and community documentation reports a 6,000-character-per-rule / 12,000-character-total limit for rules. Whatever the exact current figure, a 17 KB rule file is over it, and the overflow is silently dropped.

The same file also lacks the `trigger:` / `description:` YAML frontmatter that Windsurf rules use to decide when to activate. Compare `.cursor/rules/aspnetzero.mdc`, which correctly carries `description` and `alwaysApply: true`.

**Fix:** verify the current limits against Windsurf's own documentation, add the frontmatter, and split or trim the file.

### 3.3 `.agent/` is probably read by nothing

The emerging vendor-neutral convention is `.agents/` (plural). `.agent/` (singular) is not a directory any of the five tools documents. Unless it was created deliberately as an internal source of truth for a generation script, its 43 files are dead weight.

**Fix:** confirm the intent. If it was meant as the neutral tree, rename it to `.agents/` — which immediately makes its skills live in Cursor, Copilot, Windsurf and Gemini CLI.

### 3.4 Windsurf workflows and Copilot prompt files have no frontmatter

`.windsurf/workflows/*.md` and `.github/prompts/*.prompt.md` start straight at `# Generate Entity`. Windsurf workflows need a title and description; Copilot prompt files support `description` / `mode` frontmatter. Without it the entries are harder to discover in the slash palette. Only `.claude/commands/` currently carries proper frontmatter (`description`, `allowed-tools`, `argument-hint`).

---

## 4. Proposed target structure

```
AGENTS.md                       ← single source of truth for repository instructions
CLAUDE.md                       ← one line: @AGENTS.md  (plus Claude-only notes)
mcp.json                        ← MCP servers

.agents/
  skills/                       ← 20 skills, ONE copy
    aspnetzero-entity-patterns/SKILL.md
    ...
    generate-entity/SKILL.md    ← former commands, with disable-model-invocation: true
    ...

.claude/
  skills/  →  symlinks into ../.agents/skills/*     (Claude Code bridge)
  agents/                       ← subagents, Claude-specific frontmatter
  rules/                        ← only genuinely Claude-specific rules

.cursor/
  agents/                       ← subagents, Cursor frontmatter
  rules/aspnetzero.mdc          ← thin: "see AGENTS.md" + Cursor-only constraints
  mcp.json

.windsurf/
  rules/aspnet-zero-abp.md      ← thin, with trigger/description frontmatter, under the size limit

.github/
  copilot-instructions.md       ← thin, points at AGENTS.md
  prompts/                      ← keep only if the prompt palette matters

.vscode/mcp.json
.gemini/settings.json
```

**Roughly 60 files instead of 206**, and every skill, command and rule has exactly one editable copy.

### If symlinks are a problem

`.claude/skills/` symlinks are the weak point on Windows: `git config core.symlinks` must be enabled, and creating them needs Developer Mode. Three fallbacks:

1. **Keep `.claude/skills/` as the real directory** and symlink `.agents/skills/` to it instead — Claude Code gets the real files, and the other four tools follow the link (they run on the same filesystem, so the same Windows constraint applies, but only one link is needed if it is a directory-level link).
2. **Generate instead of link.** Keep `.agents/skills/` as the source and add a small script (`node scripts/sync-agent-config.mjs`) that copies it into the vendor directories, run from a pre-commit hook. The repository already has `.husky/`, so the hook infrastructure exists. Duplication stays on disk but there is only one file to edit, and drift becomes impossible.
3. **Ship only `.agents/skills/`** and document the one-line setting for the tools that need it (VS Code's `chat.agentSkillsLocations`). Claude Code has no equivalent setting today, so its users would lose the skills — probably not acceptable.

Option 2 is the pragmatic choice for a template that is downloaded and unzipped on Windows machines.

---

## 5. Suggested order of work

1. Add `CLAUDE.md` with `@AGENTS.md`. One line, fixes the biggest actual gap. *(Independent of everything else.)*
2. Fix the Windsurf rule file: frontmatter plus size.
3. Decide what `.agent/` is for. Rename to `.agents/` or delete it.
4. Move the 20 skills to `.agents/skills/` as the single source; bridge `.claude/skills/` by symlink or generation script.
5. Convert the 15 commands to skills in the same tree, with `disable-model-invocation: true`. Keep vendor stubs only where the slash palette genuinely matters.
6. Reduce the four instruction files (`.claude/rules/`, `.cursor/rules/`, `.windsurf/rules/`, `.github/copilot-instructions.md`) to thin pointers at `AGENTS.md` plus whatever is genuinely vendor-specific.
7. Leave subagents duplicated; there is no standard to converge on yet.
8. Once the structure settles, document it. Today the whole feature is one line in the 15.2.0 change log.

### Things to watch

- **`UI-FILTER` markers.** `AGENTS.md` uses `UI-FILTER:BEGIN:angular` / `UI-FILTER:END` blocks so the download system can strip irrelevant sections. Any consolidation has to keep those markers working, and any generation script has to run before or after that filtering deliberately, not by accident.
- **Instruction size.** Claude Code recommends keeping `CLAUDE.md` under 200 lines and notes that longer files reduce adherence. `AGENTS.md` is currently 47 lines, which is right; the per-vendor copies are 5–17 KB, which is not. Consolidating on the short file is an improvement in quality, not only in file count.
- **Re-verify before acting.** Cursor, Windsurf and Copilot all changed their skills and commands handling within the last year. Check the vendor docs again at implementation time.

---

## Sources

- [AGENTS.md](https://agents.md/)
- [Agent Skills — Overview](https://agentskills.io/) and [Specification](https://agentskills.io/specification)
- [Claude Code — How Claude remembers your project](https://code.claude.com/docs/en/memory)
- [Claude Code — Extend Claude with skills](https://code.claude.com/docs/en/skills)
- [Cursor — Rules](https://cursor.com/docs/rules)
- [Cursor — Agent Skills](https://cursor.com/docs/context/skills)
- [VS Code — Agent skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [GitHub Docs — Adding repository custom instructions for GitHub Copilot](https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot)
- [Windsurf / Devin — AGENTS.md](https://docs.devin.ai/desktop/cascade/agents-md) and [Workflows](https://docs.devin.ai/desktop/cascade/workflows)
- [Gemini CLI — Skills](https://geminicli.com/docs/cli/skills/)
