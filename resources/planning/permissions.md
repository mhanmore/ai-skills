---
id: planning.permissions
type: domain_resource
jurisdiction: England
status: draft
verified: 2026-10-31
---

# Planning permissions

Compact reference for identifying planning permissions and their family records. Preserve source wording and evidence; do not infer legal effect from a portal label, reference suffix or chronology alone.

## 1. Core distinction

- **Application:** procedural request; may be pending, withdrawn or refused.
- **Decision:** determination of an application or later request.
- **Permission:** authorising instrument established by a grant.
- **Family record:** later decision or instrument connected to a permission. Not every family record is itself a permission.

Keep these as separate graph nodes. Expose a family hierarchy for navigation, but retain typed graph relationships because a record may concern several permissions, conditions, phases or matters.

## 2. Permission node

### Required identity

| Field | Rule |
| --- | --- |
| `node_id` | Stable opaque internal ID. |
| `node_type` | `permission`. |
| `schema_ref` | Versioned permission schema. |
| `jurisdiction` | Explicit; this resource covers England. |
| `authority_id` | Link to the deciding authority. |
| `reference` | Authority-scoped reference, preserved verbatim as a string. |
| `granting_decision_id` | Decision that establishes the permission. |
| `decision_date` | Date on the granting instrument; do not substitute validation, committee or portal dates. |
| `description_verbatim` | Operative/source description without rewriting. |
| `site_id` | Link to site node; retain instrument-specific geographic scope. |
| `permission_form` | `full`, `outline`, `hybrid`, `other`, `unknown`. |
| `procedure` | How the permission arose: for example `full_application`, `outline_application`, `section_73`, `appeal`, `other`, `unknown`. |
| `source_document_ids` | Granting notice and any other relied-on instruments. |
| `assertion_ids` | Evidence for material fields. |

Keep `permission_form`, application `procedure`, decision `outcome` and `decision_route` on independent axes. Do not collapse labels such as “outline”, “section 73”, “granted” and “appeal” into one type field. Treat `hybrid` as a source-supported classification and describe its component-level full/outline treatment.

### Common detail

- Alternative and portal references.
- Application, validation, resolution, issue and appeal dates as distinct fields.
- Decision maker and route: authority, appeal, call-in, other or unknown.
- Development components, phases, buildings, units, uses and quantities.
- For each component: full/outline treatment and matters reserved.
- Conditions and reasons; condition number is local to its owning instrument.
- Approved drawings/documents with number, revision, date, scope and approval evidence.
- Time-limit wording and other milestones. Store calculated deadlines only as derived assessments.
- Linked obligations, CIL material, appeals and court decisions.
- Review scope, unresolved gaps and evidence cutoff.

Do not treat issue as proof of implementation, lawfulness, currency, expiry or which parallel permission was implemented.

## 3. Family hierarchy

```text
permission
├── section 73 permission
├── section 96A amendment decision
├── reserved-matters approval
├── condition approval/discharge decision
└── other linked records: appeal, correction, obligation, court decision, enforcement
```

This is a consumption view, not a single-parent ontology. Use the rules below when creating nodes and edges.

### Section 73: vary or remove conditions

- **Node type:** `permission`; procedure `section_73`.
- A grant under section 73 is a new, independent permission alongside the earlier permission; do not overwrite or mutate the earlier node.
- Link with `varies_under_s73` to the identified earlier permission.
- Record affected conditions and exact effect. Restated conditions belong to the new instrument.
- Do not infer that the section 73 permission was implemented or superseded the earlier permission.
- Section 73 operates through conditions, not the operative description of development; do not use it to model a changed description without authoritative evidence explaining the position.

### Section 96A: non-material amendment

- **Node type:** `amendment_decision`; procedure `section_96a`.
- Link with `amends_under_s96a` to the permission or other eligible target identified by the decision.
- The decision concerns the approved non-material change; it is not a reissued permission. Read it with the target instrument.
- Record the exact amendment, affected plans/conditions/components and decision outcome.
- Do not create a new permission node merely because a portal presents a separate application reference.

### Reserved-matters approval (RMA)

- **Node type:** `reserved_matters_decision`.
- Link with `reserved_matters_under` to the outline permission.
- Record each matter addressed: `access`, `appearance`, `landscaping`, `layout`, `scale`, plus source-qualified `other` or `unknown`.
- Record component/phase and whether the submission covers all or only some outstanding matters.
- Approval supplies detail under the outline permission; do not silently label it a standalone planning permission.
- Alternative approvals may exist for the same matter. Dates alone do not establish which scheme was implemented.

### Approval or discharge of condition

- **Node type:** `condition_decision`.
- Link with `approves_details_under_condition` to the permission and stable condition ID, not only the displayed number.
- Record application reference, submitted material, formal decision wording, date, requirements and geographic/operational scope covered.
- Preserve partial approvals, caveats, phased approvals and continuing implementation, retention, monitoring or timing duties.
- Keep four questions separate: source-reported discharge, formal decision, documentary coverage and physical/legal compliance.
- Do not propagate an approval to a parallel permission or similarly numbered condition without evidence.

## 4. Other family records

Add types when evidenced; the list is open. Common relationships include:

| Relationship | Meaning |
| --- | --- |
| `appeal_of` | Appeal targets an application or decision. |
| `corrects` | Formal correction of an identified instrument. |
| `relates_to_obligation` | Link to a planning obligation and, where known, clause/scope. |
| `reported_as_related_to` | Source states a relationship but its legal type is unresolved. |
| `possibly_related_to` | Candidate link requiring review. |

Use an unresolved target object where a source cites a missing permission or condition. Never invent the parent from a reference pattern.

## 5. Family traversal rules

1. Start from an evidenced permission, not every planning-history row.
2. Follow typed edges in both directions; retain branches and overlapping families.
3. Match references within authority and site scope; an address match alone is insufficient.
4. Preserve every instrument and decision as a separate node.
5. Treat a “current consolidated permission” as a dated assessment over a declared path and evidence set.
6. Keep conflicting assertions until reviewed resolution; do not prefer the newest-looking record automatically.

## 6. Minimum extraction from a search report

A search entry may support provisional Application and Decision nodes, but create a Permission node only when the evidence establishes an authorising grant under the applicable contract. Retain:

- authority; reference; source description; source outcome;
- every stated date with its source meaning;
- property/site wording and report scope;
- source question, row, page and report effective date;
- evidence level `search_report_only` until underlying material is reviewed;
- unresolved parent, condition or phase references.

## 7. Authoritative anchors

- [Town and Country Planning Act 1990, section 73](https://www.legislation.gov.uk/ukpga/1990/8/section/73)
- [Town and Country Planning Act 1990, section 96A](https://www.legislation.gov.uk/ukpga/1990/8/section/96A)
- [Planning Practice Guidance: flexible options for planning permissions](https://www.gov.uk/guidance/flexible-options-for-planning-permissions)
- [Planning Practice Guidance: making an application](https://www.gov.uk/guidance/making-an-application)
- [Development Management Procedure (England) Order 2015](https://www.legislation.gov.uk/uksi/2015/595/contents)

These anchors support classification; the instrument and matter-specific evidence control the extracted record. Recheck sources when this resource is operationalised or the law changes.
