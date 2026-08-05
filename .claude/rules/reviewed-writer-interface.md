---
paths:
  - ".claude/rules/persona-review-profile.md"
  - ".claude/rules/diataxis-declaration.md"
  - ".claude/personas/**"
---

# Reviewed-writer interface

These files are this repository's consuming side of the `reviewed-writer`
plugin's interface — the same plugin this repository develops. The machinery —
the `write-doc` and `persona-review` skills and the `persona-reviewer` agent —
arrives through the pin in `.claude/settings.json`, so reviews run with the
pinned release, not the working tree. The profile's `##` headings are read by
name by the plugin's skills; the persona head files and the declaration file are
read by its reviewers. Renaming or restructuring any of them is an interface
change, coordinated with the skills in this repository and with the counterpart
repositories (legendary-octo-happiness and hypothesis-awkward).
