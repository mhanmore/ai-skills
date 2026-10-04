# Agreement amendment consolidation specification

## Canonical agreement model and completeness gates

### Canonical model

Record top-level agreement metadata, parties, base date, instruments, unresolved gaps, `review_scope`, `review_summary`, structural navigation, canonical sections, correlation items, and cross-reference impacts.

`review_scope` should state which documents were reviewed; whether an independent family search was performed and against which corpus; whether execution status and order were independently verified and from what source; whether the cross-reference sweep was performed; and the limitations of each activity. `review_summary` should express the same material facts in short, user-facing prose without exposing schema terminology.

```json
{
  "review_scope": {
    "documents_reviewed": [],
    "family_search": {
      "performed": false,
      "corpus": null,
      "as_of_date": null,
      "limitations": []
    },
    "execution_verification": {
      "performed": false,
      "sources_reviewed": [],
      "ordering_verified": false,
      "limitations": []
    },
    "cross_reference_review": {
      "performed": false,
      "unverified_findings": 0,
      "limitations": []
    }
  },
  "review_summary": ""
}
```

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

### Completeness gates

- Extract definitions from the opening words of the definitions section through its closing words. Script a comparison between every defined-term pattern in the source extraction and the canonical key list.
- Enumerate clauses, schedules, parts, annexes, and other structural units from the document itself. Compare the expected and extracted sets.
- Capture every textual unit from heading through final words, including unnumbered lead-ins.
- Record non-textual content as a typed structural unit with its real source; do not omit it from the navigation model.
- Validate source pointers and native ordering programmatically before correlating amendments.

A missing base unit is an extraction defect, not an amendment-analysis caveat. Correct it before relying on the canonical model.

---

## Amendment correlation and indirect impact

For each instrument, identify its change convention: delete/insert markup, replacement wording, restatement, renumbering, substituted schedule, or another explicit mechanism.

For every stated change capture:

- `instrument_id`;
- target identifier;
- the instrument's quoted before text;
- canonical before text at the relevant point in the amendment sequence;
- proposed/after text;
- change mechanism;
- base and instrument sources;
- mismatch status and resolution;
- housekeeping/substantive classification with a one-sentence reason;
- optional matter-derived change categories.

Treat mismatches between the instrument's recital and canonical text as first-class evidence. Quote both versions and resolve only from authoritative material; otherwise preserve the uncertainty.

### Indirect-impact sweep

Search the complete canonical agreement for:

- every use of an amended defined term;
- every reference to an amended or restructured clause;
- definition-to-definition dependencies;
- renumbered, split, merged, or removed targets;
- schedules, drawings, or appendices referenced by changed text.

Record each hit with `referencing_item`, `referenced_item`, affected `instrument_id`, source, verification status, and note. Default to `verified: false`; mark verified only after the dependency has actually been reviewed.

Programmatically ensure every target and referencing item resolves to the canonical model and every correlation item has both sides or an explicit reason why one side does not exist.

---

## As-amended text and redline calculation

The primary output is the complete agreement text in native order. For a selected set of instruments:

1. Render every canonical textual unit.
2. Leave untouched text verbatim and unmarked.
3. Apply enabled instruments in established order.
4. For amended text, provide both a clean calculated version and a word/phrase-granularity redline when requested.
5. Associate editorial cross-reference warnings with the affected text and enabled instrument.

Use deletion and insertion markup plus a second visual cue such as colour in rendered formats; never rely on colour alone. Keep change-category badges visually distinct from redline semantics.

Execution status is not a calculation gate. The real gates are:

- canonical extraction passed completeness validation;
- quoted before text was checked against the correct prior version;
- indirect-impact sweep ran for the instrument;
- application order is established or ambiguity is explicitly contained.

If two enabled instruments conflict and their order cannot be established, do not synthesize a winner. Mark the affected unit unresolved while continuing to calculate unaffected units. Disabling an instrument must remove its changes and its dependent warnings.

Non-textual schedules remain typed attachments/placeholders at their native structural position. Never summarize a replacement drawing as though that produced consolidated textual content. In HTML output, encode every included schedule attachment inside the HTML file.
