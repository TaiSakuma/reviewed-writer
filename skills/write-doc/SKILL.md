---
name: write-doc
description:
  Author or substantially revise the repository's document by driving an engine
  agent through scoping, drafting, review, and revision until its reviewers
  approve
---

This skill is the orchestrator of a run and nothing else: it launches one engine
agent, sends it one call at a time, enforces the run's cap, and relays what the
engine reports to the user. It reads no repository file and holds no view of the
document, the writing style, or the review process — the engine does that work
and reports one outcome line per call.

The run's three parameters are set here: **three drafts**, a re-review cap of
**five rounds**, and the engine `reviewed-writer:diataxis-persona-engine`. The
invocation overrides any of them, so a repository that wants different values on
every run states them in the wrapper skill that invokes this one. A replacement
engine must honor the call contract below. Two is the lowest draft count the
engine's draft comparison works with.

## Call contract

Each message to the engine opens with the call name. The engine's reply opens
with the outcome line; detail stays in the engine's run dir, and the reply names
paths. `k` is the count of review rounds used and `N` the cap; the engine
reports them, and this skill only compares them. A `blocked` line from any call
ends the run: relay it to the user and stop.

| Call     | Payload                                                                                                                             | Outcome lines                                                                                                                                                                                                                    |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `scope`  | The document, the change driving the revision, the scoping the user supplied, a design brief or none, the draft count, and the cap. | `blocked — <missing path or heading>; copy <template path>` or `blocked — cannot launch reviewers`, or `scoped — run dir: <path>`, then the scope summary and the open questions, or `none`                                      |
| `write`  | The answers to the open questions, or `none`.                                                                                       | `ready — document: <path>; rounds used: k of N; run dir: <path>`, or `blocked`                                                                                                                                                   |
| `review` | Nothing.                                                                                                                            | `approve — rounds used: k of N; reviewers: continued\|relaunched`, or `revise — rounds used: k of N; reviewers: continued\|relaunched` then one line per dissenting reviewer with its single most important change, or `blocked` |
| `revise` | Nothing.                                                                                                                            | `ready — …`, as after `write`                                                                                                                                                                                                    |
| `finish` | Nothing.                                                                                                                            | `done — record: <where>; rounds used: k of N; unresolved: none\|<reviewers>`, then at most ten record lines                                                                                                                      |
| `resume` | The run dir.                                                                                                                        | The outcome of the stage the engine had reached                                                                                                                                                                                  |

## Steps

1. **Scope** — Launch the engine with the Agent tool, `subagent_type` set to the
   engine's name, and the `scope` call. Keep the engine's agent ID, the run dir,
   `k`, `N`, and the document path — nothing else. On `blocked`, relay the line
   to the user and stop. On `scoped`, relay the scope summary; when the reply
   lists open questions, put them to the user and collect the answers.

2. **Write** — Send `write` with the answers, or `none`. On `ready`, note `k`
   and the document path. If `k` is already at or above `N`, the cap was spent
   before any `review` call: relay that as the cap reached and go to step 4.

3. **Iterate** — If `k` is at or above `N`, the cap is reached: go to step 4.
   Otherwise send `review`. On `approve`, go to step 4. On `revise` with `k` at
   or above `N`, the cap is reached: relay the dissent lines to the user as the
   unresolved verdicts and go to step 4 — do not keep bending the text to chase
   the last holdout, and never send `revise` after the last round, so the text
   that ships is always text a round reviewed. Otherwise send `revise`, expect
   `ready`, and repeat this step.

4. **Finish** — Send `finish` and relay the `done` lines to the user as the run
   report.

## Guidelines

- Relay every outcome line verbatim as it arrives, together with the run dir and
  `k of N` as last reported: a subagent's messages are not shown to the user,
  and the relay keeps the values through context compaction.
- Wait for the engine's reply; never poll. Continue the engine with the
  SendMessage tool, addressed by its agent ID, not its name.
- Never open the brief, the matrix, the drafts, or the record; relay their
  paths.
- A reply without an outcome line gets one message asking for it; a second such
  reply to the same call counts as a lost engine. The count is per call, so the
  next call starts again at zero.
- When a send is refused or the engine is lost, launch a fresh engine with the
  `resume` call and the run dir; its reply is the outcome the interrupted call
  would have produced. Continue from there. If no run dir was ever reported —
  the engine was lost during `scope` — launch again with `scope` instead.
- Act on the outcome line alone. The domain content of a reply is the engine's
  to produce and the user's to read.
