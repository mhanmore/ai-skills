---
label: Decision Notice
description: >-
  Use when the user asks to read, summarise, extract from or answer questions
  about an English planning instrument: an LPA decision notice, an appeal
  decision letter, a reserved-matters approval, a condition decision or a
  section 96A non-material amendment. Trigger on requests such as "read the
  decision notice for 19/01234/FUL", "what conditions are on this
  permission?", "what does condition 7 require?" or "add these notices to the
  matter graph". Do not activate for local authority searches, which @local-search
  handles; for mapping permission families, which @permission-family
  handles; or for advising whether a permission is implemented,
  lawful, extant or complied with.
userFacingDescription: >-
  Reads planning decision notices, appeal decisions and approvals into a
  structured, page-referenced record of the permission, its conditions,
  approved plans and stated links, and answers questions from it, flagging
  gaps and points requiring human review.
---

# Decision notice

Status: draft · Rules version: 0.3-draft. Record the rules version in the graph manifest (PKG-07).

Typical prompts: "read the decision notice and list the conditions", "what does condition 7 of 19/01234/OUT require?", "here are the notices from the seller; add them to the matter graph".

Every request goes through the same two stages:
1. **Add the instruments to the matter graph, or reuse them if already there** (PKG-01). Each instrument contributes its application, decision and, for a grant of planning permission, its `permission` and `condition` nodes, with the links it states, each traced to page-level evidence.
2. **Answer the request from the graph.**

This skill is the only one that creates `permission` and `condition` nodes. It records what each instrument says. It does not map families, generate candidate links, consolidate conditions across instruments, or assess discharge, implementation or compliance (CORE-04, COND-03).

## Locate the input

