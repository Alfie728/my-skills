---
name: pr-triage
description: Triage PR review comments — pull every comment from the current branch's PR, classify which are valid vs. noise from AI review bots, address valid ones locally without committing, then let the user review before push and reply/resolve everything on GitHub. Use when AI review tools (CodeRabbit, Sentry, Copilot reviewer, etc.) leave many rounds of comments and you want to filter the noise.
argument-hint: <pr-number-optional>
metadata:
  short-description: Filter and address PR review comments before merge
---

# PR Triage

IMPORTANT: Never auto-commit. Never auto-push. Never reply to or resolve a comment on GitHub before the user has reviewed both the local changes and the planned GitHub round-trip.

## When To Use

You have an open PR on the current branch and an AI review tool (CodeRabbit, Sentry, GitHub Copilot reviewer, Vercel, etc.) has left many rounds of comments. Some are real issues. Most are pedantic noise, false positives, or factually wrong. You want a fast triage loop:

1. Pull every comment
2. Decide which are valid
3. Fix the valid ones locally
4. See the agent's reasoning for the dismissed ones
5. Approve (or push back on individual decisions)
6. Push, reply on GitHub, and resolve everything

The whole point is to **filter AI review noise** without skimming comments by hand.

## Process

### 1. Locate The PR

Auto-detect the open PR for the current branch:

```bash
gh pr view --json number,title,headRefName,url,author
```

If no PR is open on the current branch, ask the user for a PR number or URL. Do not proceed without a target.

### 2. Pull Every Comment

Fetch all review threads, inline comments, and general PR comments:

```bash
# Inline review comments grouped into threads (with resolved state)
gh api graphql -f query='
  query($owner:String!, $repo:String!, $number:Int!) {
    repository(owner:$owner, name:$repo) {
      pullRequest(number:$number) {
        reviewThreads(first:100) {
          nodes {
            id
            isResolved
            isOutdated
            comments(first:50) {
              nodes {
                id
                databaseId
                author { login }
                body
                path
                line
                originalLine
                diffHunk
              }
            }
          }
        }
      }
    }
  }
' -F owner=<owner> -F repo=<repo> -F number=<number>

# General PR comments (the conversation tab, not inline)
gh pr view <number> --json comments
```

### 3. Filter And Bucket

Drop:

- Resolved threads (`isResolved: true`)
- Outdated threads (`isOutdated: true`)
- Comments authored by the current user (`gh api user --jq .login`)

Bucket the rest by author type:

- **AI bot comments** — author logins like `coderabbitai[bot]`, `coderabbitai`, `github-actions[bot]`, `vercel[bot]`, `sentry-io[bot]`, `copilot-pull-request-reviewer[bot]`, `dependabot[bot]`. Pattern: `[bot]` suffix is a strong signal.
- **Human reviewer comments** — everything else.

### 4. Read The Diff Context

Before classifying anything, fetch the PR diff so you can verify each comment against the actual code:

```bash
gh pr diff <number>
```

