---
name: persona-review
description:
  Run one round of persona review on a document or its drafts, as the
  repository's profile directs
---

Run one round of persona review: fixed-persona reviewers, each in its own
subagent, review a document (or several drafts of it), and their reviews are
collected and consolidated for the caller — an authoring workflow or a direct
invocation.

Every repository-specific value lives in the repository's
`.claude/rules/persona-review-profile.md` (the profile); read it first. The
Diátaxis rules are the shared review core at
`${CLAUDE_PLUGIN_ROOT}/skills/persona-review/references/diataxis-review.md`;
read it too — composing the brief needs its reader questions.

Before anything else, confirm the consumer-side files exist: the profile at that
path, carrying every `##` section that
`${CLAUDE_PLUGIN_ROOT}/templates/persona-review-profile.md` lists; the
declaration file at `.claude/rules/diataxis-declaration.md`; every persona head
file the profile's Personas section lists; and the voice-rules file its Voice
rules section names. If any file is missing, or a profile section is absent or
renamed, stop: report the missing path or heading to the user, name the matching
template under `${CLAUDE_PLUGIN_ROOT}/templates/` to copy and fill in, and do
not proceed on a guess or with invented contents.

## Steps

1. **Determine the review request.** When invoked from an authoring run, the run
   state in the conversation supplies the draft path(s), the document's purpose,
   what is in and out of scope, the declared quadrant(s), each unit's status
   (when the profile's Status dimension section is enabled), the rubric, the
   verified facts and link targets, the personas chosen for the run, and — in a
   run's later rounds — each persona's reviewer from the earlier rounds, the
   units changed since, and the earlier flag rows with their disposition
   (applied, or declined with the reason). When invoked standalone, default to:
   the shipped document named in the profile's Document section, as it stands;
   all personas listed in the profile's Personas section; implemented status,
   when the status dimension is enabled; no rubric.

2. **Compose the review brief** to a temp file: the project and document
   identity, from the profile's Document section; the document's purpose; the
   declared quadrant(s) and matching reader question(s), carried as the
   profile's Declaration mechanism section directs; when the status dimension is
   enabled, each unit's status, and the design decisions for spec content — the
   brief is self-contained; what is in and out of scope; the rubric, when the
   request has one; verified facts; link targets; and the path of the Diátaxis
   core, so reviewers read the core from the brief. Reviewers never depend on
   files outside the repository, this skill, and the brief. In a run's later
   rounds, update the existing brief in place — the draft path(s), the verified
   facts, what the round changed — rather than composing a new one; its path
   stays the same.

3. **Launch or continue the panel.** In a run's first round, launch one
   `reviewed-writer:persona-reviewer` subagent per persona in the request via
   the Agent tool's `subagent_type`, all in parallel, and keep each reviewer's
   agent ID for the run. Each task prompt gives, in order, the path of the
   persona's head file from the profile's Personas section, the brief path, and
   the draft path(s). In every later round, message each persona's existing
   reviewer instead of launching a new one, all in one message so they run in
   parallel: the path of the text now under review, the units changed since its
   last review, and its earlier flag rows by matrix number with their
   disposition; it re-reads the text from its own context and returns the
   follow-up form. Wait for the replies; never poll. Launch afresh only a
   reviewer that no longer exists, and it does a full review. The report's shape
   is the agent's own contract; do not restate it.

4. **Collect and consolidate.** If a reviewer errors out, returns no `Verdict:`
   line, or cannot be continued, re-launch it — do not treat a missing verdict
   as a pass. Build the matrix from the reports' tables, shorter than any one
   report: each persona's verdict line; with a rubric, scores by axis and draft
   and the best draft by count of personas; a relevance grid, units as rows and
   personas as columns, with the marker read for each unit and any disagreement;
   then the flag rows, numbered across the run, `blocking` first, grouped by
   unit, one row per finding naming the personas raising it and keeping its
   kind, destination, and fix. In a later round, update the previous matrix
   rather than rebuilding it: replace each persona's verdict line, mark earlier
   rows landed or still open as the follow-ups report, and append new rows with
   the next numbers. Report the matrix to the caller.
