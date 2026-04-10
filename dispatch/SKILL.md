---
name: dispatch
description: Enhances task execution by discovering missing skills/MCP servers, expanding lazy prompts, and routing to the right pipeline skill. Use for any non-trivial task where you want full tooling and verified output.
disable-model-invocation: false
user-invocable: true
argument-hint: "[describe what you want to build or accomplish]"
metadata:
  short-description: Equip, expand, and route a task through the pipeline
---

**IMPORTANT: Never auto-commit. Always present the final output for user review before committing or submitting to external services.**

# dispatch

Extends the host agent's native skill dispatch with three capabilities:

1. **Skill & MCP Discovery** — Find and install skills/MCP servers you're missing
2. **Lazy Prompt Expansion** — Turn vague requests into actionable specs
3. **Pipeline Routing** — Detect deterministic vs agent work, then route to the right skill (`$grill-me`, `$user-flow`, `$arch-brief`, `$write-a-prd`, `$prd-to-issues`, `$tdd` standard or eval mode, `$cove`)

You are NOT replacing the host agent's built-in skill system. You are layering on top of it.

---

## PHASE 1: EQUIP

When `$dispatch [task]` is invoked:

### 1A. Let the host agent's native system do its job first
The host agent will automatically detect and invoke relevant installed skills. Do not duplicate this.

### 1B. Fill the gaps native dispatch misses

**Search for and INSTALL skills that aren't installed yet:**
```bash
# Search for relevant skills
npx skills find [keywords from task]

# Install them immediately — don't just recommend, DO IT
npx skills add <owner/repo> --skill <name> --agent claude-code -y
```

Do NOT ask for permission to install skills. Just install them. The `-y` flag auto-confirms.

**Install missing MCP servers.** Don't recommend — run the install command directly:
```bash
# Example: always install Context7 if not present
claude mcp add context7 -- npx -y @upstash/context7-mcp
```

Use the toolkit knowledge base below to match the project's stack to the right servers.

**Set up missing project configuration.** If there's no CLAUDE.md, AGENTS.md, settings.local.json, or .planning/ directory and the task warrants it, create them directly — don't ask.

---

## PHASE 2: EXPAND

When the user gives a lazy prompt, expand it BEFORE executing. As part of expansion, **detect whether the work is deterministic code, agent/LLM work, or both** — this changes the routing in Phase 3.

### Agent / LLM Detection

Flag the task as **agent work** if it involves any of:

- Building or tuning a prompt
- Building an agent (single-step or multi-step)
- Tool calling, function calling
- RAG pipelines
- LLM-based classification, extraction, or summarization
- Structured output generation
- System prompt design
- Multi-step reasoning chains
- Chatbots / conversational interfaces
- Anything where the same input can produce different outputs

Trigger keywords: `agent`, `prompt`, `LLM`, `system prompt`, `tool calling`, `RAG`, `embedding`, `classifier`, `extract`, `summarize`, `chatbot`, `Claude`, `GPT`, `OpenAI`, `Anthropic`, `function calling`, `streaming response`.

If flagged, the task needs **eval-driven development**, not just TDD. Note this in the expanded task so Phase 3 routes correctly.

**Hybrid tasks** (e.g., "build an agent that books appointments" → has both an agent AND deterministic plumbing) should be flagged as both. The eval loop covers the model-touching parts; the standard TDD loop covers the deterministic plumbing.

### Examples

**Input:** "add pagination"

**Expansion:**
```
Task: Add pagination to [detected table/list component]
Type: Deterministic code

Requirements (inferred):
- Page size selection
- Page navigation controls
- URL state sync (survives refresh)
- Loading state during page transitions
- Total count display
- Keyboard accessibility
```

**Input:** "build an intent classifier for support tickets"

