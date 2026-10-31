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
- **SRCH-02** `practice` Classify by issuing body and form title, not by the "CON29" prefix. CON29DW (drainage and water; from the water company) and CON29M (coal mining; Coal Authority-based) are separate searches that often appear in the same pack.
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
| `negative` | "No", "None" |
| `negative_knowledge_limited` | "Not aware", "None recorded" |
| `qualified` | An answer with conditions or caveats |
| `referred_elsewhere` | "See LLC1", "Contact the county council" |
| `see_attachment` | "See attached schedule" |
| `not_answered` | Left blank or declined |
| `not_searched` | The question was not selected or not covered |
| `not_applicable` | Stated as not applicable |
| `illegible` | Cannot be read |
| `other` | Anything else; keep the text |

- **RESP-04** `practice` Record the temporal coverage stated for a response (for example "records since 1974") as part of that response's scope, not only in the general caveats.
- **RESP-05** `model` Entry fields: source entry type; verbatim text; references; authority; labelled dates; source response IDs; linked entity IDs or unresolved references; evidence; review state.
- **RESP-06** `model` A missing schedule, an unreadable page or a failed extraction is never a negative answer (EVID-08).
- **RESP-07** `model` The same record appearing under several questions gets a separate entry for each appearance. Resolve them to one shared entity only on identity evidence: the same authority plus the same reference.
- **RESP-08** `model` Every search statement is `source_reported` until it has been checked against the underlying instrument or register.

## LLC · Local land charges

- **LLC-01** `practice` The LLC1 and CON29 answer different questions. Local land charges are public-law financial charges, restrictions or obligations that bind successive owners and occupiers. CON29 can report matters that are not charges, such as refused or pending applications and highway status. An empty LLC1 result does not establish the absence of permissions, applications or other matters, and a CON29 planning history does not substitute for the LLC1.
- **LLC-02** `model` Model each entry as a `local_land_charge` with: originating authority, registration details, category, burdened land, verbatim wording, and a link to the underlying instrument where established. The registration is not the instrument.
- **LLC-03** `practice` Planning conditions, and often the permissions themselves, are registered as local land charges. The LLC1 therefore frequently duplicates material that also appears in CON29R 1.1. Keep the entries separate and link them under RESP-07.
- **LLC-04** `practice` Council registers use the legacy parts: 1 general financial, 2 specific financial, 3 planning, 4 miscellaneous, 5 fenland ways, 6 land compensation, 7 new towns, 8 civil aviation, 9 opencast coal, 10 listed buildings, 11 light obstruction, 12 drainage schemes. HM Land Registry results use their own categories. Record the category verbatim in either case.

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

- **CON29R-01** `model` For 3.9, record the precise instrument or action, and whether it subsists or the authority has decided to take it. Never reduce it to a generic "planning issue". The instruments include enforcement, stop, listed-building enforcement and breach-of-condition notices; planning contravention notices; repairs notices and building preservation notices; directions restricting permitted development; revocation, modification and discontinuance orders; tree preservation orders; and proceedings to enforce a planning agreement or contribution.
- **CON29R-02** `practice` Section 1.1 is the planning history. Its subparts (b)–(l) are not planning permissions (ROUTE-01); (j)–(l) belong to building control (CUR-05).

## CON29O · Optional enquiries (2016 form)

This is the working baseline; verify it against the form in hand.

`4` road proposals by private bodies · `5` advertisements · `6` completion notices · `7` parks and countryside · `8` pipelines · `9` houses in multiple occupation · `10` noise abatement · `11` urban development areas · `12` enterprise zones, local development orders, BIDs · `13` inner urban improvement areas · `14` simplified planning zones · `15` land maintenance notices · `16` mineral consultation and safeguarding areas · `17` hazardous substance consents · `18` environmental and pollution notices · `19` food safety notices · `20` hedgerow notices · `21` flood defence and land drainage consents · `22` common land, town and village greens.

- **CON29O-01** `model` An unselected optional question is `not_searched`, never `negative`.

## HAND · Planning handoff

For each planning entry, apply the following rules.

- Create an application, and a decision where an outcome is stated (PERM-01). Classify with the alias table (VOCAB-05).
- Create no permission at `search_report_only` (PERM-02).
- Keep the search description as a `description_source` entry (PERM-05).
- Generate candidate family links (REL-02) and turn cited parents into unresolved references (REL-03).
- Carry the search's "as at" date and the response scope onto every entity derived from the search.

## Sources

- [Law Society: CON29 forms and guidance](https://www.lawsociety.org.uk/en/topics/property/con29-forms)
- [HM Land Registry Practice Guide 79: local land charges](https://www.gov.uk/government/publications/local-land-charges-pg79/local-land-charges)
- [HM Land Registry Local Land Charges Programme](https://www.gov.uk/government/publications/hm-land-registry-local-land-charges-programme/local-land-charges-programme)

CON29 forms are published under the publisher's terms. Store code mappings, not the form text.
