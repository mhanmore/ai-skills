---
id: planning.core
kind: resource_bundle
jurisdiction: England
status: draft
nodes: [CORE, VOCAB, REL, EVID, PKG, CUR]
---

# Planning core

Shared stance, vocabularies, relationships and evidence rules. Load with the topic bundle the task needs: [permissions](permissions.md), [local authority searches](local-authority-searches.md).

Rule tags: `law` states a legal rule (source cited); `practice` describes how documents and systems usually behave; `model` is a representation choice of this library; `currency` is a recent change to check before relying on it. Legal rules produce review flags; they never override what a document says.

## CORE · Scope and stance

- **CORE-01** `model` Scope is England. Set `jurisdiction` from the authority, not the form. CON29 and LLC forms are shared with Wales: a Welsh search may be extracted with the search bundle, but set `jurisdiction: Wales` and flag that permission rules here are not validated for Wales. Scotland and Northern Ireland use different systems; do not apply these resources.
- **CORE-02** `model` Documents first. Record what documents say, including content that looks irregular, unlawful or inconsistent. Legal knowledge here generates flags; it never corrects, discards or filters source material.
- **CORE-03** `model` Work within the task's evidence set. Do not go looking for other sources of authorisation (permitted development, orders, immunity) unless the task asks; record and flag them where a document refers to them.
- **CORE-04** `model` Assist, do not determine. Extract, link and flag. Implementation, lawfulness, expiry and which permission is operative are reviewer questions unless a skill contract asks for a scoped, dated assessment.
- **CORE-05** `model` An empty list means nothing was extracted within the declared scope, not that nothing exists.

## VOCAB · Node types and classification

| Node type | Meaning |
| --- | --- |
| `authority` | A public body, e.g. LPA, county council, Planning Inspectorate, Secretary of State, Mayor of London. |
| `application` | Procedural request. Has `procedure` and `status`. |
| `decision` | Any determination: grant or refusal of permission, reserved-matters approval, condition approval, s96A amendment, certificate, prior approval, appeal decision. |
| `permission` | A planning permission granted by a decision. |
| `condition` | A condition owned by one permission. |
| `document` | A source file, with identity and checksum. |
| `search_report`, `local_land_charge` | See the search bundle. |
| `unresolved_reference` | A record cited by a source but absent from the evidence set. |

The list is open: preserve an unfamiliar type without interpreting it; never coerce it into a known type.

- **VOCAB-01** `model` Reserved-matters approvals, condition approvals, s96A amendments and certificates are `decision` nodes, distinguished by their application's `procedure` and by typed relationships. Only a grant of planning permission creates a `permission` node.
- **VOCAB-02** `model` Each axis lives on one node type:

| Axis | Node | Values |
| --- | --- | --- |
| `procedure` | application | `full`, `householder`, `outline`, `hybrid`, `reserved_matters`, `s73`, `s73a`, `s96a`, `condition_approval`, `prior_approval`, `ldc_existing`, `ldc_proposed`, `listed_building_consent`, `conservation_area_consent`, `advertisement_consent`, `tree_works`, `permission_in_principle`, `technical_details_consent`, `other`, `unknown` |
| `status` | application | `pending`, `decided`, `withdrawn`, `invalid`, `other`, `unknown` |
| `outcome` | decision | `granted`, `refused`, `split`, `not_required`, `declined_to_determine`, `other`, `unknown` |
| `decision_route` | decision | `lpa`, `appeal`, `call_in`, `other`, `unknown` |
| `decision_maker_id` | decision | Link to the authority that decided. |
| `permission_form` | permission | `full`, `outline`, `hybrid`, `other`, `unknown` |

- **VOCAB-03** `model` `procedure` and `permission_form` overlap by design: procedure describes the request; form describes the grant. A s73 grant can be outline in form.
- **VOCAB-04** `model` Keep the source label verbatim beside every normalised value. Map it only where the alias table or the document's wording supports the mapping; otherwise use `other` and keep the label.
- **VOCAB-05** `practice` Reference codes and labels vary by authority. An alias is evidence for a procedure, not proof of it: record it as `assertion_kind: normalised` with the alias as the rule.

