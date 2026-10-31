# Planning resources and skills — schema brainstorm

Status: initial design discussion with two draft domain resources; not an implemented schema or operational skill.
Started: 31 October 2026.  
Initial scope: England; jurisdiction must be explicit so other regimes can be added without silently importing English assumptions.

## 1. Aim

Build a reusable library of planning concepts, data contracts and extraction rules. Skills consume those resources rather than each inventing a representation of a permission, condition or search result.

First use case: ingest a local authority search PDF into evidence-traceable JSON; reuse its planning history in permission-family research, condition consolidation and due diligence.

Use an **open-world domain graph with closed-world contracts**. The resource library may grow through new concepts, entity types and relationships without requiring one central ontology to anticipate the whole domain. Skills, however, must exchange data through predictable, versioned and validated interfaces.

Keep four distinct layers:

1. **Open domain graph:** independently maintainable concepts, legal authorities, jurisdictional rules, mappings, examples and typed links between them.
2. **Core graph protocol:** stable envelopes for nodes, relationships, documents, assertions, versions, evidence scope and review state.
3. **Type and skill contracts:** versioned JSON Schemas, controlled vocabularies, examples, validation rules and declared skill input/output requirements.
4. **Skills:** how to obtain, traverse, read, reconcile and validate evidence using those resources and contracts.

The graph is open in what it can represent, but closed at execution boundaries. An unfamiliar node or relationship type may be preserved without interpretation; it must not be silently coerced into a known type. A skill must declare which schema versions and graph elements it understands, what evidence it requires and the exact contract it emits.

A search is a dated report about records. It is not itself a complete permission file or proof of the current legal position.

## 2. Proposed repository layout

Illustrative future layout. The permission and local-authority-search resources now exist as first drafts; the remaining structure is not yet implemented.

- `resources/planning/` — an open graph of Markdown resource nodes with stable IDs, typed frontmatter and explicit links; larger overview resources may remain where they aid human navigation.
- `resources/planning/permissions.md` — permission and related-decision concepts.
- `resources/planning/conditions.md` — condition identity, versions, triggers and coverage.
- `resources/planning/local-authority-searches.md` — search scope, question mapping and interpretation.
- `resources/planning/evidence.md` — source pointers, assertions, uncertainty and review.
- `registries/planning/` — versioned registries for known node types, relationship types, vocabularies and jurisdiction-specific mappings; registries are extensible rather than an assertion that the domain is complete.
- `schemas/planning/core/` — graph-node, relationship, document, assertion, version and review envelopes.
- `schemas/planning/v0.1/` — domain objects and search-package JSON Schemas.
- `contracts/planning/skills/` — per-skill accepted schema versions, required evidence and exact input/output contracts.
- `examples/planning/` — synthetic examples and expected outputs.
- `tests/planning/` — validation and extraction fixtures.
- A later search-ingestion skill referencing those resources.
- Existing `planning-permission-condition-consolidation/` continues to consume its current contract until an explicit migration is agreed.

Do not put real client search reports, addresses or extracted matter JSON into this public repository. Use synthetic or suitably redacted fixtures.

## 3. Core modelling decisions

### Open graph, closed interfaces

Treat the proposed entities as the first supported node types, not as the permanent outer boundary of the planning domain. Later resources may introduce types such as enforcement notices, listed-building consents or jurisdiction-specific instruments without redesigning the core graph protocol.

Resource nodes should be independently maintainable and linkable, in the style of an open knowledge graph. Each should have a stable ID and sufficient metadata to establish its type, jurisdiction, status, sources, review date and typed relationships. Prose can carry nuance; frontmatter and registries carry machine-readable identity and semantics.

Operational data uses a stable envelope even when its type is extensible. At minimum:

- A node has a stable `node_id`, `node_type`, `schema_ref`, jurisdiction where applicable, typed data and assertion links.
- A relationship has its own ID, registered or explicitly unrecognised type, source, target, type-specific attributes, evidence and review state.
- A document, assertion and assessment uses the shared evidence/version protocol below.
- Unknown extensions are preserved and reported. They do not validate as known types and do not acquire legal meaning merely because an AI can infer a plausible interpretation.

Closed contracts apply whenever a skill consumes or emits a package. Each skill declares:

- supported core, node-type and vocabulary versions;
- node and relationship types it interprets, preserves without interpretation or rejects;
- required fields, evidence thresholds and permitted unresolved states;
- graph traversal and coverage expectations;
- exact output schema and validation level achieved.

