# Planning permission consolidation specification

## Canonical conditions schema

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
  "review_scope": {
    "documents_reviewed": [],
    "family_search": {
      "performed": false,
      "corpus": null,
      "as_of_date": null,
      "limitations": []
    },
    "compliance_assessment": {
      "performed": false,
      "sources_reviewed": [],
      "coverage": "none",
      "limitations": []
    }
  },
  "review_summary": "",
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

When `compliance_assessment.performed` is false, use `not_assessed`; do not infer a status from missing data. When it is true, the closed set is:

- `no_discharge_data`
- `pending_discharge`
- `partially_discharged`
- `discharged`

Keep the source's decision wording on each application rather than normalizing it away.

### Validation

Programmatically verify:

- condition count and identifiers reconcile to the source, allowing documented defects;
- every condition has non-empty original/current text and amendment history;
- amendment and discharge targets resolve to real condition identifiers;
- the final amendment version equals `current_text`;
- taxonomy values belong to the declared closed sets;
- every scope is represented in `scope_ancestors`;
- when compliance assessment was performed, every status belongs to the closed set;
- every assessed status other than `no_discharge_data` has a substantive rationale;
- every source pointer names a real reviewed document and valid page/block.

---

## Condition taxonomy

Derive the taxonomy after reading the complete condition set. Do not copy labels from an earlier matter.

### Triggers

Represent triggers as an array because one condition may have several milestones. Common patterns include pre-commencement, pre-above-ground, pre-occupation, pre-use, during construction, post-completion, ongoing, event-triggered, and time-limited, but include only patterns supported by the sources.

Each trigger should contain `stage`, applicable `scope`, and an optional qualifier preserving the actual milestone.

### Scope hierarchy

Build both a flat option list and a map from each scope to all broader containing scopes:

```json
{
  "unit_1a": ["phase_1", "site_wide"],
  "phase_1": ["site_wide"],
  "site_wide": []
}
```

This hierarchy is required. It supports both “what applies to this unit?” and “what applies only to this unit?” without conflating them.

### Categories and summaries

Derive categories from recurring substantive topics across the full set. Allow a bespoke category where forcing a singleton into a broad bucket would obscure meaning.

Write an approximately 8–15 word subject summary for each condition. It should distinguish mirrored or scope-varied conditions and reflect the current obligation. Do not fold status into this sentence.

Validate 100% coverage using lookup tables or another reviewable scripted mapping rather than hand-editing records individually.

---

## Documentary discharge/compliance assessment

Read the submission, supporting material, decision, and any caveats in full. A headline such as “approved” is evidence, not the complete assessment.

For each condition:

1. Start from the latest amended wording.
2. Decompose it into distinct requirements, including scope, timing, implementation, maintenance, monitoring, and follow-up duties.
3. Map each application to the requirements and geographic/operational scope it addresses.
4. Record the source's formal decision verbatim or faithfully normalized in a separate field.
5. Assess the combined documentary coverage of all applications.

Classify as:

- `discharged`: the reviewed material addresses every requirement and the decision leaves no part outstanding on its face.
- `partially_discharged`: only some requirements or part of the condition's scope are covered, or an approval caveat leaves a material requirement outstanding.
- `pending_discharge`: an application is recorded but no final decision is available, or the outcome is expressly provisional/deferred.
- `no_discharge_data`: no readable discharge/compliance material was identified in the reviewed sources.

For each application record its reference, description, formal decision, decision date, covered and outstanding requirements, notes, and exact source. Write a per-condition rationale explaining the combined effect and any ambiguity.

Do not turn missing evidence into a finding that discharge was unnecessary, a trigger had not occurred, works complied, or the condition remained legally enforceable. Those conclusions require different evidence and may require legal advice.
