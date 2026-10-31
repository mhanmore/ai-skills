---
id: planning.local_authority_searches
kind: resource_bundle
jurisdiction: England
status: draft
requires: [planning.core, planning.permissions]
nodes: [SRCH, RESP, LLC, CON29R, CON29O, HAND]
---

# Local authority searches

Rules for ingesting search reports. A search is a dated report about records; it does not prove the current legal position. Load with [core](core.md); add [permissions](permissions.md) for the planning handoff.

## SRCH · Report and components

| Component | Function | Answered by |
| --- | --- | --- |
| `LLC1` | Official search of the local land charges register | The council or HM Land Registry (CUR-06) |
| `CON29R` | Required enquiries | The council; in two-tier areas the county may answer highways and other matters |
| `CON29O` | Optional enquiries, selected per search | As CON29R |
| Additional | Request-specific questions, plans, schedules, attachments | As stated |

- **SRCH-01** `model` Never merge components. Record the provider, every replying body and the report type: `official`, `personal`, `other` or `unknown`. Take these from the report's statements, not its appearance.
- **SRCH-02** `practice` Classify by issuing body and form title, not by the "CON29" prefix. CON29DW (drainage and water; from the water company) and CON29M (coal mining; Coal Authority-based) are separate searches that often appear in the same pack. Provider packs also carry terms and conditions, search-code statements, complaints procedures and insurance policies, sometimes most of the pages. Record these as `boilerplate` page ranges; do not extract them.
- **SRCH-03** `model` Capture the envelope before reading any entries. That means:
  - report ID, form and version;
  - the searched property wording, plan or boundary, and any mismatch warnings;
  - request, issue, "as at" and question-specific dates (EVID-06);
  - the components included, the optional and additional questions selected, and any exclusions;
  - every document, its page count, and any attachments that are missing or unreadable;
  - caveats, limitations and copy-document directions.
- **SRCH-04** `model` Keep sections in source order. Map them to canonical codes separately. Retain unmapped questions and provider-specific sections: the code tables below are not a discard filter.
- **SRCH-05** `practice` The search plan governs the spatial scope; address similarity does not establish that the land is the same. Planning history can include neighbouring land.

## RESP · Responses and entries

- **RESP-01** `model` Use two linked layers. `responses[]` holds every question and subquestion with its answer, in source order, including negative and qualified answers. `entries[]` holds every listed record, linked to domain nodes only where identity is justified.
- **RESP-02** `model` Response fields: source code and text; canonical code; verbatim answer; `answer_classification`; scope; replying body; entry IDs; evidence.
- **RESP-03** `model` `answer_classification`:

| Value | Typical wording |
| --- | --- |
| `entries_listed` | A list or table of records |
| `negative` | "No", "None", "None revealed" |
| `negative_knowledge_limited` | "Not aware", "None recorded" |
| `not_known` | "Not known" (the replier does not hold the information) |
| `qualified` | An answer with conditions or caveats |
| `referred_elsewhere` | "See LLC1", "Contact the county council" |
| `see_attachment` | "See attached schedule" |
| `not_answered` | Left blank or declined |
| `not_searched` | The question was not selected or not covered |
| `not_applicable` | Stated as not applicable |
| `illegible` | Cannot be read |
| `other` | Anything else; keep the text |

- **RESP-04** `practice` Record the temporal coverage stated for a response as part of that response's scope, not only in the general caveats. Examples: "electronic record of all applications from 1st August 1977"; building regulations "on a rolling 17 year basis"; a cut-off date question. Informatives usually attach to the question or section printed directly above them.
- **RESP-05** `model` Entry fields: source entry type; verbatim text; references; authority; labelled dates; source response IDs; linked entity IDs or unresolved references; evidence; review state.
- **RESP-06** `model` A missing schedule, an unreadable page or a failed extraction is never a negative answer (EVID-08).
- **RESP-07** `model` The same record appearing under several questions gets a separate entry for each appearance. Resolve them to one shared entity only on identity evidence: the same authority plus the same reference.
- **RESP-08** `model` Every search statement is `source_reported` until it has been checked against the underlying instrument or register.

## LLC · Local land charges