This allows exploratory research and intermediate extraction to remain incomplete and extensible while making inter-skill handoffs reproducible and auditable.

### Separate application, decision and permission

An application is a procedural record; it can be pending, withdrawn or refused. A decision records an outcome. A permission is an authorising instrument where the evidence establishes one. Avoid turning every planning-history row into a permission.

Proposed entities:

| Entity | Purpose / key fields |
| --- | --- |
| Site | Stable internal ID, source address, normalised address, postcode, UPRNs as strings, parcels, geometry, coordinate reference system, location evidence |
| Authority | Internal ID, official name, external identifiers, role, aliases and relevant dates |
| Application | Authority-scoped reference, verbatim description, procedure, application/validation dates, applicant/agent if needed, portal link |
| Decision | Application link, decision-maker, date, verbatim outcome, normalised outcome, decision notice, appeal/court links |
| Permission | Granting-decision link, description, form, geographic scope, conditions, approved documents, related instruments |
| Relationship | Typed, directed link between records, targets, source, certainty/review state |
| Condition | Stable ID, instrument-local number, verbatim wording/reason, versions, requirements, triggers and scope |
| Document | Identity, type, URI/file identifier, checksum, date, readability and extraction details |
| Assertion | Field-specific observation or interpretation with exact provenance |
| Search report | Report identity, scope, effective date(s), question responses, entries, limitations and reconciliation |
| Assessment | Derived view for a defined evidence set, date, purpose and review scope |

Use stable opaque IDs. A portal reference is an external identifier, not a globally unique primary key. Preserve punctuation, leading zeroes and alternative references.

### Classify on independent axes

Do not use a single enum combining "outline", "s73", "approved" and "appeal".

- `procedure`: full application, outline application, reserved matters, condition approval, s73, s96a, prior approval, lawful development certificate, other, unknown.
- `permission_form`: full, outline, hybrid, other, unknown.
- `outcome`: granted, refused, withdrawn, pending, split, other, unknown; preserve exact source wording.
- `decision_route`: authority, appeal, call-in, other, unknown.
- `jurisdiction` and `statutory_basis`: separate fields; preserve cited provision and record the source.
- `development_components[]`: geographic/component breakdown, with full/outline treatment and matters reserved for each component.

"Detailed" may be authority terminology: retain it as a source label and map it only where justified. Hybrid needs component-level information, not just a label. Model reserved matters individually: access, appearance, landscaping, layout and scale; allow unknown/other and source-specific qualifications.

These are candidate vocabularies, to be tested against documents before being frozen.

## 4. Permission data points

### Identity and site

- Internal IDs; authority and reference(s); jurisdiction; application/decision/permission links.
- Address verbatim and structured address separately; location description for land without a postal address.
- UPRNs, title numbers and parcels where actually supplied; no assumed equivalence between them.
- Site boundary, phase/building/unit scope and geometry provenance. An address match alone does not establish identical site extent.

### Development and procedure

- Description of development verbatim; optional short summary.
- Permission form; application procedure; statutory basis; reserved/approved matters.
- Components/phases; quantities and uses where useful, preserving units, qualifiers and source.
- Application, validation, resolution, decision, issue and appeal dates as distinct fields.
- Decision outcome and decision-maker; portal/notice references.
- Time-limit and other milestone wording, with calculated deadlines stored only as separately reasoned assessments.
- Approved drawings/documents: title, number, revision, date, approval evidence and applicable component.
- Conditions and reasons; informatives separately.
- Related s106 instruments, CIL documents, appeals and court decisions as links, not embedded legal conclusions.
- Implementation, expiry, lawfulness and operative-permission assessments only where separately evidenced and scoped.

A committee resolution is not silently treated as an issued decision. A time limit is not silently treated as proof of expiry.

## 5. Families: typed relationships rather than a simple tree

Expose a convenient family view, but retain a graph as the underlying representation. One decision can target several conditions, matters or phases; a site can have overlapping permission families.

Candidate relationship types:

| Relationship | Required detail |
| --- | --- |
| `reserved_matters_under` | Outline permission, matters and component/phase concerned |
| `approves_details_under_condition` | Specific permission and condition ID(s), scope and decision caveats |
| `varies_under_s73` | Source permission, resulting instrument, affected conditions and evidence |
| `amends_under_s96a` | Target permission/instrument, exact amendment and scope |
| `appeal_of` | Application/decision and appeal reference |
| `corrects` | Instrument corrected and correction evidence |
| `relates_to_obligation` | Instrument and relevant clause/scope where known |
| `reported_as_related_to` | Source-stated relationship whose precise type is unresolved |
| `possibly_related_to` | Candidate link for review, not an established family edge |

