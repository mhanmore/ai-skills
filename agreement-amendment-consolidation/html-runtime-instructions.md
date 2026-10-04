# HTML runtime instructions: full agreement redline

Pass this profile and the validated canonical agreement model to `render-response-as-html` when HTML is explicitly requested.

## Preferred view

A full-document reading experience, not a change-log table. Reproduce every textual unit in native order and derive navigation from the actual document structure: clauses/schedules, articles/annexes, parts/exhibits, or whatever the source uses.

## Required capabilities

- Full-text reading view containing amended and unamended text.
- Navigation to useful structural units; omit a sidebar when the document is too short or flat for one.
- Full-text search across the agreement.
- One enable/disable control per instrument, labelled with identity and execution status.
- A clear statement of the enabled combination and unresolved impacts that updates with the controls.
- Inline redlines for enabled changes and clean treatment of untouched text.
- Expandable supporting evidence: classification, reason, mismatch/verification notes, and citations.
- Inline unresolved-order/conflict warnings without guessing a result.
- Non-textual schedules represented at their native position, encoded into the HTML attachment registry, and accessible through separate View and Download controls.

## Semantic invariants

- Instrument toggles control already-modelled amendment layers; JavaScript must not re-interpret source language.
- When execution status and order are independently verified, a sensible default may enable executed instruments in that verified order. Otherwise default conservatively and state the starting combination explicitly.
- Never disable a toggle merely because an instrument is draft or unsigned.
- An unamended clause has no artificial badge or disclosure control.
- Keep the redline legend separate from change-category legends.
- The scope banner must present `review_summary` in ordinary language and must not conceal execution uncertainty or open cross-reference flags. Keep structured review details available behind an “About this consolidation” disclosure rather than exposing raw schema keys.
- Every included source or attachment must be encoded in the HTML attachment registry and recoverable byte-for-byte through a Download control; linked sibling and sidecar files are prohibited.