**Expansion:**
```
Task: Build an LLM-based intent classifier for support tickets
Type: AGENT WORK — needs eval-driven development

Behaviors to define:
- Intent categories (must be enumerated upfront)
- Confidence threshold for "unknown" fallback
- Adversarial inputs: empty tickets, multi-intent tickets, sarcasm, non-English

Eval requirements:
- Golden dataset of 20-30 labeled tickets across all categories
- Adversarial subset: 5-10 hard cases
- Quality bar must be agreed before tuning
- Eval harness committed alongside the classifier
- Production logging of input/output/confidence

Suggested route: $tdd in eval mode (see evals.md)
```

**Input:** "build a chatbot that books appointments and saves them to the database"

**Expansion:**
```
Task: Build appointment-booking chatbot with database persistence
Type: HYBRID — agent + deterministic code

Agent half (eval-driven):
- Conversation flow (prompt or agent loop)
- Slot extraction (date, time, service, contact)
- Disambiguation handling
- Tool calling for the booking action

Deterministic half (TDD):
- Database schema and persistence
- Conflict detection (no double-booking)
- Confirmation email/SMS dispatch
- Input validation on extracted slots

Suggested route: $tdd — apply eval loop to the agent half and the standard TDD loop to the deterministic half
```

**Input:** "fix the bug"

**Expansion:**
```
Task: Investigate and fix the bug in [detected context]
Type: Deterministic code (unless the bug is in an agent — re-check after reading the relevant code)

Steps:
1. Read the relevant code and understand current behavior
2. Identify the root cause (not just symptoms)
3. Implement fix with verification
4. Verify fix doesn't introduce regressions
```

---

## PHASE 3: ROUTE

Route the expanded task to the appropriate pipeline skill based on the dev cycle phase and the task type from Phase 2:

| Scenario | Phase | Route to |
|----------|-------|----------|
| Idea needs stress-testing before any docs | Pre-alignment | `$grill-me` |
| Mon/Tue: need a user flow doc for business approval | Alignment 1 | `$user-flow` |
| User flow approved, need architecture brief for Slack | Alignment 2 | `$arch-brief` |
| Codebase has architectural friction blocking a clean approach | Architecture | `$improve-codebase-architecture` |
| Arch brief approved, need full engineering spec | Spec | `$write-a-prd` |
| Dev PRD ready, need to break into Linear issues | Issue breakdown | `$prd-to-issues` |
| **Deterministic** implementation with tests | Build | `$tdd` (applies `$cove` for non-trivial GREEN steps) |
| **Agent / LLM** implementation needing evals | Build | `$tdd` in **eval mode** — see `tdd/evals.md` |
| **Hybrid** (agent + deterministic plumbing) | Build | `$tdd` — apply both loops in the same story |
| Standalone non-trivial code generation | Build | `$cove` directly |
| Large autonomous task (10+ files, full feature) | Build | `$ralph-loop` |
| Open PR has review comments to triage before merge | Merge | `$pr-triage` |
| Trivial one-liners, formatting, docs | — | Just do it — no routing needed |

**Hard rule for agent work:** `$tdd` in eval mode is **not optional**. Per the team's dev cycle, every agent build includes prompt tuning + eval testing as part of the estimate from day one. Never route agent work to plain `$tdd` or `$cove` without flagging the eval requirement.

**The full pipeline:**
```
$grill-me → $user-flow → $arch-brief → $write-a-prd → $prd-to-issues → $tdd [issue-id] → $pr-triage → merge
   idea      Mon-Tue:     Mon-Tue:       Tue:            Tue:            Wed-Fri:           before merge
              flow doc     arch brief    full eng spec   Linear epics    │
              (business    (team         (schemas, API   + stories       ├─ Deterministic → red-green-refactor
              approval)    alignment)    contracts,                      │                  + $cove on tricky GREEN
                                         module design)                  │
                        ↕ if arch friction                               └─ Agent / LLM   → eval-driven loop
                        $improve-codebase-architecture                                      (golden dataset,
                                                                                            quality bar,
                                                                                            regression tracking)
```