Modelling guardrails to verify in the later legal resource:

- Preserve s73 resulting instruments as separate records; do not overwrite the earlier permission.
- Record s96a amendment effects against the identified target; do not assume every amendment is a new permission.
- Reserved-matters and condition decisions need their own records, without automatically being labelled standalone permissions.
- Never infer parentage from reference suffixes alone.
- Do not automatically propagate a condition approval or amendment across parallel permissions.
- A later date does not establish supersession, implementation or which permission is operative.
- Condition numbers are local to an instrument and may change. Cross-instrument mappings need evidence.
- Support missing targets using an unresolved reference object, without inventing a known parent.

Any "current consolidated permission" should be a derived view selecting a defined instrument/path and evidence set. Preserve alternative branches.

## 6. Conditions and existing consolidation work

Reuse the existing consolidation specification's strengths: full wording, amendment history, page-specific evidence, triggers, categories, hierarchical scope, formal decision versus documentary coverage, and explicit review limitations.

Extensions to consider:

- `condition_id` independent of displayed `condition_number`.
- Version IDs and explicit instrument ownership; mappings for renumbering, deletion, replacement and splitting.
- Reasons versioned alongside wording.
- Requirement-level records: submission, approval, implementation, retention, monitoring, timing and geographic scope.
- Approval decisions linked to requirements and scope, including partial approvals and caveats.
- Separate source-reported discharge, formal decision, documentary coverage and evidence of physical compliance.
- Keep the existing `discharge_status` vocabulary as a compatibility concern; do not silently redefine it.
- Treat current-text consolidation and documentary coverage as derived assessments with their own evidence cutoff.

The existing specification should not be rewritten until a migration and representative fixtures are agreed.

## 7. Shared evidence and uncertainty contract

Every material extracted value must resolve to evidence. Avoid an unmanageable provenance wrapper around every scalar: use simple domain objects plus an assertion registry keyed by JSON Pointer.

Candidate assertion fields:

- `assertion_id`, `subject_id`, `field_path`, `value`.
- `assertion_kind`: source_reported, normalised, inferred, reviewed_conclusion.
- `sources[]`: document ID, 1-based PDF page, printed page label, question/section, table/row/block locator, short supporting text and optional bounding box.
- `normalisation_rule` where applicable; original value always recoverable.
- `review_status`: unreviewed, verified_against_source, needs_review.
- `value_state`: present, not_stated, illegible, not_applicable, conflicting.
- `derived_from_assertion_ids[]`, reviewer/date and rationale for derived findings.

Do not give an uncalibrated numeric confidence score the appearance of legal certainty. OCR confidence can be retained as technical metadata, separately from substantive review.

Dates: ISO 8601 when exact; preserve original text and precision for month-only, approximate or ambiguous dates. Never invent a day. Use UTC timestamps for processing events and date-only values for source dates where appropriate.

Null needs a stated reason. Empty arrays mean no extracted items within the declared scope, not that no such records exist. Conflicting assertions coexist until a documented resolution; never silently prefer the newest-looking source.

## 8. Local authority search resource

### Report envelope

Capture:

- Search ID, provider, authority, report/reference number and report type (official, personal, other, unknown).
- Request, issue and source "as at" dates separately; question-specific dates where provided.
- Property address, search plan/boundary, identifiers and mismatch warnings.
- Included components: LLC1, CON29, optional enquiries, attachments and other sections, using source labels.
- Form/version if known; jurisdiction; selected questions, exclusions and qualifications.
- Documents/pages supplied and reviewed; missing schedules, unreadable pages and report caveats.

### Questions, responses and entries

Use two linked representations:

1. `responses[]` retains every question/subquestion and answer in source order, including negative and qualified answers.
2. `entries[]` retains each listed record and links to reusable domain objects such as an application, permission, local land charge, enforcement record or designation.

Response fields: source question code/text, canonical topic if mapped, verbatim answer, answer classification, applicable scope/date, entry IDs and evidence.

Entry fields: entry type, verbatim text, reference(s), registration/decision dates with distinct meanings, source response IDs, authority, related entity IDs and evidence.

Important distinctions:

