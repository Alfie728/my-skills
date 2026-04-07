# Eval-Driven Development For Agents And LLM Features

The eval set is the test suite for non-deterministic systems. Agents go off rails. The eval set is the data that proves they don't.

This doc draws heavily on [Hamel Husain's framework for product evals](https://hamel.dev/blog/posts/evals-skills/). Read it. The core insight: **product evals measure whether your pipeline works on your task with your data.** They are not foundation model benchmarks. Your evals must be grounded in your stack, your domain, and your actual failures.

## When To Use This

- Building or tuning a prompt
- Building an agent (single-step or multi-step)
- Tool-calling models
- RAG pipelines
- LLM-based classification or extraction
- Structured output generation
- Anything where the same input can produce different outputs

If the work is purely deterministic plumbing around an LLM call (validation, retries, parsing, persistence), use the standard TDD loop for that part and the eval loop for the model-touching part.

## The Loop

```
1. Error analysis: look at real (or synthetic) traces
        ↓
2. Open-code failures into a vocabulary
        ↓
3. Cluster failures into a taxonomy
        ↓
4. Build the dataset around the failure modes
        ↓
5. Write eval harness → run → expect failure  (RED)
        ↓
6. Tune the prompt or agent                    (GREEN)
        ↓
7. Re-run full eval set, compare per-category scores
        ↓
8. Add new failures that surface               (dataset only grows)
        ↓
9. Hit the agreed quality bar → ship
```

The first three steps are the part most teams skip. They are the most important. **Error analysis comes before metrics.** If you start by writing aggregate scores, you will lump distinct failure modes together and miss the ones that matter.

## Step 1: Error Analysis — Look At Real Traces

Before listing behaviors from your imagination, look at what the system actually produces.

- **If the agent already exists in any form** — pull production traces or run the current version against realistic inputs and capture the outputs.
- **If this is a brand-new build** — write a throwaway prompt, run it against 20-30 realistic inputs you would expect users to send, and look at the outputs.

You are not grading yet. You are reading. The goal is to *see what is actually happening* before you decide what to measure.

If you cannot describe a single realistic user input, you do not understand the feature well enough to build it. Go back to `$user-flow` or `$grill-me`.

## Step 2: Open-Code The Failures

For each trace that looks wrong, write a free-text label describing what went wrong. Do not use a predefined taxonomy yet. The labels should be specific and concrete:

- "Claimed plan included free returns when it does not"
- "Said 'I have canceled your order' but never called the cancel tool"
- "Used the wrong tool when the user asked about billing"
- "Gave a refund amount in the wrong currency"
- "Refused a benign request because of safety overcorrection"

These labels are your raw vocabulary. Borrow Hamel's distinction: "support bot hallucinating facts" and "support bot inventing user actions" are both hallucinations, but they are structurally different failure modes. If you collapse them into one "hallucination" bucket, you will miss the one that matters more.

## Step 3: Cluster Into A Failure Taxonomy

Group your free-text labels into a small set of failure categories. A typical taxonomy has 5-10 buckets:

- Hallucinated facts about the product or policy
- Hallucinated user actions (claimed to do something it did not do)
- Wrong tool selected
- Tool called with wrong arguments
- Refused a valid request
- Followed an invalid request
- Wrong output format / schema violation
- Unsafe or off-brand response
- Failed gracefully with the wrong fallback

This taxonomy is the contract you will measure against. Each example in the dataset will be tagged with the category it tests. Aggregate scores are reported **per category**, never as a single average. The team's quality bar is set per category too.

## Step 4: Build The Dataset Around The Failure Modes

Now build the golden dataset, sized 10-30 examples to start. Each row needs:

- **Input** — exact input the agent will receive
- **Failure category** — which bucket from your taxonomy this row tests
- **Expected behavior** — what should happen (action taken, tool called, output shape)
- **Grading criteria** — how do you decide pass / fail / partial

Coverage targets:

- Cases drawn from real (or realistic) traces — especially the failures you found in step 1
- At least one example per category in your taxonomy
- Adversarial inputs: jailbreaks, prompt injection, contradictory instructions, role-play attempts, tool misuse
- Dumb inputs: typos, partial sentences, single words, empty strings

Store the dataset as a versioned file (CSV, JSONL, or YAML) committed alongside the agent code. **Not in a notebook. Not in someone's laptop.**

## Step 5: Write The Eval Harness Before The Prompt

The harness should:

- Load the dataset
- Run each example through the agent
- Apply the grading criteria (see "Grading: Humans, Code, Or LLM Judges" below)
- Output: per-example verdict, **per-category aggregate**, diff against the previous run
- Log every input, output, latency, and token cost

Run the harness against an empty / placeholder prompt and confirm it fails. This is your RED. If the harness reports passing on a non-existent prompt, the harness is broken — fix it before tuning.

## Step 6: Tune The Prompt Or Agent

Make one focused change at a time:

- Add an instruction
- Reword a section
- Add a few-shot example
- Adjust tool descriptions
- Change the system prompt structure

Do not stack five changes and re-run. You will not know which one helped.

After each change, run the **full** eval set, not just the failing ones. Improvements on one example often cause regressions elsewhere.

## Step 7: Compare Per-Category Scores Between Runs

Track scores **by category**, not just overall. Save each run with a timestamp and a note describing what changed.

If a tuning step regresses earlier examples:

- That is a real failure, not a rounding error
- Either fix it or accept the tradeoff explicitly with the user
- "It got better on average" is not enough — find out which examples got worse, in which category, and why

A 90% overall score that hides a 40% pass rate on the "hallucinated user actions" category is a shipping bug. Per-category reporting catches this; aggregate scores hide it.

## Step 8: The Dataset Only Grows

When you discover a new failure mode while tuning, label it, slot it into the taxonomy (or create a new bucket), and add it to the dataset. Then keep tuning until you pass it without breaking the rest.

**Never delete examples to make scores look better.** If an example feels unfair, talk to the user about whether the requirement is wrong, do not silently drop the row.

## Step 9: Set And Defend A Quality Bar

Before you start tuning, get the user's commitment on the pass rate that ships — **per category**:

- "100% on safety / never-do categories"
- "95% on hallucinated user actions" (a high bar because the consequence is bad)
- "85% on hallucinated facts"
- "70% on adversarial inputs"
- "Latency under 3s at p95, cost under $0.02 per call"

If you do not set a bar up front, you will tune forever. If you set it after the fact, you will move it to match the result.

---

## Grading: Humans, Code, Or LLM Judges

For each example in the dataset, you need to decide pass / fail. There are three options:

### Code-based grading (best when possible)

If the expected output is structured (a tool call, a JSON shape, an extracted value), grade with code. This is deterministic, fast, and free.

### Human grading (best for nuance)

If the output is open-ended (a generated message, a summary, a multi-turn conversation), a human reviewer is the gold standard. Build an annotation interface — a small web tool that shows the input, the output, and lets the reviewer mark pass / fail / partial with a free-text reason. Do not review traces in a CSV; the friction will kill the practice.

### LLM-as-judge (best for scale, but must be calibrated)

When you cannot grade with code and cannot review every example by hand, use an LLM as the judge. Rules:

1. **Binary pass/fail, not 1-5 scores.** Fuzzy scores produce fuzzy decisions. A judge that returns "3.7" is unfalsifiable. A judge that returns pass or fail can be measured against a human label.

2. **One thing per judge.** Do not write a judge that grades five dimensions at once. Write one judge per failure category. They compose better and you can tune them independently.

3. **Calibrate against human labels.** Hand-label 30-50 examples yourself. Run the judge against them. Measure:
   - **True positive rate (TPR)** — when a real failure exists, does the judge catch it?
   - **True negative rate (TNR)** — when there is no failure, does the judge correctly say so?
   - A judge with high TPR but low TNR over-flags. A judge with high TNR but low TPR misses real bugs. Both are unusable until tuned.

4. **Watch for judge bias.** LLM judges tend to favor verbose outputs, prefer their own model family's style, and grade leniently when the prompt sounds confident. Calibration data exposes these.

5. **Re-calibrate when you change the judge prompt or the underlying model.** Treat the judge like any other model: it has a version and a known accuracy.

A judge that has not been calibrated against human labels is not a judge. It is a vibes machine wearing a lab coat.

---

## Production Observability

You cannot tune what you cannot see. Wire up before shipping:

- Log every input and output to a queryable store
- Capture latency, token count, cost, model version
- Build (or install) an annotation interface so you can review traces with one click — not from a CSV
- Sample a subset for human review on a recurring basis
- Set up alerts for regressions on metrics that matter (refusal rate, tool-call failure rate, average latency)

Production logs become future eval examples. The eval set should grow as you learn what real users do.

The annotation interface is not optional. If reviewing a trace takes more than 5 seconds of friction, no one will do it, and the eval set will rot.

## What Counts As "Done"

A story is shippable when:

- Per-category pass rates meet the agreed bars (not just overall average)
- The dataset includes adversarial cases and real production-shaped failures, not just happy path
- LLM judges have been calibrated against human labels with TPR/TNR reported
- Regressions from prior runs are explicitly acknowledged
- The eval harness is committed alongside the agent code
- Production observability and the annotation interface are wired up
- A teammate could re-run the evals tomorrow and reproduce your numbers

If any of these are missing, the story is not done — even if the code works in your terminal.

## Anti-Patterns

- **Metrics before error analysis.** Aggregate scores designed before you have looked at real failures will measure the wrong things. Look at the data first.
- **One generic "hallucination" score.** Hallucinated facts and hallucinated actions are different failure modes. Lumping them hides the dangerous one. Always score per category.
- **Imagination-based datasets.** Examples invented at a desk miss the failure modes real users surface. Pull traces.
- **Uncalibrated LLM judges.** A judge without TPR/TNR against human labels is not a measurement, it is a guess.
- **Fuzzy 1-5 judge scores.** Binary pass/fail can be calibrated. A 3.7 cannot be defended.
- **Reviewing traces from a CSV.** If the friction is high, no one reviews. Build an annotation interface.
- **Vibes-based tuning.** "It looks better now" is not data. Run the eval set.
- **One-example evals.** A single happy-path test is not an eval. It is a demo.
- **Cherry-picked datasets.** If your dataset only has cases the agent already passes, the bar is meaningless.
- **Notebook-only evals.** If the eval cannot be re-run by a teammate from a clean checkout, it is not part of the codebase.
- **Untracked tuning history.** If you cannot say what changed between v1 and v7, you cannot defend the regressions.
- **Ship-then-eval.** Evals are part of the build, not a Friday afternoon afterthought.

## Further Reading

- [Hamel Husain — Evals as a Skill](https://hamel.dev/blog/posts/evals-skills/) — the framing this doc is built on
- Hamel's broader writing on error analysis, LLM judges, and product evals
