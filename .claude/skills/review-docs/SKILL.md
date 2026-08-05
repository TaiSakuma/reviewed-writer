---
name: review-docs
description:
  Review README.md or CONTRIBUTING.md once with the persona panel and report; do
  not modify
argument-hint: "[README.md|CONTRIBUTING.md]"
---

The single-round counterpart of `write-docs`: the same panel, one round, no
revision.

## Steps

1. **Review.** Invoke the `reviewed-writer:persona-review` skill on the named
   document (`README.md` or `CONTRIBUTING.md`) as it stands, with its standalone
   defaults: every persona the profile lists, implemented status, no rubric.
2. **Report.** Present the consolidated matrix — each persona's verdict and
   flags, per-section relevance, the reviewers' declaration passes, and proposed
   fixes. Do not modify the document; route anything needing rework to
   `write-docs`. Acting on the report is the user's decision.