- "No", "not answered", "not searched", "see attachment" and "illegible" are different states.
- A planning application, local land charge entry, planning obligation, enforcement notice and policy designation remain different objects even when cross-referenced.
- Planning history can list refusals, withdrawn applications and neighbouring land.
- Report statements remain source-reported observations until checked against underlying documents.
- Preserve duplicate appearances in separate responses; link them to a shared entity only where justified.
- Do not hard-code a single provider's question numbering. Map by form/version plus content, retaining unmapped questions.
- Keep non-planning responses in the report representation even if the first downstream skill uses only planning information.

The reusable package should allow later enrichment from portal documents without overwriting what the original search said.

## 9. Proposed PDF-to-JSON ingestion workflow

1. Inventory documents and page counts; identify report boundaries, plans, schedules and attachments.
2. Assess text-layer quality page by page; OCR where needed. Preserve the original PDF identity/checksum and extraction method/version.
3. Read layout, not just linearised text. Inspect tables, continuation pages, footnotes, ticks and merged cells visually where needed.
4. Extract question responses and rows verbatim before normalising or resolving entities.
5. Reconcile page/section/question/row coverage. Retain unclassified material and missing attachments as gaps.
6. Map entries to domain objects and typed links. Record unresolved references and tentative matches explicitly.
7. Validate the structural schema, references, enumerations, dates, evidence pointers and coverage counts.
8. Review material fields against the page image: address/site scope, references, description, dates, decision, negations and relationship targets.
9. Emit canonical JSON plus a short exception/review report and extraction manifest.
10. Downstream skills select an evidence scope; they do not treat successful extraction as legal verification.

Never let a failed or incomplete extraction produce a clean-looking "no entries" result. Reprocessing should create a traceable extraction revision; human corrections must survive as explicit reviewed assertions.

## 10. Illustrative JSON shape

Synthetic, deliberately incomplete example. This is valid JSON but is not yet a formal JSON Schema or a complete permission record.

```json
{
  "schema_version": "0.1-draft",
  "package_id": "pkg-example",
  "jurisdiction": "England",
  "documents": [
    {
      "document_id": "doc-search",
      "document_type": "local_authority_search",
      "file_name": "synthetic-search.pdf",
      "page_count": 12
    }
  ],
  "nodes": [
    {
      "node_id": "search-1",
      "node_type": "search_report",
      "schema_ref": "planning/search-report/0.1-draft",
      "jurisdiction": "England",
      "data": {
        "document_ids": ["doc-search"],
        "report_type": "unknown",
        "as_of_date": null,
        "responses": [
          {
            "response_id": "response-1",
            "source_question_code": "EXAMPLE-PLANNING",
            "answer_text": "Planning history: EX/2024/001 — permission granted.",
            "answer_classification": "entries_listed",
            "entry_ids": ["entry-1"]
          }
        ],
        "entries": [
          {
            "entry_id": "entry-1",
            "entry_type": "planning_history",
            "source_response_ids": ["response-1"],
            "entity_ids": ["application-1"]
          }
        ]
      },
      "assertion_ids": []
    },
    {
      "node_id": "application-1",
      "node_type": "application",
      "schema_ref": "planning/application/0.1-draft",
      "jurisdiction": "England",
      "data": {
        "authority_id": "authority-example",
        "reference": "EX/2024/001",
        "procedure": "unknown"
      },
      "assertion_ids": []
    },
    {
      "node_id": "authority-example",
      "node_type": "authority",
      "schema_ref": "planning/authority/0.1-draft",
      "jurisdiction": "England",
      "data": {
        "name": "Example Council"
      },
      "assertion_ids": []
    },
    {
      "node_id": "decision-1",
      "node_type": "decision",
      "schema_ref": "planning/decision/0.1-draft",
      "jurisdiction": "England",
      "data": {
        "application_id": "application-1",
        "outcome": "granted",
        "decision_date": null,
        "evidence_level": "search_report_only"
      },
      "assertion_ids": ["assertion-1"]
    }
  ],
  "relationships": [],
  "assertions": [
    {
      "assertion_id": "assertion-1",
      "subject_id": "decision-1",
      "field_path": "/nodes/3/data/outcome",
      "value": "granted",
      "assertion_kind": "normalised",
      "normalisation_rule": "Map explicit source phrase permission granted to granted.",
      "review_status": "unreviewed",
      "value_state": "present",
      "sources": [
        {
          "document_id": "doc-search",
          "pdf_page": 5,
          "section": "Planning history",
          "row": 1,
          "supporting_text": "EX/2024/001 — permission granted."
        }
      ]
    }
  ],
  "review_scope": {
    "underlying_decision_notices_reviewed": false,
    "independent_family_search_performed": false
  },
  "unresolved_gaps": [
    {
      "subject_id": "search-1",
      "field_path": "/nodes/0/data/as_of_date",
      "value_state": "not_stated",
      "message": "Effective date not established in this illustrative extract."
    },
    {
      "subject_id": "decision-1",
      "field_path": "/nodes/3/data/decision_date",
      "value_state": "not_stated",
      "message": "Decision notice and decision date not available."
    }
  ]
}
```

