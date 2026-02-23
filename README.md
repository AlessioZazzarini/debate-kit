<p align="center">
  <h1 align="center">debate-kit</h1>
  <p align="center">
    Cross-model adversarial review for Claude Code.<br/>
    Spawn a second LLM to challenge your plans, diagnoses, and code.
  </p>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License" /></a>
  <a href="https://claude.ai/claude-code"><img src="https://img.shields.io/badge/built%20for-Claude%20Code-blueviolet" alt="Built for Claude Code" /></a>
  <a href="#cost-breakdown"><img src="https://img.shields.io/badge/cost-%240.50--1.50%2Frun-green" alt="Cost per run" /></a>
</p>

---

You write a plan or diagnose a bug in Claude Code. Instead of trusting yourself, you run `/debate` and a **separate LLM instance** (Claude Sonnet or GPT-5.2) reviews your work with read-only codebase access. The reviewer gives a verdict: **APPROVED** or **REVISE**. If REVISE, the orchestrator fixes the issues and resubmits — up to 3 rounds.

```
You (Claude Opus)           Reviewer (Sonnet / GPT-5.2)
 │                           │
 ├─ Write plan               │
 ├─ /debate ────────────────►│ Read codebase (read-only)
 │                           │ Analyze plan
 │◄──── VERDICT: REVISE ─────┤
 ├─ Fix issues               │
 ├─ Resubmit ───────────────►│ Re-review
 │◄──── VERDICT: APPROVED ───┤
 └─ Continue with confidence
```

## Quick Start

**1. Copy into your project:**

```bash
# Clone and copy
git clone https://github.com/AlessioZazzarini/debate-kit.git
cp -r debate-kit/ your-project/.claude/skills/debate/
```

**2. Describe your project** (the only required step):

Open `.claude/skills/debate/architecture-brief.md` and fill in your tech stack, directory structure, and key patterns. A [commented-out example](architecture-brief.md) is included at the bottom of the template.

**3. Use it:**

```
/debate                        # Review the plan you're writing
/debate debug                  # Challenge a bug diagnosis
/debate review                 # Code review recent git changes
/debate review src/lib/        # Code review a specific path
/debate --provider codex       # Use GPT-5.2 instead of Claude Sonnet
```

## Prerequisites

| Requirement | Notes |
|------------|-------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | The CLI tool — you're probably already using it |
| `ANTHROPIC_API_KEY` | Set in your environment |
| [Codex CLI](https://github.com/openai/codex) *(optional)* | For cross-model diversity: `npm install -g @openai/codex` + `OPENAI_API_KEY` |

## How It Works

The skill operates in three modes, each tailored to a different stage of development:

| Mode | Command | What the Reviewer Does |
|------|---------|----------------------|
| **Plan Review** | `/debate` | Challenges architecture decisions, finds security gaps, spots race conditions |
| **Debug Review** | `/debate debug` | Proposes alternative root causes, identifies untested assumptions |
| **Code Review** | `/debate review [path]` | Finds logic bugs, missing error handling, performance issues |

Plan and debug modes loop (up to 3 rounds) until the reviewer approves or the limit is hit. Code review is single-pass — findings are presented once.

For the full technical flow with diagrams, see **[docs/how-debate-works.md](docs/how-debate-works.md)**.

## Customization

### Architecture Brief *(required)*

[`architecture-brief.md`](architecture-brief.md) is the single file the reviewer reads to understand your project. The more specific you are, the better the reviews:

> "We use Prisma middleware for tenant isolation on all queries" > "Standard patterns"

### Domain Context *(optional)*

Drop specialized context files into [`domain-context/`](domain-context/) for deeper reviews of specific subsystems. Each file declares when it should be included via YAML frontmatter:

```yaml
---
name: "API Layer"
triggers:
  paths: ["src/api/", "src/routes/"]
  keywords: ["endpoint", "route", "middleware"]
---
```

When a plan or review touches matching paths or keywords, the context is automatically included. Two examples are provided:

- [`example-web-app.md`](domain-context/example-web-app.md) — Full-stack TypeScript app (routes, auth, DB)
- [`example-data-pipeline.md`](domain-context/example-data-pipeline.md) — Python ML pipeline (DAGs, features, models)

Copy the closest example, rename it, and customize. Files starting with `_` are templates and are excluded from matching.

### Providers *(optional)*

| Provider | Model | Flag | Notes |
|----------|-------|------|-------|
| Claude CLI *(default)* | Sonnet | — | Uses your existing Anthropic key |
| Codex CLI | GPT-5.2 | `--provider codex` | Cross-model diversity; different blind spots |

## Cost Breakdown

| Mode | Max Rounds | Cost per Round | Max Cost |
|------|-----------|---------------|----------|
| Plan review | 3 | ~$0.50 | **~$1.50** |
| Debug review | 3 | ~$0.50 | **~$1.50** |
| Code review | 1 | ~$0.50 | **~$0.50** |

Early exit on APPROVED — if round 1 passes, you pay for 1 round only.

## Repository Structure

```
.claude/skills/debate/          # ← Where it lives in your project
├── SKILL.md                    # Executable skill definition
├── architecture-brief.md       # Your project description (edit this)
├── docs/
│   ├── how-debate-works.md     # Technical deep-dive with diagrams
│   └── TRIAGE-2026-02-22.md    # Case study: the skill reviewing itself
└── domain-context/
    ├── _template.md            # Blank template for new domains
    ├── example-web-app.md      # Reference: TypeScript web app
    └── example-data-pipeline.md # Reference: Python ML pipeline
```

## Background

This skill was built for the [Roger](https://github.com/AlessioZazzarini/Roger) project and improved by its own review system. We ran `/debate review` on the original skill using GPT-5.2 as the reviewer. It found 11 issues — after triage, 4 were real correctness bugs (broken session resume, unreliable exit codes, ambiguous verdict parsing) and 4 were security theater inappropriate for a local dev tool.

The full triage is preserved in [docs/TRIAGE-2026-02-22.md](docs/TRIAGE-2026-02-22.md) — a useful reference for evaluating AI code reviews critically.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "No architecture-brief.md found" | Fill in the template — the skill works without it but reviews will be generic |
| "Reviewer process failed (exit N)" | Auth issue. Verify `ANTHROPIC_API_KEY` is set, or run `codex login` for Codex |
| "Reviewer produced no output" | Rate limit or expired token. Wait a minute, retry. For Codex: `codex login` |
| Reviews are too generic | Add more detail to your architecture brief. Add domain context files for key subsystems |

## License

[MIT](LICENSE)
