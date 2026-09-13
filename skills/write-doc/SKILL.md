---
name: write-doc
description:
  Author or substantially revise the repository's document by driving a writer
  agent and a review-panel agent through scoping, drafting, review, and revision
  until the panel approves
---

This skill is the orchestrator of a run and nothing else: it launches one writer
agent and one panel agent, sends each one call at a time, enforces the run's
cap, and relays what they report to the user. It reads no file and holds no view
of the document, the writing style, or the review process — the writer and the
panel do that work, share a run dir, and report one outcome line per call; this
skill passes paths between them and never opens them.

The run's four parameters are set here: **three drafts**, a re-review cap of
**five rounds**, the writer `reviewed-writer:diataxis-writer`, and the panel
`reviewed-writer:diataxis-persona-panel`. The invocation overrides any of them,
so a repository that wants different values on every run states them in the
wrapper skill that invokes this one. A replacement writer or panel must honor
its call contract below. Two is the lowest draft count the writer's draft
comparison works with.

## Call contracts

Each message to an agent opens with the call name. The agent's reply opens with
the outcome line; detail stays in the run dir, and the reply names paths. A
`blocked` line from any call ends the run: relay it to the user and stop.

### The writer

| Call         | Payload                                                                                                                    | Outcome lines                                                                                                                                        |
| ------------ | -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `scope`      | The document, the change driving the revision, the scoping the user supplied, a design brief or none, and the draft count. | `blocked — <missing path or heading>; copy <template path>`, or `scoped — run dir: <path>`, then the scope summary and the open questions, or `none` |
| `write`      | The answers to the open questions, or `none`.                                                                              | `drafted — drafts: <paths>; run dir: <path>`, or `blocked`                                                                                           |
| `synthesize` | `matrix: <path>; another draft round: available\|none`.                                                                    | `again — drafts: <paths>`, valid only after `available`; or `ready — document: <path>`; or `blocked`                                                 |
| `revise`     | `matrix: <path>`.                                                                                                          | `ready — document: <path>`                                                                                                                           |
| `finish`     | `rounds used: k of N; unresolved: none\|<dissent lines>`.                                                                  | `done — record: <where>; unresolved: none\|<reviewers>`, then at most ten record lines                                                               |
| `resume`     | `run dir: <path>; call: <the interrupted call line verbatim>`.                                                             | The outcome of the interrupted call                                                                                                                  |

### The panel

| Call     | Payload                                                                           | Outcome lines                                                                                                                                                                                                          |
| -------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `scope`  | The document, the change driving the revision, and the scoping the user supplied. | `blocked — <missing path or heading>; copy <template path>`, or `blocked — cannot launch reviewers`, or `scoped — personas: <n>`                                                                                       |
| `review` | `run dir: <path>; drafts: <paths>`.                                               | `reviewed — matrix: <path>; reviewers: launched\|continued\|relaunched`, then a verdict summary, or `blocked`                                                                                                          |
| `review` | `run dir: <path>; document: <path>`.                                              | `approve — matrix: <path>; reviewers: continued\|relaunched`, or `revise — matrix: <path>; reviewers: continued\|relaunched` then one line per dissenting reviewer with its single most important change, or `blocked` |
| `resume` | `run dir: <path>; call: <the interrupted call line verbatim>`.                    | The outcome of the interrupted call                                                                                                                                                                                    |

### Rounds

`k` is the count of review rounds used and `N` the cap; this skill keeps both.
`k` counts every `review` outcome received after the run's first `reviewed` —
the draft round is not a round — and increments when the outcome arrives, never
when the call is sent, so a resumed review counts once. Another draft round is
`available` when `k + 1 < N`, which leaves at least one round for the document.

## Steps

1. **Scope** — In one message, launch the writer and the panel with the Agent
   tool, `subagent_type` set to each one's name, each with the `scope` call.
   Keep the two agent IDs, the run dir, `k = 0`, `N`, the document path, the
   draft paths, and the last dissent lines — nothing else. On any `blocked`,
   relay every `blocked` line to the user and stop. On both `scoped`, relay the
   writer's scope summary; when it lists open questions, put them to the user
   and collect the answers.

2. **Write** — Send the writer `write` with the answers, or `none`. On
   `drafted`, note the draft paths and the run dir.

3. **Draft round** — Send the panel `review` with the run dir and the drafts. On
   `reviewed`, the run's first leaves `k` at 0 and every later one adds 1. Send
   the writer `synthesize` with the matrix path and
   `another draft round: available` when `k + 1 < N`, else `none`. On `again`,
   repeat this step with the drafts it names. On `ready`, note the document
   path.

4. **Iterate** — If `k` is at or above `N`, the cap is reached: go to step 5.
   Otherwise send the panel `review` with the run dir and the document; when the
   outcome arrives, add 1 to `k`. On `approve`, go to step 5. On `revise` with
   `k` at or above `N`, the cap is reached: relay the dissent lines to the user
   as the unresolved verdicts and go to step 5 — do not keep bending the text to
   chase the last holdout, and never send `revise` after the last round, so the
   text that ships is always text a round reviewed. Otherwise send the writer
   `revise` with the matrix path, expect `ready`, and repeat this step.

5. **Finish** — Send the writer `finish` with `rounds used: k of N` and the
   unresolved verdicts, or `none`, and relay the `done` lines to the user as the
   run report. The panel is not called.

## Guidelines

- Relay every outcome line verbatim as it arrives, together with the run dir and
  `k of N`: a subagent's messages are not shown to the user, and the relay keeps
  the values through context compaction.
- Wait for each reply; never poll. Continue an agent with the SendMessage tool,
  addressed by its agent ID, not its name.
- Never open the run dir's files — the drafts, the document, the brief, the
  matrix, the record; relay their paths.
- A reply without an outcome line gets one message asking for it; a second such
  reply to the same call counts as a lost agent. The count is per agent and per
  call, so the next call starts again at zero. An `again` after
  `another draft round: none` is a reply without an outcome line.
- When a send is refused or an agent is lost, launch a fresh agent of the same
  name with the `resume` call, the run dir, and the interrupted call line; its
  reply is the outcome the interrupted call would have produced. The other agent
  keeps its ID. Continue from there. If no run dir was ever reported — the agent
  was lost during `scope` — launch again with `scope` instead.
- Act on the outcome line alone. The domain content of a reply is the agents' to
  produce and the user's to read.
