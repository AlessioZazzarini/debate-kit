# How the `/debate` Skill Works

> This is a standalone document. It describes the cross-model adversarial review system that ships as a Claude Code skill. No project-specific dependencies required.

A cross-model adversarial review system built into your Claude Code workflow. You (Claude) are the orchestrator — the reviewer is a **different model family** (ChatGPT, Gemini, or Claude Sonnet as fallback) running as a read-only subprocess. Different model families catch different blind spots, which is the whole point.

---

## The Core Idea

You're working in Claude Code (Opus). You write a plan or diagnose a bug. Instead of trusting yourself, you invoke `/debate` and a **different model** (ChatGPT via Codex CLI, Gemini, or Claude Sonnet as fallback) reviews your work with read-only access to the codebase. It can find files, read code, and grep — but never edit anything. Claude is the orchestrator AND the author being challenged — the reviewer is ideally from a different model family.

The reviewer gives a verdict: **APPROVED** or **REVISE**. If REVISE, the orchestrator (Opus) revises the plan based on feedback, then sends the updated version back for another round. This continues for up to 3 rounds.

---

## Three Modes

| Command | What It Reviews |
|---------|----------------|
| `/debate` | A plan you're writing (default) |
| `/debate debug` | Your proposed root cause for a bug |
| `/debate review [path]` | Recent code changes (git diff or specific path) |

---

## Three LLM Providers

The skill auto-detects installed CLIs and picks the first available, preferring cross-model diversity.

### Codex CLI / ChatGPT (recommended)

Runs GPT-5.2 via the `codex` CLI — a different model family from Claude:

```
codex exec -m gpt-5.2 -s read-only - < context.md > review.txt
```

- Sandbox: read-only mode (no writes)
- Uses your `OPENAI_API_KEY`
- Invoke with `--provider codex` flag

### Gemini CLI

Runs Gemini 2.5 Pro — yet another model family:

```
gemini -m gemini-2.5-pro < context.md > review.txt
```

- Read-only access to codebase
- Uses your `GEMINI_API_KEY` (or `GOOGLE_API_KEY`)
- Invoke with `--provider gemini` flag

### Claude CLI (same-family fallback)

Runs Claude Sonnet as a subprocess. Same model family as the orchestrator, so less adversarial diversity — but requires zero extra setup:

```
claude -p --model sonnet --allowedTools "Read,Grep,Glob" \
  --max-budget-usd 0.50 < context.md > review.txt
```

- Tools: Read, Grep, Glob (read-only)
- Cost cap: $0.50 per round
- Uses your existing `ANTHROPIC_API_KEY`
- Requires `env -u CLAUDECODE` to allow nested invocation

---

## Step-by-Step Flow

### Step 1: Build the Context Pack

The orchestrator creates a temporary directory and assembles a `context.md` file with three sections:

1. **Architecture Brief** — Overview of your project's tech stack, patterns, and structure (loaded from `architecture-brief.md` in the skill folder)
2. **Domain Context** — Conditionally included based on what the plan touches. Domain context files in the `domain-context/` folder declare path/keyword triggers via YAML frontmatter; matching files are automatically included.
3. **The Subject** — The actual plan, diagnosis, or code diff being reviewed

### Step 2: Invoke the Reviewer

The context pack is piped to the reviewer's stdin. The reviewer reads the codebase with its allowed tools, analyzes the subject, and writes findings to stdout.

### Step 3: Parse the Verdict

The orchestrator:
1. Checks the exit code (appended to the same shell call)
2. Verifies the output is non-empty
3. Strips markdown formatting
4. Finds the **last** `VERDICT:` line (case-insensitive)
5. Extracts issue and critical counts

Verdict format: `VERDICT: APPROVED` or `VERDICT: REVISE issues=3 critical=1`

### Step 4: Act on the Verdict

- **APPROVED** → Display confirmation, clean up, done
- **REVISE** → Display feedback, revise the plan, rebuild context pack with the revision + prior feedback, re-invoke the reviewer (back to Step 2)
- **No verdict found** → Warn user, treat as REVISE

Maximum: 3 rounds (1 initial + 2 revisions). Early exit on APPROVED — if round 1 passes, done immediately.

For **code review mode**, there's no revision loop — findings are presented once with an option to file a GitHub issue.

---

## What the Reviewer Looks For

### Plan Review
1. Security — missing auth, injection vectors, leaked secrets, missing RLS
2. Data integrity — schema conflicts, race conditions, missing constraints
3. Concurrency — unsafe parallel jobs, missing idempotency, stale reads
4. Edge cases — API failures, empty states, rate limits, partial failures
5. Architecture — simplicity, tech debt, pattern conflicts

### Debug Review
1. Alternative causes — what else could produce these symptoms?
2. Assumptions — what is being assumed that might not be true?
3. Missing evidence — what log/query/test would confirm or rule out the cause?
4. Scope — is this a symptom of a deeper issue?

### Code Review
1. Correctness — logic bugs, off-by-one, broken control flow
2. Error handling — unhandled failures, silent swallows
3. Data integrity — race conditions, incorrect queries
4. Performance — N+1 queries, missing indexes
5. Security — auth gaps, injection vectors (production code only)

Code review includes an anti-theater directive: prioritize issues that would cause real failures over theoretical concerns.

---

## Diagram

```
You (Claude Opus in Claude Code)
 │
 ├─ Write plan / diagnose bug / make code changes
 │
 ├─ /debate
 │   │
 │   ├─ [1] Assemble context.md (architecture + domain + subject)
 │   │
 │   ├─ [2] Spawn reviewer subprocess ──────────────────────┐
 │   │       (ChatGPT / Gemini / Claude Sonnet)             │
 │   │       Read-only tools: Read, Grep, Glob              │
 │   │       Budget: $0.50/round                            │
 │   │                                                      │
 │   │   ┌──────────────────────────────────────────────────┘
 │   │   │
 │   │   ▼
 │   │   Reviewer reads codebase, analyzes subject
 │   │   Writes findings + VERDICT
 │   │   │
 │   │   ▼
 │   ├─ [3] Parse verdict from review.txt
 │   │
 │   ├─ APPROVED? ──→ Done. Clean up.
 │   │
 │   └─ REVISE? ──→ Revise plan, append prior feedback
 │                   to context.md, go back to [2]
 │                   (max 3 rounds)
 │
 └─ Continue working with approved plan
```

---

## Key Design Decisions

**No session resume.** Each round is a fresh invocation with the full context repacked. Session resume was dropped because:
- Claude CLI session ID extraction is unreliable (ordering not guaranteed)
- Codex CLI stderr is human-readable text, not JSON — parsing fails
- Fresh invocations with context packing are simpler and equally cost-effective

**Read-only reviewer.** The reviewer can explore the codebase but never modify it. This makes it safe to run against any provider without risk.

**File-based I/O.** Plan text is never interpolated into shell arguments — always written to files and piped via stdin. This avoids shell quoting issues with complex content.

**Cross-model diversity.** The architecture supports multiple reviewer providers (Codex/ChatGPT, Gemini, Claude). Using a different model family from the orchestrator catches blind spots that same-family models share. The skill auto-detects installed providers and prefers cross-model options.
