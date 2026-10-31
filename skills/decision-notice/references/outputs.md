# Decision notice: templates and examples

Load at the Emit and Respond steps. Templates fix the shape of the output; examples show common mistakes next to the correct version. All references are synthetic.

## Templates

### T1 · Instrument summary

```markdown
**[Instrument kind] [reference(s)] · [authority] · decided by [decision maker] ([route])**
Site (as printed): [address] · Decision (p[n]): "[decision wording]" ([outcome])
Dates: [label: value; …]
Description (operative, p[n]): "[verbatim, shortened with … if long]"
Conditions: [n] · Approved plans: [n] (listed in condition [n]) · Informatives: [n]
Stated links: [relationship] → [target as cited] ([in graph / unresolved])
Flags: [code: one line each, or "none"]
```

### T2 · Conditions list

```markdown
**Conditions of [reference]** ([n] printed, pp[n–n])

| No. | Heading | Trigger (as worded) | Condition (verbatim, shortened) | Reason (shortened) | Page | Linked in graph |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | — | "within three years" | "The development hereby permitted shall be begun before…" | "To comply with Section 91…" | 2 | — |
| 7 | Prior to commencement | "No development shall take place until…" | "…a Construction Management Plan has been submitted to and approved…" | "In the interests of highway safety." | 3 | EX/20/0990/DOC, approved in part [I] |
```

"Linked in graph" lists the decisions and later instruments linked to the condition, or "—". An empty cell says nothing about discharge (CORE-05, COND-03).

### T3 · Specific condition

```markdown
**Condition [n] of [reference]** ([heading], p[n])
"[verbatim condition text]"
Reason: "[verbatim reason]"
Trigger as worded: "[phrase]"
Linked in graph: [decision ref, wording, evidence level] / [s73 or s96A instrument that varies it] / none
Reading note: [only rule-based context, e.g. S73-02 or READ-04; no conclusions]
```

### T4 · Reply footer (every reply)

```markdown
---
Instruments reviewed: [refs] · Context only: [docs or "none"]
Graph: [location] r[n]
```

### T5 · Exceptions report

```markdown
# Exceptions: instruments added at graph r[n]
1. Unreadable or missing: [page / schedule / plan list, effect]
2. Numbering and counts: [condition numbering defects; stated vs extracted]
3. Flags for review: [code: subject, page, one line]
4. Unresolved references cited: [ref / condition, cited by, page]
Validation level reached: [graph-valid / …]
```

### T6 · Completion reply (instruments added, no question asked)

```markdown
Added [n] instruments ([refs]) to the matter graph as r[n].
[n] permissions, [n] conditions, [n] other decisions; [n] stated links ([n] to unresolved references). Promoted from search evidence: [refs or "none"].
Exceptions: [n] (top items: [one line each, at most three]).
Exceptions report: [location] · Validation: [level]
```

No tables. Offer T1, T2 or T3 in one line, and @permission-family where unresolved references remain.

## Examples

Each pair shows the mistake (✗) and the correct version (✓).

**E1 · s96A treated as a new permission (S96A-01)**
✗ A `permission` node for EX/21/0789/NMA
✓ A `decision` node with `procedure: s96a`, linked to its stated target by `amends_under_s96a`, recording the exact change

**E2 · Issue date inferred (EVID-06, PERM-07)**
✗ "Issued: 12/03/2020", copied from the committee resolution date
✓ "Committee resolution: 18/12/2019; decision: 12/03/2020 (as printed). Issue date: not stated."

**E3 · Search description treated as operative (PERM-05)**
✗ Operative description "Erection of dwellings" (the truncated search wording)
✓ `description_source` (search): "Erection of dwellings"; `description_operative` (notice p1): "Erection of up to 120 dwellings with associated access…". No conflict: the labels differ.

**E4 · s73 conditions recorded as changes only (S73-01, S73-02)**
✗ EX/22/0111/VAR recorded with condition 4 only, and condition 4 of the outline overwritten
✓ A new `permission` with all 22 conditions printed on it, `varied_conditions: ["4"]` and `varies_under_s73` to the outline. The outline node is unchanged.

**E5 · Numbering defect silently corrected (COND-06)**
✗ Conditions recorded as 1–14
✓ Conditions recorded as printed (1–8, 8, 10–14), each with its own `condition_id`, and flag `numbering_defect`

**E6 · Informative recorded as a condition (READ-01)**
✗ "Condition 15: The applicant is reminded that a Section 106 agreement…"
✓ An entry in `informatives[]`, plus `relates_to_obligation` to an `unresolved_reference` for the s106 agreement

**E7 · Absence of an unprinted condition (READ-04, CUR-02)**
✗ "There is no biodiversity net gain condition."
✓ "No biodiversity gain condition is printed. Application received 14/05/2024 (as printed). ⚑ unprinted_condition_possible."

**E8 · Discharge concluded from a condition decision (COND-02, COND-03)**
✗ "Condition 7 is discharged."
✓ EX/20/0990/DOC: a `decision` with `procedure: condition_approval`, linked by `approves_details_under_condition` to condition 7, with decision wording "approved in part" and outstanding "phase 2 details" recorded verbatim

**E9 · Condition created from a citation (COND-06)**
✗ A `condition` node "EX/19/0123/OUT condition 9" created from a DOC letter, with the outline notice not supplied
✓ An `unresolved_reference` for EX/19/0123/OUT condition 9, with `candidate_parent_node_id` pointing at the outline application from the search

**E10 · Appeal grant attributed to the LPA (PERM-03)**
✗ A `permission` granted by the LPA decision on EX/18/0555/FUL, which was a refusal
✓ The LPA decision stays `refused`. The appeal decision (APP/X1234/W/19/3200000, decided by an Inspector, `decision_route: appeal`) has `appeal_of` to the application and `grants` to the new permission, whose conditions come from the schedule.

**E11 · Candidate links during ingestion (contract, REL-02)**
✗ `possibly_related_to` between two notices because they share a site address
✓ No candidate edge. Family mapping generates candidates.
