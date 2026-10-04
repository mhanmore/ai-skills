# Amendment correlation and indirect impact

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

## Indirect-impact sweep

Search the complete canonical agreement for:

- every use of an amended defined term;
- every reference to an amended or restructured clause;
- definition-to-definition dependencies;
- renumbered, split, merged, or removed targets;
- schedules, drawings, or appendices referenced by changed text.

Record each hit with `referencing_item`, `referenced_item`, affected `instrument_id`, source, verification status, and note. Default to `verified: false`; mark verified only after the dependency has actually been reviewed.

Programmatically ensure every target and referencing item resolves to the canonical model and every correlation item has both sides or an explicit reason why one side does not exist.
