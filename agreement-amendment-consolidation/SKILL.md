---
name: agreement-amendment-consolidation
description: Consolidate a contract, deed, lease, licence, facility agreement, shareholders' agreement, or similar structured agreement against separate amending instruments. Use when the user needs full as-amended text, redlines, amendment correlation, or point-in-time instrument combinations; do not use for statutory numbered-condition regimes with a discharge mechanism.
---

# Agreement Amendment Consolidation

Produce a trustworthy full-text representation of the base agreement with amendments applied or shown inline. The agreement's own words, in native order, are the primary deliverable; the correlation register and impact analysis support that text.

HTML is additive, not automatic. When the user requests an interactive HTML view, also use `render-response-as-html` with [html-rendering-profile.md](html-rendering-profile.md). Otherwise deliver the requested document/data format from the same canonical model.

## Choose the operating level

- **Level 1 — supplied documents:** no independent search for missing or superseding instruments.
- **Level 2 — supplied documents plus search:** search the authorized corpus for later instruments, execution copies, and related family members; report unresolved gaps.
- **Level 3 — verified execution source:** a register, data-room index, or completion record establishes execution status and ordering sufficiently to support a default current-state view.

At Levels 1–2, still allow proposed or unexecuted instruments to be modelled, but identify their status and do not silently present them as the legal current position.

## Workflow

1. Extract every definition, substantive clause, and textual schedule from the base agreement verbatim and in native order. Apply the completeness gates in [canonical-document-schema.md](canonical-document-schema.md).
2. Read every amending instrument in full and correlate each stated change to the canonical base text. Follow [amendment-correlation.md](amendment-correlation.md).
3. Sweep the full base agreement for indirect effects: uses of amended definitions, references to restructured clauses, and dependencies between definitions.
4. Assemble and validate the correlation register, instrument metadata, impact list, unresolved mismatches, and `scope_note`.
5. If the register is large, derive a matter-specific change taxonomy. Do not impose planning-condition trigger or spatial-scope taxonomies on agreements.
6. Calculate full as-amended and redline views using [redline-calculation.md](redline-calculation.md).
7. Deliver the full text and supporting evidence in the requested format. If HTML was requested, pass the canonical model and rendering profile to the HTML renderer; the presentation layer must not redo amendment logic.

## Essential constraints

- Never substitute headings, summaries, or “not amended” placeholders for textual clauses.
- Include chapeaux, lead-in sentences, connective prose, schedules, and definitions—not merely numbered subparagraphs.
- Preserve page/block provenance for base and amending text.
- Verify an instrument's quoted “before” text against the canonical base/current version; flag mismatches rather than silently preferring one.
- Keep execution status separate from inclusion. Draft, engrossed, unsigned, and executed instruments may all be modelled when relevant.
- Do not force plans, drawings, inventories, or other non-textual schedules into prose; preserve their position and include the source artifact. In HTML output, encode it inside the single HTML file rather than creating a sidecar.
- Never guess the order of conflicting instruments. Resolve unaffected text and flag the conflict locally.
