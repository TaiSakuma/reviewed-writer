# Contributing

## Setup

After cloning, install the [pre-commit](https://pre-commit.com/) hook so the
checks run at commit time:

```bash
pre-commit install
```

The hook runs prettier over markdown (`.prettierignore` keeps it away from
`CHANGELOG.md` and `.github/`). CI runs the same check on every PR. Run it over
the whole tree with `pre-commit run --all-files`; note that it only covers files
tracked by git, so `git add` new files first.

## PR Title Convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/)
for **PR titles**. Since we squash-merge, the PR title becomes the final commit
message.

### Format

```text
type: description
```

### Allowed Types

| Type       | Purpose                                                 |
| ---------- | ------------------------------------------------------- |
| `feat`     | A new feature                                           |
| `fix`      | A bug fix                                               |
| `docs`     | Documentation only                                      |
| `style`    | Code style (formatting, semicolons, etc.)               |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf`     | Performance improvement                                 |
| `test`     | Adding or updating tests                                |
| `build`    | Build system or external dependencies                   |
| `ci`       | CI configuration                                        |
| `chore`    | Other changes that don't modify src or test files       |
| `revert`   | Reverts a previous commit                               |

Append `!` to indicate breaking changes (e.g., `feat!: description`).

### Examples

- `feat: add user authentication`
- `fix: handle empty input`
- `docs: update installation instructions`
- `feat!: remove get_user()`

### Individual Commits

Individual commit messages within a PR are free-form. Only the PR title is
enforced.

## Releasing

Releases use a two-tag flow. The `u` tag triggers changelog generation, which in
turn creates the `v` tag and GitHub Release. The pipeline assumes one release in
progress at a time: push the next `u` tag only after the previous release has
appeared.

### Steps

1. **Bump the version:**

   Run the repo-local `/bump-version` skill in Claude Code with `patch`,
   `minor`, `major`, or an explicit version. It updates the `version` field in
   `.claude-plugin/plugin.json`, creates the bump commit
   (`Bump version <old> → <new>`), and tags it `u<version>` (annotated).

   Without Claude Code, do the same by hand:

   ```bash
   # edit the "version" field in .claude-plugin/plugin.json, then:
   git add .claude-plugin/plugin.json
   git commit -m "Bump version 0.1.0 → 0.2.0"
   git tag -a u0.2.0 -m "Bump version 0.1.0 → 0.2.0"
   ```

2. **Push the tag:**

   ```bash
   git push origin u<version>
   ```

   Push only the `u` tag, not `main --tags`: a stale local `latest` tag makes
   `--tags` fail. GitHub Actions publishes the changelog commit and the other
   tags back to you.

3. **Wait for CI:**
   - The **Changelog** workflow first verifies that the tag matches the
     `version` in `.claude-plugin/plugin.json` at the tagged commit and fails
     otherwise. It then creates a `release/<version>` branch at the tagged
     commit, generates `CHANGELOG.md` and the `v` tag on it, then merges the
     branch back into `main` (a backport, cut from a commit off `main`'s line,
     keeps the branch and leaves `main` untouched).
   - The **Release** workflow then creates a GitHub Release with categorized
     notes, marking it latest only when it is the newest version.

### Releasing from an older commit

Check the commit out first (`git switch --detach <commit>`) and release from
there. The chosen commit must already carry the pipeline's workflow files: a tag
push runs the workflow files as of the tagged commit, and a tag on a commit
without them starts no run at all.

### If the release fails

If the "Generate changelog" run fails, a "Release a new version" run still
appears, but its job is skipped and no release is created. Delete the trigger
tag on GitHub (`git push origin --delete u0.2.0`), delete the `release/0.2.0`
branch if the failed run left one behind
(`git push origin --delete release/0.2.0`), and, if the failed run had already
created the `v` tag (visible under the repository's tags), delete that tag too
(`git push origin --delete v0.2.0`). If the version-check step failed, the local
bump commit records the wrong version: fix `.claude-plugin/plugin.json` (or redo
the bump) and move the `u` tag to the corrected commit. Then fix any other cause
and push the trigger tag again (`git push origin u0.2.0`).

If instead the "Release a new version" run fails after a successful changelog
run (for example, a transient error while creating the GitHub Release), the `v`
tag and the changelog are already correct: re-run that workflow run from the
Actions tab; do not delete any tags.

### After the release

After a merge-back, pull the changelog commit and the new tags:
`git pull --tags --force origin main` (`--force` lets the moved `latest` tag
update; without it the fetch is rejected). When the merge-back was skipped (a
backport), `main` has nothing new; `git fetch --tags --force origin` retrieves
the release branch and the tags.
