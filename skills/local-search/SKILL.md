---
name: local-search
description: Read a local authority search for land in England: an LLC1 and CON29 pack, council-issued, from HM Land Registry or a provider's personal search. It builds an evidence-traceable graph package, then answers the user's request from it (a planning history, a summary, a question about an answer or entry, an export). Use whenever a local search or local land charges search is mentioned.
status: draft
---

# Local search

Typical prompts: "read the /local-search and summarise the planning history", "what does the /local-search say about tree preservation orders?", "build the /local-search package for later".

Every request goes through the same two stages:
1. **Add the search to the matter graph, or reuse it if already there** (PKG-01). The search contributes every question and answer, every listed record, and provisional planning nodes, each traced to page-level evidence.
2. **Answer the request from the graph.**

This skill reports what the search says. It does not assess legal effect, implementation or risk.

## Locate the input

1. Use files attached to the conversation or paths named in the prompt.
2. Otherwise, look in `matters/<matter_id>/sources/` and the working directory. If more than one file could be the search, ask which.
3. Choose the matter (PKG-01): the one the user names, or the only one in the workspace; otherwise create `matters/default/`.
4. Checksum each file. If the graph already holds those documents (PKG-07), skip to [Respond](#respond).

## Load

Read these completely before starting. Rule IDs cited below are defined in them.

1. [core.md](../../resources/planning/core.md)
2. [local-authority-searches.md](../../resources/planning/local-authority-searches.md)
3. [permissions.md](../../resources/planning/permissions.md), needed for the planning handoff (HAND, PERM-01/02/05)

## Contract

| | |
| --- | --- |
| Input | One or more PDFs forming one search; optional task context (purpose, property). |
| Output | A new revision of `matters/<matter_id>/graph.json` (PKG-02, PKG-03), plus `search-<sha8>.exceptions.md` beside it. Then a reply answering the request. |
| Node types interpreted | `search_report`, `local_land_charge`, `application`, `decision`, `authority`, `document`, `unresolved_reference` |
| Not emitted | `permission`, `condition` (PERM-02: search evidence is `search_report_only`) |
| Validation level claimed | `graph-valid` at most, plus `type-valid` once schemas exist |

## Workflow

### 1. Inventory and split components

- Run `pdfinfo` for page counts and checksum each file (`shasum -a 256`).
- Get the text layer with `pdftotext -layout`. Any page with little or no text is image-only (HM Land Registry results often are): render it with `pdftoppm -r 120 -png` and read it visually.
- Assign every page to a component (SRCH-02). Recognise components by title and issuer:
  - `llc1_council`: "Register of Local Land Charges", schedules by Part.
  - `llc1_hmlr`: "Local Land Charges official search" under HM Land Registry branding.
  - `con29r`, `con29o`.
  - `additional`.
  - `con29dw`, `con29m` and `other_search`: record these but do not extract them.
  - `plan`: the search plan or property map.
  - `boilerplate`: terms, search code, complaints procedure, insurance.

  Provider packs can be mostly boilerplate (one 87-page sample had about 20 substantive pages).
- Identify form versions (CON29R-04) and the LLC format (LLC-04) before extracting anything.

### 2. Envelope first (SRCH-03)

Capture these before any entries:
- report references (council search number, provider reference, client reference);
- every labelled date (EVID-06: UK day-first);
- the property wording exactly as printed;
- the search plan page;
- the authority or authorities and each replying body;
- the report type and provider.

Stated counts ("There is 1 local land charge…", "revealed 15 subsisting registrations", a per-part summary) go into the component record for LLC-07.

### 3. Responses, in source order (RESP-01–04)

- Make one response per lowest-level subpart, including nested ones such as 3.4(c)(i), 3.10(b)(iv) and 3.15(a)(iii). A single combined answer (e.g. "(i) a decision to make an entry; or (ii) an entry … No") applies to each subpart; record that sharing.
- Answers may sit in a right-hand column (provider layouts) or on indented lines below the question (council layouts). Strip repeated page headers ("Search Reference …", "SAMPLE", provider addresses) and page numbers.
- Attach informatives to the question or section printed directly above them. Promote any temporal coverage in an informative into `scope.temporal_coverage` (RESP-04).
- Classify each answer with RESP-03. Common traps:
  - "Please see Part 3 Land Charges and/or Schedule 1 – if applicable" is `referred_elsewhere`.
  - "See attached" is `see_attachment`; find the attachment or record a gap.
  - "Not known" followed by advice is `not_known`.
  - "Not Applicable" followed by an explanatory informative is `not_applicable`; keep the informative.
- Retain any question that has no canonical code (SRCH-04).

### 4. Entries

Create an entry for each listed record, keeping its source field labels and values verbatim in `fields`. Observed shapes:

- **CON29 1.1 planning block:** `Reference`, description, `Date Decision Issued`, `Decision`, sometimes `Status`, `Site Address`, `Valid Date`.
- **CON29 1.1(j)–(l) building control:** a reference with a suffix such as `/BN`, `/NICEIC` or `/CORGI`; a status ("Plans Received", "Building works started"); a completion date. Use `entry_kind: building_control`, never planning.
- **Council LLC schedule row:** the charge description (procedure label, e.g. "Full Application"; "Conditional Planning Permission dated …"; the development description; `Application Number`; the statutory designation), originating authority, place of inspection, registration date, and an internal register reference (e.g. `TLC Ref`). Rows continue across pages, so rebuild each row from its columns before reading it. Where an address block could belong to the row before or the row after, decide from the layout image; if it is still unclear, attach the block to neither row and flag `continuation_uncertain`.
- **HM Land Registry charge block:** the labelled fields listed in LLC-04, one block per horizontal rule. Carry a value split by a page break (often `Authority reference`) onto the next page. Record that a map is present and its page; do not digitise geometry. Mark `Law`, `Creation date` and `Location` as register-supplied (LLC-05).
- **1.2 designations:** policy codes and designation names verbatim. Keep "(Within 200m)" items separate from those that apply to the property itself.

Data minimisation: omit applicants' personal names (e.g. "Issued to: …") unless the task needs them; record `personal_data_omitted: true`.

### 5. Build nodes

- **`local_land_charge`**: one per LLC entry (LLC-02), holding its format, verbatim category or part, and fields.
- **`application`** and **`decision`**: one per distinct planning record (HAND, PERM-01). `procedure` comes from the label or reference suffix through the alias table (VOCAB-05: `FUL`, "Full Application" → `full`). `outcome` comes from "Approved" or "Conditional Planning Permission" (outcome alias table). `evidence_level: search_report_only`.
- **Existing nodes:** if the graph already holds the application (for example from reviewed decision notices), merge into it (PKG-05). Never lower an `evidence_level`.
- **Identity across components (RESP-07):** the same authority plus the same reference, e.g. a council LLC row "Application Number: 14/03676/FUL" and CON29 1.1(a) "Reference: 14/03676/FUL", gives one application with two entries. Without a reference (LLC-06), or on an address or description match alone, use `possibly_related_to` (REL-02).
- **Dates:** each date keeps its own label (EVID-06). An LLC "dated 24/05/1994" with registration 24/05/1994, alongside a CON29 "Date Decision Issued: 26/05/1994", gives three labelled dates and no conflict. Same label, different value is `conflicting` (EVID-05).
- **Addresses:** use `site_address_verbatim` for each entry (PERM-06). Variant addresses for one property, such as an old street name, are normal; flag `address_variant` and never normalise them into one.
- **`authority`**: one node per named body. Record the originating authority as named, including predecessor names and "Secretary of State" or "Historic England" (CUR-07).
- **Cited parents** (e.g. "pursuant to condition 3 of 19/01234/OUT"): record the typed link at `search_report_only` (REL-05, FAM-08). The target is the matching node if one exists, otherwise an `unresolved_reference` (REL-03).

### 6. Read components together

- Apply CON29R-03. A Part 3 "No" next to an LLC entry for the same matter is expected; record a `reads_with_llc` note on the response.
- Never turn a missing schedule, an unreadable page or an unselected optional question into a negative (RESP-06, CON29O-01).

### 7. Validate and reconcile

- Every response and entry has page evidence, and every link resolves or is an unresolved reference.
- Stated counts match the counts extracted (LLC-07), or the difference is a gap.
- Every page is assigned to a component, with none left over.
- Every selected question has a response.
- No `permission` nodes; no dates derived from other dates; no translated use classes (CUR-01).

### 8. Emit

Write the graph revision (PKG-03), then the exceptions report: under one page, ordered by unreadable or missing material, count mismatches, flags needing review, and unmapped questions. Finish with one line stating the validation level reached. Do not add legal commentary.

## Respond

Answer only from the graph. Cite page numbers, and keep source wording for references, descriptions and outcomes.

- **Planning history:** a chronological table with columns reference, procedure (source label), description (search wording), outcome (source label), labelled dates, source (component, question, page) and flags. Merge appearances of the same record (RESP-07) and show every source. Order rows by the earliest labelled date, and say which label was used. Include refusals, withdrawals and pending items. Exclude building control and list it separately if asked.
- **Summary:** go through each component in order. For each, list the material entries; group negative answers in one line; then add qualifications. Mention every gap and exception.
- **Specific question:** quote the relevant response or entry. Apply CON29R-03, so a Part 3 answer is always read together with the LLC.
- **Every reply** ends with:
  - the search's "as at" date or dates;
  - the evidence level: search only, or which instruments in the graph have been reviewed;
  - any stated coverage limits (RESP-04);
  - the graph path and revision.

  For family structure, suggest `/permission-family`.

  Do not state legal conclusions (CORE-04).

## Search data

The graph envelope and assertion rules are in PKG-02 and PKG-04. `search_report.data`:
- `report_refs`, `provider`, `report_type`, `property_wording`, `plan_page`, `dates[]`, `caveats[]`;
- `components[]`: `component_id`, `kind`, `form_version`, `replying_body_id`, `pages`, `stated_count`;
- `responses[]`: `response_id`, `component_id`, `source_code`, `canonical_code`, `question_text`, `answer_verbatim`, `answer_classification`, `scope`, `informatives[]`, `entry_ids[]`, `shared_answer_with[]`, `assertion_ids`;
- `entries[]`: `entry_id`, `component_id`, `entry_kind`, `source_response_ids`, `verbatim_text`, `fields{}`, `references[]`, `dates[]`, `linked_node_ids[]`, `assertion_ids`.

Flag codes: `address_variant`, `continuation_uncertain`, `count_mismatch`, `form_version_uncertain`, `conflicting_values`, `register_supplied_field`, `reference_missing`, `component_unrecognised`, `reads_with_llc`, `pre_2016_numbering`.

## Calibration

These public samples informed the patterns above. Use them as test inputs; never commit them.

- A council-issued LLC1 and CON29 (2016) from 2020, with a planning history duplicated across the LLC Part 3(b) and CON29 1.1(a), variant addresses and differing date labels.
- A personal search from 2012 on a pre-2016 CON29 (3.10 = conservation area), with an Article 4 direction and a conservation area found only in LLC Part 3.
- A regulated search from 2019 (87 pages, mostly boilerplate), with a per-part LLC summary and "See attached" at 1.1(a).
- An HM Land Registry official search example, image-only with one charge per block and the authority reference split across a page, plus the per-authority migration notes on GOV.UK.
