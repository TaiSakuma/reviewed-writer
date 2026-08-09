---
name: bump-version
description:
  Bump the plugin version in .claude-plugin/plugin.json, create the bump commit,
  and tag it u<version> to trigger the release pipeline
argument-hint: "[patch|minor|major|X.Y.Z]"
---

Bump the plugin version and create the `u<version>` trigger tag — the manual
step that starts the release pipeline described in `CONTRIBUTING.md`'s Releasing
section. Invoking this skill is the user's request to create the bump commit and
tag; nothing here is pushed. The bump commit also refreshes the current-release
literals in the documents and the pin in `.claude/settings.json`; the pipeline's
merge-back carries them to `main`, and skips them for a backport — deliberately,
since `main`'s pin must not move to an older version.

## Steps

1. **Argument.** The invocation supplies `patch`, `minor`, `major`, or an
   explicit version `X.Y.Z`. If it supplies none, ask which to use.

2. **Preconditions.** Abort with a clear message if any fails:

   - The working tree is clean: `git status --porcelain` prints nothing.
   - HEAD carries the pipeline:
     `git cat-file -e HEAD:.github/workflows/changelog.yml` succeeds. A tag on a
     commit without the workflow files starts no workflow run at all.
   - HEAD is the commit to release — normally `main`'s pulled head; for a
     release from an older commit the user has already run
     `git switch --detach <commit>`.

3. **Compute the new version.** Read the current version with
   `jq -r .version .claude-plugin/plugin.json`. A bump rule increments its
   component and zeroes the lower ones. An explicit version must match
   `^[0-9]+\.[0-9]+\.[0-9]+$` and be greater than the current version: the
   pipeline's `u*.*.*` trigger and its version ordering (`sort -V`) assume plain
   three-part versions, with no pre-release or build suffixes.

4. **Collision check.** Abort if `u<new>` or `v<new>` already exists locally
   (`git rev-parse -q --verify refs/tags/<tag>`) or on origin
   (`git ls-remote --tags origin`).

5. **Edit.** Change the `version` value in `.claude-plugin/plugin.json` with the
   Edit tool. Do not round-trip the file through `jq`, which would reformat it.

   Then sweep the current-release literals from the old version to the new one:

   - `.claude/settings.json` — the pin's `ref` value.
   - `README.md` — every statement of the current release: the pin example's
     `ref`, the `@v<old>` re-registration command, and the Versioning examples.
   - `CONTRIBUTING.md` — the "as this is written" value in the plugin-install
     section.

   Two exclusions: every version in `CONTRIBUTING.md`'s Releasing section stands
   for an arbitrary version and stays as it is, and `CHANGELOG.md` is
   CI-generated and never edited. Check the sweep by grepping the three files
   for the old version — the only remaining hits must be in the Releasing
   section.

6. **Commit and tag.** The commit must exist: the pipeline derives the released
   commit as the tag's parent.

   ```bash
   git add .claude-plugin/plugin.json .claude/settings.json README.md CONTRIBUTING.md
   git commit -m "Bump version <old> → <new>"
   git tag -a "u<new>" -m "Bump version <old> → <new>"
   ```

   The message is deliberately not a Conventional Commits line: git-cliff
   filters unconventional commits, which keeps the bump commit — the literal
   sweep included — out of `CHANGELOG.md`. If the pre-commit prettier hook
   reflows a swept file and aborts the commit, re-stage and commit again.

7. **Hand off.** Do not push. Tell the user the next step is
   `git push origin u<new>` — push only the tag, never `--tags` (a stale local
   `latest` tag makes `--tags` fail) — and point to `CONTRIBUTING.md`'s
   Releasing section for what CI does next and how to verify it. Remind the user
   that the checked-in `ref` does not re-point machines that already registered
   the marketplace: after CI completes, each re-registers with
   `/plugin marketplace add TaiSakuma/reviewed-writer@v<new>`.
