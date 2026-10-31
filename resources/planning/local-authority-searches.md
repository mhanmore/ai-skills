---
id: planning.local_authority_searches
type: domain_resource
jurisdiction: England
status: draft
verified: 2026-10-31
---

# Local authority searches

Reference for ingesting search reports without treating them as complete proof of the current legal position. Preserve the report's form/version, selected enquiries, source order, answers, qualifications and effective dates.

## 1. What a search contains

A commonly described full local authority search has distinct components:

| Component | Function |
| --- | --- |
| `LLC1` / official local land charges certificate | Search of the applicable Local Land Charges Register for entries affecting the searched land. Registration responsibility is migrating authority by authority to HM Land Registry. |
| `CON29R` / CON29 required enquiries | Standard enquiries answered from local-authority and related records; includes planning/building control, roads/public rights of way and other matters. |
| `CON29O` | Separately selected optional enquiries. |
| Additional enquiries/attachments | Request-specific questions, plans, schedules and supplied documents. |

Do not merge these components. A personal search, provider report or combined PDF may have different responsibility, assurance, coverage and presentation. Record the provider, replying bodies and report type rather than inferring them from appearance.

## 2. Report envelope and sections

Capture before interpreting entries:

- report/search ID, provider, ordering party if required, report type and form/version;
- searched property wording, plan/boundary, parcels and mismatch warnings;
- relevant authority/authorities and separate replying bodies;
- request, issue, effective/“as at” and question-specific dates;
- included components, selected optional/additional questions and exclusions;
- every source document, page count, attachment and missing/unreadable item;
- qualifications, caveats, limitations and contact/copy-document directions.

Retain source sections in their original order. Map them to canonical topics separately; do not require every provider to use the headings below.

## 3. CON29R standard enquiry subject list

Use form/version plus question text when mapping. The current working baseline is the CON29 Enquiries of Local Authority (2016), questions 1 to 3.15. Preserve subquestions even when the answer is combined.

### 1. Planning and building regulations

| Code | Canonical subject | Material covered |
| --- | --- | --- |
| `1.1` | Planning and building decisions; pending applications | Planning permission; listed-building and conservation-area consents; existing/proposed lawful-development certificates; proposed-works certificate for listed buildings; heritage partnership agreements and listed-building consent orders; building-regulations approvals, completion certificates and competent-person certificates/notices. |
| `1.2` | Planning designations and proposals | Existing/proposed development-plan land-use designations and specific proposals for the property. |

Treat each listed record as an entry. A permission, application, consent, certificate, agreement and building-control record remain different entity types.

### 2. Roads and public rights of way

| Code | Canonical subject |
| --- | --- |
| `2.1` | Status of named roads, footways and footpaths: publicly maintainable, bonded adoption, frontager works or adoption without reclaiming frontager cost. |
| `2.2` | Public rights of way abutting or crossing the property shown on a definitive/revised map. |
| `2.3` | Pending applications to record such public rights of way. |
| `2.4` | Unimplemented/unmapped orders to stop up, divert, alter or create such public rights of way. |
| `2.5` | Plan showing the approximate route where applicable. |

### 3. Other matters

| Code | Canonical subject |
| --- | --- |
| `3.1` | Land required for public purposes. |
| `3.2` | Land to be acquired for road works. |
| `3.3` | Sustainable drainage matters. |
| `3.4` | Nearby road schemes. |
| `3.5` | Nearby railway, tramway, light-railway or monorail schemes. |
| `3.6` | Approved but unimplemented traffic schemes affecting named routes near the property. |
| `3.7` | Outstanding statutory notices: building, environment, health and safety, housing, highways, public health, flood/coastal erosion. |
| `3.8` | Authorised proceedings for contravention of building regulations. |
| `3.9` | Planning-Act notices, orders, directions and proceedings. |
| `3.10` | Community Infrastructure Levy schedules, notices, payments, appeals, liability orders and enforcement. |
| `3.11` | Historic or proposed conservation-area designation matters specified by the form. |
| `3.12` | Enforceable compulsory-purchase order or decision. |
| `3.13` | Contaminated-land notices, register decisions/entries and pre-remediation-notice consultation. |
| `3.14` | Radon Affected Area indication. |
| `3.15` | Asset-of-community-value nomination/listing, review/appeal, restriction and disposal/bidder status. |

