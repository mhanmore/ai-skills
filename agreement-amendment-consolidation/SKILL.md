---
name: agreement-amendment-consolidation
description: Consolidate a contract, deed, lease, licence, facility agreement, shareholders' agreement, or similar structured agreement against separate amending instruments. Use when the user needs full as-amended text, redlines, amendment correlation, or point-in-time instrument combinations; do not use for statutory numbered-condition regimes with a discharge mechanism.
---

# Agreement Amendment Consolidation

Produce a trustworthy full-text representation of the base agreement with amendments applied or shown inline. The agreement's own words, in native order, are the primary deliverable; the correlation register and impact analysis support that text.

Read [consolidation-spec.md](consolidation-spec.md) completely before performing the consolidation. HTML is additive, not automatic. When the user requests an interactive HTML view, also use `render-response-as-html` and read [html-runtime-instructions.md](html-runtime-instructions.md) completely. Otherwise deliver the requested document/data format from the same canonical model.

## Establish the review scope

Do not use or expose tier labels. Determine the work from the user's request, readable documents, and authorized sources. Record these independent evidence dimensions:

- **Core consolidation:** extract the complete base agreement and correlate every supplied/readable amending instrument.
- **Family completeness check:** when an authorized searchable corpus is available and the request warrants it, search for later instruments, execution copies, and related family members. Otherwise state that no independent family search was performed.
- **Execution verification:** record whether a register, data-room index, completion record, or other source independently confirms each instrument's status and ordering.
- **Cross-reference review:** state whether the indirect-impact sweep was performed and which findings remain unverified.

Store these facts in `review_scope` and summarize them for users in plain `review_summary` prose. Never ask the user to choose an internal tier. Proposed or unexecuted instruments may still be modelled, but identify their status and do not silently present them as the legal current position.

## Workflow

1. Extract every definition, substantive clause, and textual schedule from the base agreement verbatim and in native order. Apply the completeness gates in the consolidation specification.
2. Read every amending instrument in full and correlate each stated change to the canonical base text using the consolidation specification.
3. Sweep the full base agreement for indirect effects: uses of amended definitions, references to restructured clauses, and dependencies between definitions.
4. Assemble and validate the correlation register, instrument metadata, impact list, unresolved mismatches, `review_scope`, and `review_summary`.
5. If the register is large, derive a matter-specific change taxonomy. Do not impose planning-condition trigger or spatial-scope taxonomies on agreements.
6. Calculate full as-amended and redline views using the consolidation specification.
7. Deliver the full text and supporting evidence in the requested format. If HTML was requested, pass the canonical model and rendering profile to the HTML renderer; the presentation layer must not redo amendment logic.

## Essential constraints

- Never substitute headings, summaries, or “not amended” placeholders for textual clauses.
- Include chapeaux, lead-in sentences, connective prose, schedules, and definitions—not merely numbered subparagraphs.
- Preserve page/block provenance for base and amending text.
- Verify an instrument's quoted “before” text against the canonical base/current version; flag mismatches rather than silently preferring one.
- Keep execution status separate from inclusion. Draft, engrossed, unsigned, and executed instruments may all be modelled when relevant.
- Do not force plans, drawings, inventories, or other non-textual schedules into prose; preserve their position and include the source artifact. In HTML output, encode it inside the single HTML file rather than creating a sidecar.
- Never guess the order of conflicting instruments. Resolve unaffected text and flag the conflict locally.
