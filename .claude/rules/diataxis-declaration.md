# Diátaxis declaration

How units of content in this repository's documents declare their Diátaxis
quadrants. The review rules themselves — the reader questions, the per-quadrant
guidance, the restructuring rules, and the reviewers' self-check — are the
shared core in the `reviewed-writer` plugin's `persona-review` skill
(`references/diataxis-review.md` in its directory); this file supplies only the
marker legend, the section shapes, and the declaration record.

## Markers

A unit of content is a section of `README.md` or `CONTRIBUTING.md`. It declares
its quadrant with a visible marker in its heading; the markers are deliberately
not explained in the documents themselves:

| Quadrant    | Marker |
| ----------- | ------ |
| tutorial    | 🎓     |
| how-to      | 🔧     |
| reference   | 📋     |
| explanation | 📖     |

The marker sits immediately after the `#` hashes and before the heading text,
separated by single spaces — `## 🔧 Install`. In the core's declaration pass, a
well-formed marker sits in the section's heading with a value from this legend.
The tutorial quadrant is currently unused; it becomes relevant only if a
document gains a learn-by-doing walkthrough.

## Section shapes

- **Marked section** — a heading that carries one legend marker; that is its
  declared quadrant. Unmarked subheadings belong to the unit and share its
  quadrant.
- **Container** — a heading that groups subsections which each declare their own
  quadrant: it carries no marker and at most one orientation sentence of its
  own. The subsections may span quadrants or share one.
- **Front matter** — the document title and any prose before the first `##`
  heading: orientation only (what the project or document is, links), no marker.

An unmarked `##` heading with body content of its own fits no shape; the core's
declaration pass reports it as a unit with no marker.

## Record

The markers in the shipped documents are the authoritative record of their
quadrants; there is no separate table to keep in sync. During a `write-doc` run
each draft declares its own structure the same way — every draft heading carries
its marker — so working declarations travel with the text, and the review brief
carries each unit's status and reader question, not its quadrant. Changing a
marker while keeping the content — reclassification — is a scoping decision,
never a review-round outcome.