---

## TOOLKIT KNOWLEDGE BASE

**Install directly when relevant — don't ask, just do it.**

### MCP Servers — Auto-Install When Relevant

| Server | When to Install | Command |
|--------|-----------------|---------|
| Context7 | Any library usage, prevents doc hallucinations | `claude mcp add context7 -- npx -y @upstash/context7-mcp` |
| GitHub | PR workflows, issue tracking, code review | `claude mcp add github --transport http https://api.githubcopilot.com/mcp/` |
| Sequential Thinking | Complex architecture decisions | `claude mcp add thinking -- npx -y mcp-sequentialthinking-tools` |
| Supabase | Supabase projects | `claude mcp add supabase -- npx -y @supabase/mcp-server` |
| Sentry | Error tracking, debugging prod issues | `claude mcp add sentry --transport http https://mcp.sentry.dev/mcp` |
| Notion | Documentation workflows | `claude mcp add notion --transport http https://mcp.notion.com/mcp` |

### Skill Repositories to Search

| Repository | Best For | Install Prefix |
|------------|----------|----------------|
| `vercel-labs/agent-skills` | React, Next.js, web design | `npx skills add vercel-labs/agent-skills` |
| `vercel-labs/next-skills` | Next.js 15/16 specifics | `npx skills add vercel-labs/next-skills` |
| `anthropics/skills` | Docs, design, MCP builder | `npx skills add anthropics/skills` |
| `trailofbits/skills` | Security auditing | `npx skills add trailofbits/skills` |
| `obra/superpowers-marketplace` | TDD, planning, debugging | `/plugin marketplace add obra/superpowers-marketplace` |
| `expo/skills` | React Native / Expo | `npx skills add expo/skills` |
| `stripe/skills` | Payments | `npx skills add stripe/skills` |
| `supabase/agent-skills` | Postgres best practices | `npx skills add supabase/agent-skills` |
| `cloudflare/skills` | Workers, edge, web perf | `npx skills add cloudflare/skills` |
| `huggingface/skills` | ML training, datasets | `npx skills add huggingface/skills` |
| `sentry/skills` | Code review, commits | `npx skills add sentry/skills` |
| `K-Dense-AI/claude-scientific-skills` | 125+ scientific tools | `npx skills add K-Dense-AI/claude-scientific-skills` |

### Dynamic Search
```bash
npx skills find [keyword]
```

---

## EXECUTION SUMMARY

```
$dispatch [task]
    │
    ├─ PHASE 1: EQUIP
    │   ├─ Search skills.sh for uninstalled skills → install
    │   ├─ Detect missing MCP servers → install
    │   └─ Create project config if missing
    │
    ├─ PHASE 2: EXPAND
    │   ├─ Turn lazy prompts into actionable specs
    │   └─ Detect task type: deterministic / agent / hybrid
    │
    └─ PHASE 3: ROUTE
        ├─ $grill-me                       → stress-test the idea
        ├─ $user-flow                      → Mon-Tue: non-technical flow doc for business approval
        ├─ $arch-brief                     → Mon-Tue: Slack-sized architecture brief for team alignment
        ├─ $improve-codebase-architecture  → surface + fix architectural friction before spec work
        ├─ $write-a-prd                    → full engineering spec (schemas, API contracts, module design)
        ├─ $prd-to-issues                  → break dev PRD into Linear issues + workspace sequencing
        ├─ $tdd (standard)                 → deterministic code: red-green-refactor + $cove
        ├─ $tdd (eval mode)                → agent / LLM work: golden dataset + quality bar (see tdd/evals.md)
        ├─ $tdd (hybrid)                   → both loops in the same story
        ├─ $cove                           → standalone verified code generation
        ├─ $ralph-loop                     → autonomous multi-file execution
        └─ $pr-triage                      → filter AI reviewer noise, address valid comments, merge
```

---

Now processing: **$ARGUMENTS**