In a complete output, the same evidence contract must also cover the reference, authority and response text. A search-only provisional grant can later resolve to a permission record when sufficient evidence is available; record the resolution history.

JSON Pointers describe positions in a particular package revision. Stable subject IDs support cross-revision identity; regenerate and validate pointers when serialising a new revision.

## 11. Validation and representative fixtures

Structural checks:

- Unique IDs; all links resolve or explicitly identify an unresolved external target.
- Authority-scoped reference matching; no accidental reference-only joins.
- Valid source pages and document IDs; all material facts have assertions.
- Response/row counts reconcile to reviewed sections; every omitted section has a reason.
- No missing decision date fabricated from an application or registration date.
- Inferences are labelled and traceable; unresolved conflicts remain visible.
- Graph checks distinguish prohibited self-links from legitimate cross-links; do not demand a single-parent tree.
- Extracted text is retained alongside normalised fields.
- Schema version, extraction revision, tool versions and evidence cutoff are recorded.

Validation should report three distinct levels rather than a single pass/fail result:

1. **Graph-valid:** core envelopes, IDs, links, versions and provenance are structurally sound; unrecognised extensions are preserved and identified.
2. **Type-valid:** each recognised node and relationship conforms to its declared type schema and registered vocabulary version.
3. **Skill-ready:** the graph meets a named skill's required evidence, coverage, review and input-contract rules.

A package may be graph-valid while incomplete or not ready for a particular skill. Validation must never turn open-world absence into a closed-world conclusion such as “no relevant records exist.” Published assessments and inter-skill handoffs must state the highest validation level achieved and the applicable skill contract, if any.

Fixture shortlist:

- Clean text PDF and scanned PDF; mixed text/OCR pages.
- Multi-page planning history with repeated headers and split rows.
- A negative answer, an omitted question and a missing attachment.
- Hybrid permission; phased outline with several reserved-matters decisions.
- s73 branch and s96a amendment; similarly numbered conditions on different instruments.
- Partial condition approval with continuing implementation obligations.
- Same reference at different authorities; changed address; overlapping site boundaries.
- Refusal followed by appeal; duplicate history entries; contradictory dates.
- Search-only history versus underlying decision notice discrepancies.

Start with synthetic fixtures; later test privately against representative real reports.

## 12. Decisions to settle next

1. **Consumer needs:** which first outputs matter most — search summary, chronology, condition tracker or due-diligence issues list?
2. **Granularity:** how much of a search needs structured domain modelling in v0.1 beyond preserved responses?
3. **Evidence storage:** document-relative pointers only, or also immutable cropped page/block references?
4. **Fact model:** assertion registry as above, or field wrappers for a small set of critical values?
5. **Permission creation threshold:** allow provisional search-only permission objects, or require a granting instrument? The example takes the more cautious second approach.
6. **Compatibility:** extend the existing consolidation contract or introduce an adapter to a shared domain model?
7. **Identity resolution:** which matches can be automatic and which require review?
8. **Review policy:** which fields require visual verification before marking an extraction reviewed?
9. **Versioning:** separate domain schema version, extraction run revision and later enrichment history.
10. **Legal resources:** establish authoritative sources for procedure/effect rules before making the resource operational; record jurisdiction, source date and review date.
11. **Extension governance:** define how a provisional node, relationship or vocabulary term becomes registered, and how aliases or deprecated terms remain readable.
12. **Skill compatibility:** decide whether compatibility is declared by exact schema versions, version ranges or tested conformance profiles.

## 13. Suggested first increment

Agree the minimal core graph protocol for node identity, Relationship, Document, Assertion, versioning and review state. Register Authority, Site, Application, Decision, Permission and Condition as the first supported node types, without treating that list as closed. Write the permission resource first, then the search resource that consumes it. Add one synthetic search package, its type schemas and a named skill-readiness profile. Only then draft the ingestion skill and connect the existing condition-consolidation skill through an explicit adapter.

Keep extraction, entity resolution and substantive assessment independently reviewable. Permit the graph to grow through explicit extensions, but require validated contracts at every skill boundary. That combination is what makes one JSON package both reusable across several planning skills and safe to interpret consistently.
