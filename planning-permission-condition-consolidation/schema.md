# Canonical conditions schema

Use a structure equivalent to the following. Adapt labels to the governing regime, but do not remove provenance or scope fields.

```json
{
  "schema_version": "1.0",
  "subject": {},
  "principal_instrument": {
    "reference": "",
    "decision_date": null,
    "source": {"document": "", "page": null}
  },
  "operating_level": 1,
  "scope_note": "",
  "unresolved_gaps": [],
  "condition_numbering_note": null,
  "family_index": {
    "amendments_identified": [],
    "discharge_applications_identified": []
  },
  "taxonomy": {
    "trigger_stage_options": [],
    "category_options": [],
    "scope_options": [],
    "scope_ancestors": {}
  },
  "consolidated_conditions": []
}
```

Each condition should include:

```json
{
  "condition_number": "",
  "heading": "",
  "condition_group": null,
  "subject_summary": "",
  "original_text": "",
  "current_text": "",
  "current_reason": "",
  "amendment_history": [
    {
      "version": 1,
      "amending_reference": null,
      "date": null,
      "full_text": "",
      "text_summary": "As originally granted",
      "source": {"document": "", "page": null}
    }
  ],
  "triggers": [{"stage": "", "scope": [], "qualifier": null}],
  "categories": [],
  "scope": [],
  "discharge_applications": [],
  "discharge_status": "not_assessed",
  "discharge_rationale": null,
  "current_source_ref": {"document": "", "page": null}
}
```

At Levels 1–2, use `not_assessed`; do not infer a status from missing data. At Level 3 the closed set is:

- `no_discharge_data`
- `pending_discharge`
- `partially_discharged`
- `discharged`

Keep the source's decision wording on each application rather than normalizing it away.

## Validation

Programmatically verify:

- condition count and identifiers reconcile to the source, allowing documented defects;
- every condition has non-empty original/current text and amendment history;
- amendment and discharge targets resolve to real condition identifiers;
- the final amendment version equals `current_text`;
- taxonomy values belong to the declared closed sets;
- every scope is represented in `scope_ancestors`;
- every Level 3 status belongs to the closed set;
- every Level 3 status other than `no_discharge_data` has a substantive rationale;
- every source pointer names a real reviewed document and valid page/block.