- **LLC-01** `practice` The LLC1 and CON29 answer different questions. Local land charges are public-law financial charges, restrictions or obligations that bind successive owners and occupiers. CON29 can report matters that are not charges, such as refused or pending applications and highway status. An empty LLC1 result does not establish the absence of permissions, applications or other matters, and a CON29 planning history does not substitute for the LLC1.
- **LLC-02** `model` Model each entry as a `local_land_charge` with: originating authority, registration details, category, burdened land, verbatim wording, and a link to the underlying instrument where established. The registration is not the instrument.
- **LLC-03** `practice` Planning conditions, and often the permissions themselves, are registered as local land charges. The LLC1 therefore frequently duplicates material that also appears in CON29R 1.1. Keep the entries separate and link them under RESP-07.
- **LLC-04** `practice` There are two result formats; record which one is present.
  - **Council schedule** (unmigrated authorities and older reports). It is organised by legacy parts: 1 general financial, 2 specific financial, 3 planning (often split into (a) and (b)), 4 miscellaneous, 5 fenland ways, 6 land compensation, 7 new towns, 8 civil aviation, 9 opencast coal, 10 listed buildings, 11 light obstruction, 12 drainage schemes. Entries are table rows with columns for description, originating authority, place of inspection and registration date. Rows wrap and continue across pages, and address lines can sit between entries ambiguously, so read the layout, not the linear text. Provider reports may add a summary listing each part as "None revealed" or "Included".
  - **HM Land Registry official search** (migrated authorities). The header certifies the search area, a reference, and the time and date. Below it, each charge is a block separated by a horizontal rule, containing: a map showing the search area and the charge area; `Category` as "Type - Subcategory" (for example "Planning - Conditional planning consent"); `Location`; `Description`; `Law`; `Legal document`; `Originating authority`; `Authority reference`; `Source information`; `Registration date`; `Creation date`; and the `HM Land Registry reference` (LLC-…). There are no parts. A field's value can continue onto the next page.