1. Use documents attached to the conversation or named in the prompt.
2. Otherwise, look in the matter's documents not yet in the graph (PKG-07). If it is unclear which are instruments, ask.
3. Find the matter graph (PKG-01), or start one. Skip any document it already holds, and go to [Respond](#respond) if nothing new remains.

## Load

Read these completely before starting. Rule IDs cited below are defined in them.

1. [core.md](../../resources/planning/core.md)
2. [permissions.md](../../resources/planning/permissions.md)
3. [outputs.md](references/outputs.md), the templates and examples, read at the Emit and Respond steps

## Contract

| | |
| --- | --- |
| Input | One or more PDFs holding planning instruments; optional context documents |
| Output | A new revision of the matter graph (PKG-01 to PKG-03), plus an exceptions report beside it. Then a reply answering the request. |
| Node types created or enriched | `application`, `decision`, `permission`, `condition`, `authority`, `document`, `unresolved_reference` |
| Relationships | Stated links only (REL-05, FAM-08): `determines`, `grants`, `varies_under_s73`, `amends_under_s96a`, `reserved_matters_under`, `approves_details_under_condition`, `appeal_of`, `corrects`, `relates_to_obligation`, `reported_as_related_to` |
| Rule overrides | REL-02: no `possibly_related_to` candidates. Candidates are generated at family mapping, as HAND-03 does for searches. |
| Not produced | Family views, candidate links and the documents-to-obtain list (@permission-family); consolidated condition wording, discharge status and the COND-04 unapproved-condition flag (a planned Permission Conditions skill) |
| Validation level claimed | `graph-valid` at most, plus `type-valid` once schemas exist |

## Workflow

### 1. Inventory and classify (READ-05, READ-06)

- Record each document's identity and page count (PKG-07).
- Read each document with [@ kind="capability" id="document-reading" label="Document reading"]. Extract text page by page first. Decision notices are often scans: inspect a page visually where it has little or no text, a stamp, signature or handwritten date, or a conditions table whose layout is lost. Record the method used for each page in the manifest (PKG-02).
- A single PDF may bundle several instruments, often with plans or letters between them. Split them by page range and assign every page to an instrument, an attachment (plans, schedules) or `context`.
- Classify each instrument: LPA decision notice, appeal decision letter, reserved-matters notice, condition decision, s96A letter or notice, correction notice.
- Officer and committee reports, application forms and portal printouts are context (READ-06). Record them as `document` nodes, with the references and links they cite as `reported_as_related_to` at `portal_record` or as `unresolved_reference`s. Never take operative content from them.
- A planning obligation deed is recorded as a `document` only. Do not extract its obligations.
- If the authority is Welsh, extract as normal, set `jurisdiction: Wales` and flag `jurisdiction_not_validated` (CORE-01).

### 2. Extract each instrument (READ-01, READ-05)

Keep all wording verbatim:
- reference(s), including any appeal (APP/…) and portal references (PERM-04); the authority and the decision maker and route (PERM-03);
- every labelled date (EVID-06, PERM-07). Never infer an issue date;
- the site address as printed (PERM-06);
- the operative description (PERM-05);
- the decision wording and outcome (VOCAB-05 outcome aliases);
- for hybrid permissions, each component's full or outline treatment and its reserved matters (READ-03);
- conditions, each with its number as printed, group heading, verbatim text, reason and trigger wording (COND-06, COND-04);
- the approved plans and documents list (number, revision, title, date), often inside a "plans" condition, and any document incorporated by reference (READ-02);
- informatives, kept separate from conditions;
- time-limit wording as printed (PERM-08);
- every cited reference: parent permission, condition numbers, obligations, earlier approvals.

By instrument type:
- **s73 grant:** extract the complete condition set of the new instrument, including conditions restated unchanged (S73-02). Record which conditions it says it varies or removes, and any cross-reference to earlier approvals. Flag `description_changed_under_s73` or `time_limit_extended_under_s73` where the instrument does so (S73-03), and `obligation_linkage_not_evidenced` unless a deed of variation or linking clause is cited (S73-04).
- **Appeal letter:** the "Decision" paragraph and the conditions schedule, usually at the end. Any costs decision is recorded separately and creates no permission.
- **Condition decision:** the condition numbers and parent reference, the submitted documents, the decision wording ("approved", "discharged in part"), what remains outstanding and any continuing duties (COND-02).
- **s96A:** the target as stated, the exact changes, replacement plans and any amended or new conditions (S96A-02, S96A-04).
- Flag `unprinted_condition_possible` where CUR-02 may apply (READ-04), and `unincorporated_reliance` where the notice relies on application material it does not incorporate (READ-02).

Data minimisation: omit applicants' and agents' personal names unless the task needs them; record `personal_data_omitted: true`.

### 3. Build nodes (PKG-04, PKG-05, PKG-06)

**Evidence model.** Each instrument section (header, description, decision, each condition, plans list, informatives) carries one `source_reported` assertion with its page, locator and a short quote. Derived values cite it. Add a field-level assertion only for a value that is normalised, inferred or conflicting.

- **Merge on identity** (PKG-05): authority plus reference. A search-built `application` or `decision` gains the instrument's assertions; its search assertions stay. Raise `evidence_level` to `instrument_reviewed` (PKG-06), and never lower one. A truncated search description against the operative description is not a conflict; a different decision date under the same label is `conflicting`.
- **`application`** and **`decision`** for every instrument, with `procedure` from the instrument's wording or the alias table (VOCAB-05), and `determines` between them.
- **`permission`** only for a grant of planning permission, by the LPA or on appeal (PERM-02, VOCAB-01), linked from its decision by `grants`, with `permission_form`. s73 and s73A grants are new permissions (S73-01, S73-05); never change the earlier node. s96A, reserved-matters, condition and ROUTE-01 decisions and refusals create no permission.
- **`condition`** for each printed condition of a reviewed permission, with a stable `condition_id` and `data.permission_id` (COND-01, COND-06). Keep numbering as printed, including duplicates and gaps, and flag `numbering_defect`.
- **Stated links** (REL-05, FAM-08), recorded with the instrument's wording and `evidence_level: instrument_reviewed`:
  - a link to a reviewed permission or condition targets that node;
  - a link to a permission not yet reviewed targets its `application` if it is in the graph, otherwise an `unresolved_reference` (REL-03);
  - a link to a condition of a permission not yet reviewed targets an `unresolved_reference` for the permission and condition number, holding `candidate_parent_node_id` where the application is in the graph (COND-06);
  - a citation whose relationship the instrument does not state is `reported_as_related_to`;
  - record the target exactly as cited, even where a later s73 branch exists. Branch ambiguity is flagged at family mapping (FAM-09).
- **Re-targeting** (PKG-06): when this run reviews a permission that earlier links point at (an application or `unresolved_reference` targeted by a search or an earlier instrument), add the typed link to the new `permission` or `condition` node. On each `unresolved_reference` now matched, add an assertion setting `data.resolved_node_id`. Never delete the earlier link or reference.
- **`authority`**: one per named body, including Inspectors' and Secretary of State decisions as decision makers (PERM-03, CUR-07).

### 4. Validate

Run these checks once over the finished extraction. Fix any failure, re-run only the checks it affects, then stop.

- Every PKG-02 key is present, IDs are unique, and every link resolves to a node or an `unresolved_reference`.
- Every page is assigned to an instrument, attachment or context.
- Every printed condition number has a `condition` node with text and reason, or a `value_state`; the count matches the schedule, or the difference is a gap or `numbering_defect`.
- Every grant of planning permission has `determines`, `grants` and a `permission`; nothing else has a `permission`.
- Every condition approval and s96A decision links to its stated target.
- Every date keeps its label, and no date is derived from another; no use class is translated (CUR-01).

Go back to the page image only for flagged items, numbering defects, and values read as illegible or conflicting. Record what was checked in the manifest.

### 5. Emit

Write the graph revision (PKG-03), then the exceptions report (template T5). These two files are the whole output of an ordinary run; write diagnostics only in test mode.

## Respond

Answer only from the graph. Use the templates in [outputs.md](references/outputs.md), and check the reply against its examples before sending. Cite page numbers and keep source wording. Do not state legal conclusions (CORE-04).

- **Instruments added, no question asked (T6):** headline counts, promotions from search evidence and the main exceptions. Do not repeat the graph.
- **Instrument summary (T1)** and **conditions list (T2):** one block per instrument, in decision-date order.
- **Specific condition (T3):** quote the condition and its reason. List any approvals or later instruments the graph links to it, without saying whether it is discharged or what its current wording is (COND-03).
- **Family questions:** if the request also asks about the family, branches or missing documents, call @permission-family once the graph is written and give its answer. Otherwise, where cited records remain unresolved, offer it in one line.
- **Every reply** ends with the footer (T4).

## When the user asks to test this skill

Use this section only when the user explicitly asks to test, evaluate or debug the skill. Run the skill on the supplied documents for real. Keep the task output (Part A) separate from a diagnostic report (Part B). Part B covers:
- ambiguous, missing or misleading instructions, and the rule IDs involved;
- instrument layouts or link wording (FAM-08) the workflow did not anticipate;
- which validation checks were actually performed;
- which steps were limited by the environment, such as being unable to save the graph or read page images.

Propose concrete instruction changes, but do not change the skill unless asked.

## Instrument data

The graph envelope and assertion rules are in PKG-02 and PKG-04. Fields with non-obvious placement:
- `permission.data`: `permission_form`, `description_operative`, `components[]` (hybrid), `time_limit_verbatim`, `approved_plans[]` (`number`, `revision`, `title`, `date`, `listed_in`), `incorporated_documents[]`, `informatives[]`, `varied_conditions[]` (s73).
- `condition.data`: `permission_id`, `condition_number` (as printed), `group_heading`, `text_verbatim`, `reason_verbatim`, `trigger_wording`.
- `decision.data` for condition approvals: `condition_refs` (as printed), `submitted_documents[]`, `decision_wording`, `scope`, `partial`, `outstanding`, `continuing_duties`.

Flag codes: `description_changed_under_s73`, `time_limit_extended_under_s73`, `obligation_linkage_not_evidenced`, `unprinted_condition_possible`, `unincorporated_reliance`, `numbering_defect`, `conflicting_values`, `jurisdiction_not_validated`.

## Calibration

This skill is uncalibrated. Fixtures to build first (synthetic or redacted, under `tests/`):
- a scanned decision notice with a stamped issue date and a conditions schedule split across pages;
- a PDF bundling a notice, its approved plans and a condition decision letter;
- an s73 grant restating the full condition set, with a numbering defect;
- an appeal decision allowing an appeal against refusal, with a conditions schedule;
- a partial condition approval citing the pre-s73 reference;
- a permission granted after February 2024 with no printed biodiversity gain condition;
- a notice for a record already in the graph from a search, to test promotion and re-targeting.