For each comment, read the file/line it references in the current working tree (the comment's `diffHunk` shows what the reviewer saw, but the file may have moved on).

### 5. Classify Each Comment

For **AI bot comments**, classify into one of these buckets and be willing to dismiss with reasoning:

| Verdict | When to use |
|---|---|
| **VALID — Address** | Real bug, real risk, real improvement that fits the codebase patterns |
| **VALID — Question** | Reviewer is asking, not requesting a change. Draft an answer. |
| **DISMISS — Factually wrong** | The bot claims the code does X but it actually does Y |
| **DISMISS — Pedantic / style noise** | Stylistic preference that conflicts with the codebase's existing patterns |
| **DISMISS — Already addressed** | Another part of the diff handles it |
| **DISMISS — Out of scope** | Valid concern, but belongs in a separate PR or issue |
| **DISMISS — False positive** | Flagged something that isn't actually a problem |
| **DISMISS — Unsafe suggestion** | Would introduce a bug, regression, or security issue |
| **NEEDS USER INPUT** | You can't tell without more context from the user |

For **human reviewer comments**, be conservative:

- Default to classifying as VALID — Address or VALID — Question
- Only mark as NEEDS USER INPUT if you genuinely can't tell what they want
- **Never silently dismiss a human comment.** If you think it's wrong, surface it as NEEDS USER INPUT with your reasoning, and let the user decide.

### 6. Flag Non-Trivial Fixes Before Touching Code

For each comment marked VALID — Address, decide if the fix is trivial or non-trivial.

**Trivial** (apply directly):
- Rename, typo, missing semicolon
- Unused import or variable
- Obvious null check
- Comment / docstring fix
- Single-line refactor with no behavior change

**Non-trivial** (pause and ask the user which path):
- Touches state, async, persistence, auth
- Requires test changes
- Refactor across multiple files
- Performance-sensitive code
- Anything in a security-sensitive path

For non-trivial fixes, ask: "Comment from `<author>` on `<file>:<line>` looks valid but the fix is non-trivial. Use `$cove` for verification, `$tdd` to drive it with tests, or apply directly?"

### 7. Apply Local Changes

For each VALID — Address comment, make the fix in the working tree.

- Group related fixes together when they touch the same file
- Run the relevant tests after each non-trivial fix
- **Do not commit. Do not stage. Do not push.**

### 8. Present The Triage Report

Show the user a single report with all the buckets. Use this template:

```markdown
# PR Triage Report — #<number> <title>

## Addressed Locally (<count>)

### <author> on `<file>:<line>` — <thread-id>
> <truncated comment body>

**Fix:** <one-line description of what was changed>
**Files touched:** `<file1>`, `<file2>`

---

## Dismissed (<count>)

### <author> on `<file>:<line>` — <thread-id>
> <truncated comment body>

**Verdict:** DISMISS — <category>
**Reasoning:** <2-3 sentences explaining exactly why this comment is wrong, pedantic, out of scope, etc.>

---

## Needs Your Input (<count>)

### <author> on `<file>:<line>` — <thread-id>
> <truncated comment body>

**Why I'm not sure:** <reason>
**Options:**
1. <option 1>
2. <option 2>

---

## Questions Drafted (<count>)

### <author> on `<file>:<line>` — <thread-id>
> <question body>

**Draft answer:** <your draft response>

---

## Local Diff

```diff
<full diff of all local changes>
```

## What I Will Do On GitHub After You Approve

1. Commit with message: `<proposed commit message>`
2. Push to `<branch>`
3. Reply "Done in <sha>" on <N> addressed threads
4. Reply with the dismissal reasoning on <M> dismissed threads
5. Post the drafted answers as replies on <Q> question threads
6. Mark **all** triaged threads as resolved (<total>)
```

### 9. Wait For User Approval

The user reviews:

- The local diff
- The dismissal reasoning (this is where they catch the agent being wrong)
- The needs-input bucket (they make calls on these)
- The question drafts
- The planned GitHub round-trip

The user can:

- Approve everything → proceed to step 10
- Override individual verdicts ("actually that dismissal was wrong, address it", "don't address that one, dismiss it instead")
- Skip the GitHub round-trip ("just commit and push, don't touch GitHub")
- Bail out completely

**Do not proceed to step 10 without an explicit approval from the user.**

### 10. Commit, Push, And Round-Trip On GitHub

Only after explicit approval:

1. **Commit** with the approved message. Use `git add` for the specific files only — never `git add -A` or `git add .`.
2. **Push** to the PR branch.
3. **Reply on addressed threads** with the commit SHA:
   ```bash
   gh api graphql -f query='
     mutation($threadId:ID!, $body:String!) {
       addPullRequestReviewThreadReply(input:{pullRequestReviewThreadId:$threadId, body:$body}) {
         comment { id }
       }
     }
   ' -F threadId=<thread-id> -F body="Done in <sha>"
   ```
4. **Reply on dismissed threads** with the reasoning from the triage report.
5. **Reply on question threads** with the drafted answer.
6. **Resolve every triaged thread**:
   ```bash
   gh api graphql -f query='
     mutation($threadId:ID!) {
       resolveReviewThread(input:{threadId:$threadId}) {
         thread { isResolved }
       }
     }
   ' -F threadId=<thread-id>
   ```

Most AI review tools auto-resolve threads after a new push, but explicit resolution is the safety net.

### 11. Final Status

Report to the user:

- Commit SHA pushed
- Threads addressed, dismissed, answered, resolved
- Any threads that failed to resolve (with the error)
- Suggest re-running `$pr-triage` if a new round of comments comes in after CI re-runs

## Quality Checks

- Every dismissed comment has reasoning specific enough that the user can disagree with it
- Human reviewer comments are never silently dismissed
- Non-trivial fixes were flagged before touching the code, not after
- The local diff matches the addressed comments — no surprise changes
- The commit message references what was addressed (not just "address PR comments")
- All threads in the triage report are accounted for in step 10 (no orphans)

## Anti-Patterns

- **Silently dismissing human comments.** If a human took the time to leave it, surface it. Never decide for them.
- **Auto-committing.** This skill exists because you do not trust the AI reviewers — do not trust the agent to commit either.
- **Vague dismissal reasoning.** "Not relevant" is not reasoning. Say *what* the bot got wrong and *how you know*.
- **Stacking many fixes into one big commit.** Group related fixes into focused commits when possible.
- **Skipping the diff fetch.** Reading the comment without reading the current state of the file leads to addressing things that are already fixed.
- **Resolving threads before pushing.** Push first, then reply, then resolve. If push fails you do not want resolved threads referencing missing changes.