For `3.9`, distinguish at least:

- enforcement, stop, listed-building enforcement and breach-of-condition notices;
- planning contravention and other breach-of-planning-control notices;
- listed-building repairs/building-preservation measures;
- directions restricting permitted development;
- revocation/modification and discontinuance orders;
- tree preservation orders;
- proceedings to enforce a planning agreement or planning contribution.

Do not turn a `3.9` response into a generic “planning issue”; retain the precise instrument/action and whether it subsists or the authority has decided to initiate it.

## 4. LLC1 / Local Land Charges explanation

The register and CON29 answer different questions. Local land charges are generally public-law financial charges or restrictions/obligations affecting land and binding successive owners or occupiers. Common register material includes:

- conditions imposed in planning permissions;
- listed buildings and conservation areas;
- tree preservation orders;
- planning and enforcement notices;
- highways agreements and other registrable agreements;
- financial and environmental-health charges or notices.

Model an LLC entry as a source-reported `local_land_charge` with its originating authority, registration/creation details, category, burdened land, source wording and linked instrument/entity where established. Do not treat registration as the underlying permission, agreement or notice; link separate nodes.

An empty LLC result does not establish absence of permissions, applications or all other matters. CON29 can report information not held as a local land charge, including refused or pending applications and highway status. Conversely, a planning-history list is not a substitute for the LLC result.

## 5. Response and entry model

Retain two linked layers:

1. `responses[]`: every question/subquestion and answer in source order, including negative and qualified answers.
2. `entries[]`: every listed record, linked to reusable domain nodes only where identity is justified.

Minimum response fields:

- source and canonical question codes/text;
- verbatim answer and answer classification;
- applicable land/date/scope;
- entry IDs and page/block evidence.

Minimum entry fields:

- source entry type and verbatim text;
- reference(s), authority and explicitly labelled dates;
- source response IDs;
- linked entity IDs or unresolved references;
- exact evidence and review state.

## 6. Interpretation rules

- Distinguish `negative`, `not_answered`, `not_searched`, `not_applicable`, `see_attachment`, `illegible`, `qualified` and `entries_listed`.
- Never convert failed extraction, a missing schedule or an unreadable page into a negative answer.
- Preserve duplicate appearances under separate questions; resolve to a shared entity only with adequate identity evidence.
- Planning history may include grants, refusals, withdrawals, pending applications and records outside the searched land. Do not convert every row into a permission.
- Keep registration, application, validation, resolution, decision and issue dates distinct.
- A search statement is `source_reported` until checked against the underlying instrument or register.
- Record the spatial basis: search plan/boundary governs; address similarity does not establish identical land.
- Retain unmapped questions and provider-specific sections. The canonical subject list is not a discard filter.

## 7. Permission-family handoff

For each planning item, apply [planning.permissions](permissions.md):

- create provisional Application/Decision records where supported;
- create a Permission only at the evidence threshold declared by the consuming contract;
- type section 73, section 96A, reserved-matters and condition decisions separately;
- preserve unresolved parent/condition references;
- mark evidence level `search_report_only` until underlying documents are reviewed.

## 8. Authoritative anchors

- [Law Society: CON29 forms and guidance](https://www.lawsociety.org.uk/en/topics/property/con29-forms)
- [HM Land Registry Practice Guide 79: Local Land Charges](https://www.gov.uk/government/publications/local-land-charges-pg79/local-land-charges)
- [Search for local land charges](https://www.gov.uk/search-local-land-charges)
- [HM Land Registry Local Land Charges Programme](https://www.gov.uk/government/publications/hm-land-registry-local-land-charges-programme/local-land-charges-programme)

CON29 forms and guidance can change and are distributed under publisher terms. Store canonical topic mappings, not an assumed perpetual form text; re-verify the form/version before operational use.
