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
   state supplies the draft path(s), the document's purpose, what is in and out
   of scope, the declared quadrant(s), each unit's status (when the profile's
   Status dimension section is enabled), the rubric, the verified facts and link
   targets, the personas chosen for the run and which of them are primary, and —
   in a run's later rounds — each persona's reviewer from the earlier rounds,
   the units changed since, whether the text is new to them, and the earlier
   flag rows with their disposition (applied, or declined with the reason, among
   others). The run state is in the conversation, or in the run dir files the
   request names: `scope.md` for the purpose, scope, section set, status and
   design decisions, rubric, verified facts, and link targets; `changes.md` for
   the units changed, the `text: new|revised` line, and the dispositions;
   `matrix.md` for the earlier rows. When invoked standalone, default to: the
   shipped document named in the profile's Document section, as it stands; all
   personas listed in the profile's Personas section; implemented status, when
   the status dimension is enabled; no rubric. The request may also name a
   directory for the round's files — an authoring run's run dir — and may state
   that the reviewers from earlier rounds cannot be continued.

2. **Compose the review brief** to a temp file — `brief.md` in the directory the
   request names, when it names one: the project and document identity, from the
   profile's Document section; the document's purpose; the declared quadrant(s)
   and matching reader question(s), carried as the profile's Declaration
   mechanism section directs; when the status dimension is enabled, each unit's
   status, and the design decisions for spec content — the brief is
   self-contained; what is in and out of scope, and whether the section set is
   an output of the run; the rubric, when the request has one; verified facts;
   link targets; and the path of the Diátaxis core, so reviewers read the core
   from the brief. Reviewers never depend on files outside the repository, this
   skill, and the brief. In a run's later rounds, update the existing brief in
   place — the draft path(s), the verified facts, what the round changed —
   rather than composing a new one, reading `scope.md` and `changes.md` afresh
   when the request names them; its path stays the same.

3. **Launch or continue the panel.** In a run's first round, launch one
   `reviewed-writer:persona-reviewer` subagent per persona in the request via
   the Agent tool's `subagent_type`, all in parallel, and keep each reviewer's
   agent ID for the run. Each task prompt gives, in order, the path of the
   persona's head file from the profile's Personas section, the brief path, and
   the draft path(s). In every later round, continue each persona's existing
   reviewer with the SendMessage tool instead of launching a new one, all in one
   message so they run in parallel: the path of the text now under review, the
   units changed since its last review, the brief path when the brief changed
   since then, and its earlier flag rows by matrix number with their
   disposition; say when the text is new to it, as after synthesis —
   `changes.md`'s `text:` line says so when the request names that file; it
   re-reads the text at that path and returns the follow-up form, or the full
   report when the text is new. Wait for the replies; never poll. Launch afresh
   only a reviewer that no longer exists, with the first-round task prompt's
   shape — the head file path, the brief path, and the path of the text now
   under review — plus its earlier rows and their dispositions; it does a full
   review. When the request states that the reviewers cannot be continued,
   launch every reviewer afresh the same way and keep the new IDs for the run.
   If the session cannot continue reviewers, say so in the report — the run's
   rounds then cost full reviews. The report's shape is the agent's own
   contract; do not restate it.

4. **Collect and consolidate.** If a reviewer errors out, returns no `Verdict:`
   line, or cannot be continued, re-launch it — do not treat a missing verdict
   as a pass. Build the matrix from the reports' tables, shorter than any one
   report: each persona's verdict line, marked `(primary)` for a primary
   persona; with a rubric, scores by axis and draft and the best draft by count
   of personas; a relevance grid, units as rows and personas as columns — a
   primary persona's column marked `(primary)` — with the marker read for each
   unit and any disagreement; then the flag rows, numbered across the run,
   `blocking` first, grouped by unit, one row per finding naming the personas
   raising it and keeping its kind, destination, and fix. In a later round,
   update the previous matrix — read from `matrix.md` when the request names a
   directory — rather than rebuilding it: replace each persona's verdict line
   and, with a rubric over drafts, its scores; mark earlier rows landed,
   withdrawn, or still open as the follow-ups report — a shared row stays open
   while any persona that raised it holds it open; merge a re-raised finding
   into its existing row and otherwise append new rows with the next numbers, a
   relaunched reviewer's name leaving the rows it no longer holds; and when the
   text is new to the reviewers, rebuild the relevance grid from their full
   Units tables and drop the draft scores. Report the matrix to the caller, and
   when the request names a directory, write it to `matrix.md` there as well.