- **LLC-05** `practice` HM Land Registry data was cleansed at migration, under rules published for each authority ([HMLR search results guidance](https://www.gov.uk/guidance/hm-land-registry-local-land-charges-search-results)). Where a migrated authority's result is being interpreted, record that its customer document exists and cite it. If the document is available to the task, check it for removed record classes and rewritten fields. The cleansing affects interpretation:
  - `Law` may be a normalised provision rather than the instrument's citation; for example, older permissions shown as "TCPA 1990 section 70".
  - Creation dates may be "logical" dates constructed at migration.
  - Addresses may have been replaced.
  - Some authorities removed historic entries, for example planning consents before 1 August 1977.

  Treat these fields as register-supplied (`source_reported` from the register), not as evidence of the instrument. Never infer that a record is absent from its absence in a migrated register.
- **LLC-06** `practice` `Authority reference` can be blank. With no reference, an LLC entry can only be a candidate match to a CON29 entry (REL-02). The charge area can differ from the search area, so record both.
- **LLC-07** `model` Reconcile counts. Certificates state the number of charges or registrations (for example "There is 1 local land charge in your search area", "revealed 15 subsisting registrations"); provider summaries list each part. Record any difference between that figure and the number of entries extracted as a gap.

## CON29R · Required enquiries (2016 form)

This is the working baseline. Before mapping, check the form and version printed on the report, and map on question text as well as the code.

| Code | Subject |
| --- | --- |
| `1.1` | Planning and building decisions and pending applications. Subparts: (a) planning permissions; (b) listed building consents; (c) conservation area consents; (d) certificates of lawfulness, existing; (e) certificates of lawfulness, proposed; (f) certificates of proposed works for listed buildings; (g) heritage partnership agreements; (h) listed building consent orders; (i) local listed building consent orders; (j) building regulations approvals; (k) building regulations completion certificates; (l) competent-person certificates or notices. Each subpart is a separate response, even where the answer is combined. |
| `1.2` | Planning designations and proposals in existing or proposed development plans |
| `2.1` | Status of named roads, footways and footpaths |
| `2.2` | Public rights of way abutting or crossing the property |
| `2.3` | Pending applications to record public rights of way |
| `2.4` | Orders to stop up, divert, alter or create public rights of way |
| `2.5` | Plan of the route |
| `3.1` | Land required for public purposes |
| `3.2` | Land to be acquired for road works |
| `3.3` | Drainage matters (SuDS) |
| `3.4` | Nearby road schemes |
| `3.5` | Nearby railway schemes |
| `3.6` | Traffic schemes |
| `3.7` | Outstanding notices |
| `3.8` | Contravention of building regulations |
| `3.9` | Notices, orders, directions and proceedings under the Planning Acts |
| `3.10` | Community Infrastructure Levy |
| `3.11` | Conservation areas |
| `3.12` | Compulsory purchase |
| `3.13` | Contaminated land |
| `3.14` | Radon |
| `3.15` | Assets of community value |

- **CON29R-01** `model` For 3.9, record the precise instrument or action, and whether it subsists or the authority has decided to take it. Never reduce it to a generic "planning issue". The 2016 subparts are (a) to (n): enforcement notice; stop notice; listed building enforcement notice; breach of condition notice; planning contravention notice; other breach notice; listed building repairs notice; compulsory purchase with a minimum-compensation direction; building preservation notice; direction restricting permitted development; revocation or modification order; discontinuance order; tree preservation order; proceedings to enforce a planning agreement or contribution.
- **CON29R-02** `practice` Section 1.1 is the planning history. Its subparts (b)–(l) are not planning permissions (ROUTE-01); (j)–(l) belong to building control (CUR-05).
- **CON29R-03** `practice` Questions 3.1 to 3.15 are asked "apart from matters entered on the registers of local land charges". A "No" at 3.9(m) or 3.11 therefore says nothing about a tree preservation order or a conservation area registered in the LLC1. Read Part 3 answers together with the LLC result. Never record that combination as a conflict, and never record it as an absence.
- **CON29R-04** `practice` Identify the form version from its header (for example "Con.29 (2016)") or from the question set. Earlier forms number differently: 1.1 has (a)–(h), with building regulations at (f)–(h); 3.3 is drainage agreements; 3.10 is conservation areas, 3.11 compulsory purchase, 3.12 contaminated land and 3.13 radon; there are no CIL or asset-of-community-value questions. Map on question text whenever the version is uncertain, because a code alone is ambiguous: 3.10 means CIL in 2016 but conservation areas earlier.
- **CON29R-05** `practice` Pre-1992 certificates (for example "Established Use Certificate") can appear under 1.1(d) or (e), sometimes misfiled. Record the label verbatim, using `procedure: other`.

## CON29O · Optional enquiries (2016 form)

This is the working baseline; verify it against the form in hand.

`4` road proposals by private bodies · `5` advertisements · `6` completion notices · `7` parks and countryside · `8` pipelines · `9` houses in multiple occupation · `10` noise abatement · `11` urban development areas · `12` enterprise zones, local development orders, BIDs · `13` inner urban improvement areas · `14` simplified planning zones · `15` land maintenance notices · `16` mineral consultation and safeguarding areas · `17` hazardous substance consents · `18` environmental and pollution notices · `19` food safety notices · `20` hedgerow notices · `21` flood defence and land drainage consents · `22` common land, town and village greens.

- **CON29O-01** `model` An unselected optional question is `not_searched`, never `negative`.

## HAND · Planning handoff

For each planning entry, apply the following rules.

- **HAND-01** `model` Create an application, and a decision where an outcome is stated (PERM-01). Classify with the alias table (VOCAB-05). Create no permission or condition at `search_report_only` (PERM-02). Keep the search description as a `description_source` entry (PERM-05). Derived nodes cite the search response or entry as their evidence (PKG-04) rather than restating it.
- **HAND-02** `model` A link the search states (REL-05) is recorded with its cited wording verbatim and `evidence_level: search_report_only`. Because no permission or condition node exists yet:
  - a link stated against a permission targets the cited record's `application` node where it is in the graph, otherwise an `unresolved_reference` (REL-03);
  - a link stated against a condition targets an `unresolved_reference` for the permission and condition number, with `candidate_parent_node_id` where the cited application is in the graph;
  - the source is the decision where an outcome is stated, otherwise the application. A link from a pending, withdrawn or undecided application has `attributes.proposed: true`.

  PKG-06 re-targets these links once the instruments are reviewed.
- **HAND-03** `model` During search ingestion, generate a `possibly_related_to` candidate (REL-02) only from a reference pattern or from wording that suggests a procedural connection. A shared address, inclusion in the same search or the chronology alone is left to family mapping.
- **HAND-04** `model` Carry the search's "as at" date and the response scope onto every entity derived from the search.

## Sources

- [Law Society: CON29 forms and guidance](https://www.lawsociety.org.uk/en/topics/property/con29-forms)
- [HM Land Registry Practice Guide 79: local land charges](https://www.gov.uk/government/publications/local-land-charges-pg79/local-land-charges)
- [HM Land Registry Local Land Charges Programme](https://www.gov.uk/government/publications/hm-land-registry-local-land-charges-programme/local-land-charges-programme)
- [HM Land Registry: local land charges search results](https://www.gov.uk/guidance/hm-land-registry-local-land-charges-search-results). It has an example official search, plus a customer document for each migrated authority. Those documents record data cleansing at migration (removed historic entries, normalised `Law` citations, constructed creation dates, replaced addresses). The list of authorities also shows which councils have migrated (CUR-06).

CON29 forms are published under the publisher's terms. Store code mappings, not the form text.
