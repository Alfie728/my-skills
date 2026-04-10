---
name: daily-updates
description: Generate Alfie's daily work update for Rovi Health by pulling PRs, Linear tickets, Slack messages, and Conductor sessions, then drafting a concise standup summary for Tarun. Use when writing or sending a daily update.
argument-hint: "[date or 'today']"
metadata:
  short-description: Draft the daily work update for Tarun
---

# Daily Update Skill Spec

Guidelines for generating Alfie's daily work update for Rovi Health.

## Steps

1. **Check GitHub PRs.** Run `gh pr list --author @me --state all` from any conductor workspace to find PRs created, updated, merged, or reviewed today. For each relevant PR, run `gh pr view <number>` to get the title, description, and status.

2. **Check Linear tickets.** Use the Linear MCP tools to find tickets assigned to Alfie that were updated or completed today. Look for status changes (moved to Done, In Progress, In Review), new comments, and newly created tickets. Cross-reference with the ROVI-XXX ticket numbers found in PR branches and conductor workspace attachments.

3. **Check Slack messages.** Search all of Alfie's DMs, group DMs, and channel mentions from today using the Slack MCP tools. Filter for messages directed at Alfie or sent by Alfie. Look across Rovi Health's Slack channels for work that was discussed, requested, completed, or acknowledged.

4. **Check recent Conductor sessions.** Scan `/Users/alfiechen/conductor/workspaces/main/` for recent work:
   - For each workspace, check the git branch name (contains the Linear ticket ID, e.g. `alfie/rovi-556-core-metrics-pipeline`)
   - Read `.context/notes.md` and `.context/todos.md` for session context
   - Read `.context/plans/*.md` for implementation details and what was worked on
   - Read `.context/attachments/[LINEAR]-ROVI-*.md` for the Linear ticket descriptions
   - Check `git log --oneline --since="today" --author="alfie"` in each workspace for today's commits
   - Check recent session transcripts in `~/.claude/projects/-Users-alfiechen-conductor-workspaces-main-*/` (read the .jsonl files) for work done today

5. **Check active/in-progress work.** Look for open and draft PRs (`gh pr list --author @me --state open`) and Linear issues in "In Progress" or "In Review" status. These represent ongoing work that may span multiple days. For each, check recent pushes, review comments, and whether it's getting closer to merge. Include notable progress on long-running PRs (e.g. addressing review feedback, significant pushes).

6. **Cross-reference everything.** Combine GitHub PRs, Linear tickets, Slack messages, and Conductor session history into a full picture of the day's work. Group related items (e.g. a PR + its Linear ticket + the Slack discussion about it).

7. **Draft the update** following the guidelines below.

8. **Show the draft to Alfie for approval.**

## Audience

Tarun (CTO) reads these. He cares about progress and status, not implementation details.

## Tone

Casual, direct, concise. Like a quick standup summary.

## Length

3-5 bullet points max per day. Each bullet is 1-2 lines. If it's longer, trim it.

## Formatting

- No em dashes or en dashes (use "to" or commas instead)
- No internal workflow references (path letters, branch conditions, attribute configs)
- No overly specific technical details
- Backticks for Slack channel names
- Bold for pending item titles

## Structure

- `## {ordinal} {Month}` heading (e.g. `## 5th March`)
- Group by status: `### Merged`, `### In Progress`, `### In Review`, etc. Only include sections that have items.
- Under each status section, group by Linear issue: bold the issue title and number as a subheading, e.g. `**Appointment Booking MVP (ROVI-468)**`
- Bullet points under each issue describing what happened (PR merged, comments addressed, progress made, etc.)
- Optional freeform note after bullets for context or caveats

## Content Focus

What got done: PRs merged, Linear tickets completed, code shipped, bugs fixed, features built. Not local note reorganisation, vault work, or tooling setup.

## Level of Detail

Describe the what and outcome, not the how.

Good: "Bug reports now get tagged by feature and forwarded to `#project-ats`"
Bad: "Created Bug Feature = ATS branch condition in D. Path and dragged to new path box"
