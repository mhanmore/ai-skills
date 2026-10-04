# Canonical agreement model and completeness gates

## Canonical model

Record top-level agreement metadata, parties, base date, operating level, instruments, unresolved gaps, `scope_note`, structural navigation, canonical sections, correlation items, and cross-reference impacts.

Each textual unit should resemble:

```json
{
  "id": "schedule-3-paragraph-12",
  "kind": "schedule_paragraph",
  "heading": "",
  "parent_id": "schedule-3",
  "order": 120,
  "original_text": "",
  "source": {"document": "", "page": null, "block": null}
}
```

Each instrument should record an identifier, title, date, execution status, executed date if known, relative order if established, source, and provenance for the status determination.

## Completeness gates

- Extract definitions from the opening words of the definitions section through its closing words. Script a comparison between every defined-term pattern in the source extraction and the canonical key list.
- Enumerate clauses, schedules, parts, annexes, and other structural units from the document itself. Compare the expected and extracted sets.
- Capture every textual unit from heading through final words, including unnumbered lead-ins.
- Record non-textual content as a typed structural unit with its real source; do not omit it from the navigation model.
- Validate source pointers and native ordering programmatically before correlating amendments.

A missing base unit is an extraction defect, not an amendment-analysis caveat. Correct it before relying on the canonical model.