| Source label (examples) | `procedure` |
| --- | --- |
| FUL, FULL, "full", "detailed" (check the authority's usage) | `full` |
| HH, HOU, "householder" | `householder` |
| OUT, OUTL, "outline" | `outline` |
| "hybrid", "part full part outline" | `hybrid` |
| RM, REM, RES, ARM, "reserved matters" | `reserved_matters` |
| VAR, VOC, ROC, MMA, "variation/removal of condition", "minor material amendment" | `s73` |
| "retrospective", "regularisation" (may be s73 retrospective: check) | `s73a` |
| NMA, AMD, "non-material amendment" | `s96a` |
| DOC, DIS, DISC, COND, AOD, "approval/discharge of condition/details" | `condition_approval` |
| PA, PN, PRIOR, GPDO, "prior approval/notification" | `prior_approval` |
| CLEUD, LDCE, LUE, "certificate of lawfulness, existing" | `ldc_existing` |
| CLOPUD, LDCP, LUP, "certificate of lawfulness, proposed" | `ldc_proposed` |
| LBC, LBA | `listed_building_consent` |
| CAC (pre-October 2013) | `conservation_area_consent` |
| ADV | `advertisement_consent` |
| TPO, TCA, TWA, "works to trees" | `tree_works` |
| PIP / TDC | `permission_in_principle` / `technical_details_consent` |

| Outcome label (examples) | `outcome` |
| --- | --- |
| "Approved", "Permitted", "Granted", "Conditional planning permission/consent", "Consent given" | `granted` |
| "Refused", "Rejected" | `refused` |
| "Prior approval not required" | `not_required` |
| "Part approved/part refused", "split decision" | `split` |

Building-control records (approvals, completion and competent-person certificates) belong to a separate regime: keep them as source entries, not planning applications.

## REL · Relationships

| Type | Direction and detail |
| --- | --- |
| `determines` | decision → application |
| `grants` | decision → permission |
| `reserved_matters_under` | decision → outline permission; matters and phase or component |
| `approves_details_under_condition` | decision → condition(s); scope, partial approval and caveats |
| `varies_under_s73` | new permission → earlier permission; conditions affected |
| `amends_under_s96a` | decision → target permission or approval; changes made |
| `appeal_of` | appeal decision → application or decision |
| `corrects` | correction → instrument corrected |
| `relates_to_obligation` | instrument → obligation; clause or scope where known |
| `reported_as_related_to` | A source states a link whose type is unresolved |
| `possibly_related_to` | Agent-generated candidate link |

- **REL-01** `model` A confirmed edge needs a document that states the link (for example, a decision citing the parent reference or the condition number).
- **REL-02** `model` Actively generate `possibly_related_to` candidates from reference patterns (`/RM`, `/NMA`, `/DIS`, a shared stem), a shared address or the chronology. Mark each `assertion_kind: inferred` with its basis. Never promote a candidate without document evidence.
- **REL-03** `model` A cited record that is missing becomes an `unresolved_reference`, never a guessed parent.
- **REL-04** `model` The family is a graph, not a tree. Multiple parents, branches and overlapping families are normal; keep them all.
- **REL-05** `model` A typed link stated by any source counts as stated: for example, a search description "Variation of condition 2 of 19/01234/FUL" or a decision notice "pursuant to condition 5 of …". Record the typed relationship with an `evidence_level` attribute taken from its source. A link stated only by a search stays `search_report_only` until an instrument confirms it.

## EVID · Evidence and uncertainty

- **EVID-01** `model` Every material value traces to an assertion: document ID, 1-based PDF page, printed page label, section or question, row or block, and a short supporting quote.
- **EVID-02** `model` `assertion_kind`: `source_reported`, `normalised` (original value and rule kept), `inferred` (basis stated), `reviewed_conclusion`.
- **EVID-03** `model` `value_state`: `present`, `not_stated`, `illegible`, `not_applicable`, `conflicting`. Every null carries a `value_state`.
- **EVID-04** `model` `evidence_level` on applications, decisions and permissions: `search_report_only`, `portal_record` or `instrument_reviewed`.
- **EVID-05** `model` Conflicting assertions coexist until a reviewed resolution. Never prefer the newest-looking source.
- **EVID-06** `model` Record each date with its source label. Use ISO 8601 only for a date stated in full, and keep the original text and precision. Never invent a day, and never derive one date from another (for example, an issue date from a decision or validation date). Numeric source dates are UK day-first (`06/07/2012` is 6 July 2012). Differently labelled dates for the same record (for example an LLC registration date and a CON29 "date decision issued") are not a conflict; keep both.
- **EVID-07** `model` No numeric confidence scores. OCR confidence is technical metadata only.
- **EVID-08** `model` A failed or incomplete extraction never produces a clean negative; record the gap.

## PKG · Matter graph

- **PKG-01** `model` Keep one graph per matter at `matters/<matter_id>/graph.json`, with source files in `matters/<matter_id>/sources/` and skill reports alongside the graph. `matters/` is gitignored. Skills add to the graph; no skill owns it. A matter is scoped to the workspace, and usually there is one per workspace. Use the user's name for it if given; otherwise use `default`. If the workspace holds exactly one matter, use it without asking; ask only when there are several.
- **PKG-02** `model` Envelope. Keys are fixed: use `[]` or a `value_state` rather than omitting a key.

```json
{
  "schema_version": "planning.matter-graph/0.1-draft",
  "matter_id": "", "jurisdiction": "England", "revision": 0,
  "history": [{"revision": 0, "utc": "", "skill": "", "documents_added": [], "summary": ""}],
  "documents": [{"document_id": "", "file_name": "", "sha256": "", "page_count": 0, "document_kind": ""}],
  "nodes": [{"node_id": "", "node_type": "", "schema_ref": "", "jurisdiction": "", "data": {}, "assertion_ids": []}],
  "relationships": [{"relationship_id": "", "type": "", "source_id": "", "target_id": "", "attributes": {"evidence_level": ""}, "assertion_ids": []}],
  "assertions": [],
  "gaps": [{"subject_id": "", "field_path": "", "value_state": "", "message": ""}],
  "flags": [{"code": "", "subject_ids": [], "rule": "", "message": ""}],
  "manifest": {"pages": [{"document_id": "", "page": 0, "component_id": "", "method": "text|visual|ocr"}],
               "counts": {}, "tools": [], "rules_version": "", "validation_level": ""}
}
```

- **PKG-03** `model` Append only. Before each write, copy the current graph to `graph.r<revision>.json`, increment `revision` and append a `history` entry. Never edit or delete an assertion. A correction is a new assertion that names the one it supersedes (`supersedes_assertion_id`).
- **PKG-04** `model` Assertions address a `subject_id` plus a node-relative `field_path` (for example `/data/outcome`), never a pointer into the whole package, so they survive revisions. One `source_reported` assertion can cover a whole block (a response, an entry or an instrument section), with its page, locator and a quote of at most 200 characters. Add field-level assertions only for `normalised` and `inferred` values.
- **PKG-05** `model` Merge on identity. A node from a new source merges into an existing node of the same type with the same authority and reference (PERM-04); otherwise create a new node with a candidate link. A merge adds assertions and never replaces values. The same label with different values becomes `conflicting` (EVID-05); different labels, such as a truncated search description against the operative description, do not conflict.
- **PKG-06** `model` Promotion. Reviewing an instrument raises the `evidence_level` of the nodes and links it evidences. A reviewed grant of planning permission creates the `permission` node (PERM-02), linked from its decision by `grants`. Earlier search-reported assertions remain.
- **PKG-07** `model` Documents are deduplicated by `sha256`. A document already in the graph is not re-extracted unless `rules_version` has changed or the user asks.
- **PKG-08** `model` Derived views (a family tree, a planning-history table, a condition register) are written beside the graph, stamped with the graph revision they were built from. They are never stored as graph facts.

## CUR · Changes models often miss

Each item was correct when written. Check commencement and transitional rules before relying on it. Use these items to flag, not to conclude.

- **CUR-01** `currency` Use classes: from 1 September 2020 (SI 2020/757), Classes E and F replaced A1–A3, B1, D1 and D2; A4 and A5 became sui generis. Older instruments use the old classes: record them verbatim and never translate silently.
- **CUR-02** `currency` Biodiversity net gain: most permissions on applications made from 12 February 2024 (major) or 2 April 2024 (other) are subject to the statutory biodiversity gain condition (TCPA 1990 Sch 7A). That condition is deemed, not usually printed as a numbered condition, and exemptions apply. Flag that it may apply; never infer absence from the notice.
- **CUR-03** `currency` Immunity from enforcement: LURA 2023 set a 10-year period for all breaches in England from 25 April 2024, with transitional rules. The 4-year rules survive only under those transitional rules.
- **CUR-04** `currency` LURA 2023 inserted s73B; do not assume it is in force.
- **CUR-05** `currency` Building control: from 6 April 2024, registered building control approvers replaced approved inspectors, and the Building Safety Regulator handles higher-risk buildings. Sources may use either terminology.
- **CUR-06** `currency` Local land charges registers are migrating to HM Land Registry authority by authority. An LLC1 result and the CON29 replies may come from different bodies.
- **CUR-07** `currency` Local government reorganisation creates successor authorities. Record the authority named on the instrument; any mapping to a successor is a separate assertion.
