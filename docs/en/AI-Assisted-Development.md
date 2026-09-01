# AI-Assisted Development

Every ASP.NET Zero solution ships with a configuration set for AI coding
assistants. It teaches the assistant the layering rules of the solution, the
patterns it should follow, and the repeatable procedures it can run — so that
generated code lands in the right project, uses the right base classes, and
respects permissions, localization and multi-tenancy without being told every
time.

The configuration works with **Claude Code, Cursor, Windsurf, GitHub Copilot /
VS Code and Gemini CLI**. There is nothing to install: open the solution folder
in your assistant of choice and it picks the configuration up.

## What Ships

| Location | Purpose |
|----------|---------|
| `AGENTS.md` | Repository-wide instructions. The first thing every assistant reads. |
| `CLAUDE.md` | Imports `AGENTS.md` for Claude Code, which reads `CLAUDE.md` rather than `AGENTS.md`. |
| `GEMINI.md` | The same import for Gemini CLI, which reads `GEMINI.md`. Generated. |
| `.agents/` | **The single source** for everything below. |
| `mcp.json` | MCP server configuration. |

Inside `.agents/`:

| Directory | Contents |
|-----------|----------|
| `skills/` | Reusable implementation patterns in the [Agent Skills](https://agentskills.io/) format — entities, application services, DTOs, permissions, localization, EF Core, Mapperly, error handling, xUnit testing, and the patterns of your project's front end. |
| `commands/` | Repeatable procedures such as generating an entity, an application service, a migration, a permission set, a CRUD page, or running a pre-push review. |
| `agents/` | Specialised subagents: `aspnetzero-developer`, `backend-architect`, `code-reviewer`, `qa-engineer`, `debugger`, and the front-end developer agent for your project type. |
| `rules/` | The detailed architecture and layering rules of the solution. |

## How Each Assistant Sees It

`.agents/skills/` is a vendor-neutral directory that **Cursor, GitHub Copilot /
VS Code, Windsurf and Gemini CLI all read directly**. Claude Code discovers
skills only from `.claude/skills/`, so that one copy is generated.

The same applies to commands and rules: each assistant has its own directory and
its own frontmatter, so the vendor-specific files are generated from `.agents/`:

| Source | Generated into |
|--------|----------------|
| `.agents/skills/` | `.claude/skills/` |
| `.agents/commands/` | `.claude/commands/`, `.cursor/commands/`, `.windsurf/workflows/`, `.github/prompts/` |
| `.agents/agents/` | `.claude/agents/`, `.cursor/agents/` |
| `.agents/rules/` | `.claude/rules/`, `.cursor/rules/`, `.windsurf/rules/`, `.github/copilot-instructions.md`, `GEMINI.md` |
| `mcp.json` | `.mcp.json`, `.cursor/mcp.json`, `.vscode/mcp.json`, `.gemini/settings.json` |

The script also writes `.claude/SKILL-INDEX.md`, `.claude/COMMAND-INDEX.md` and
`.claude/AGENT-INDEX.md`, so Claude Code can see everything that is available at
a glance.

Every generated file starts with a `GENERATED FILE` comment naming its source.

## Customizing It

The configuration is yours to change — it describes *your* solution, so extend
it as your project grows.

1. Edit the file under `.agents/`.
2. Regenerate the assistant directories:

```bash
npm run sync-agent-config
```

3. Commit the source change together with the generated output.

A pre-commit hook runs `npm run sync-agent-config:check` and fails the commit if
the generated files are out of date, so the copies cannot drift apart.

> Never edit a generated file directly — anything under `.claude/`, `.cursor/`,
> `.windsurf/`, `.github/prompts/`, plus `.github/copilot-instructions.md`,
> `GEMINI.md`, `.mcp.json`, `.vscode/mcp.json` and `.gemini/settings.json`. The
> next run of the sync script overwrites it. Every one of them carries the
> `GENERATED FILE` comment at the top as a reminder.

### Adding a Skill

A skill is a directory with a `SKILL.md`. Only `name` and `description` are
required; the name must be lowercase with hyphens and match the directory name.

```
.agents/skills/invoice-patterns/SKILL.md
```

```markdown
---
name: invoice-patterns
description: "Invoice domain rules for this solution. Use when creating or changing invoice entities, services or reports."
---

# Invoice Patterns
...
```

Assistants load a skill only when the task matches its description, so a long
skill costs nothing until it is needed. Keep `SKILL.md` under 500 lines and move
reference material into a `references/` subdirectory next to it.

### Adding a Command

```
.agents/commands/generate-report.md
```

```markdown
---
name: generate-report
claudePath: generate/report
description: "Scaffold a new report page"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "$REPORT_NAME - The report name in PascalCase"
---

# Generate Report
...
```

`claudePath` is the folder and file name Claude Code should use, which is how the
command becomes `/generate:report` there and `/generate-report` elsewhere.

### Path Placeholders

Never write a vendor path such as `.claude/skills/` inside a source file. Use a
placeholder and the generator rewrites it for each assistant:

| Placeholder | Becomes |
|-------------|---------|
| `{{SKILLS_DIR}}` | the assistant's skills directory |
| `{{COMMANDS_DIR}}` | the assistant's command, workflow or prompt directory |
| `{{RULES_FILE}}` | the assistant's rules file |

## Notes

- **Keep the rules short.** Instructions that load on every request compete with
  your actual prompt for context. Put facts and non-negotiables in
  `.agents/rules/`, and put procedures in a skill so they load on demand.
  Windsurf additionally caps the size of a rule file; the sync script warns you
  when a generated file goes over.
- **The assistant is not a substitute for [Power Tools](Power-Tools-Overview).**
  Power Tools generates a complete, regenerable CRUD feature from an entity
  definition. The AI configuration helps with everything around it: one-off
  changes, reviews, debugging, and code that does not fit the CRUD shape.
- **Review what it writes.** The rules make an assistant far more likely to
  produce layer-correct ABP code, but they do not make its output correct.
