---
label: Local Search
description: >-
  Use when the user asks to read, summarise, extract from or answer questions
  about a local authority search for land in England or Wales: an LLC1 local
  land charges result (council-issued or HM Land Registry official search),
  CON29 required or optional enquiries, or a provider's personal or regulated
  search pack. Trigger on requests such as "summarise the planning history in
  the local search", "what does the CON29 say about tree preservation
  orders?", "read the search pack" or "add the search to the matter graph".
  Use this skill for reading the search itself, not for reading planning
  decision notices or permission-family mapping: use decision-notice or
  permission-family respectively for those tasks. Do not use for drainage and
  water searches (CON29DW), coal mining searches (CON29M), environmental
  reports or title documents.
userFacingDescription: >-
  Reads a local authority search (LLC1 and CON29) into a structured,
  page-referenced record and answers questions from it, such as planning
  history or specific enquiries, while flagging gaps, unreadable pages and
  points requiring human review.
---

# Local search

Status: draft · Rules version: 0.3-draft. Record the rules version in the graph manifest (PKG-07).

Typical prompts: "read the local search and summarise the planning history", "what does the search say about tree preservation orders?", "add the search to the matter graph for later".

Every request goes through the same two stages:
1. **Add the search to the matter graph, or reuse it if already there** (PKG-01). The search contributes every question and answer, every listed record, and provisional planning nodes, each traced to page-level evidence.
2. **Answer the request from the graph.**

This skill reports what the search says. It does not assess legal effect, implementation or risk.

## Locate the input

1. Use documents attached to the conversation or named in the prompt.
2. Otherwise, look in the matter's documents. If more than one could be the search, ask which.
3. Find the matter graph (PKG-01). If it already holds these documents (PKG-07), skip to [Respond](#respond).

## Load

Read these completely before starting. Rule IDs cited below are defined in them.

1. [core.md](../../resources/planning/core.md)
2. [local-authority-searches.md](../../resources/planning/local-authority-searches.md)
3. [permissions.md](../../resources/planning/permissions.md), needed for the planning handoff (HAND, PERM-01/02/05)
4. [outputs.md](references/outputs.md), the templates and examples, read at the Emit and Respond steps

## Contract

| | |
| --- | --- |
| Input | One or more PDFs forming one search; optional task context (purpose, property). |
| Output | A new revision of the matter graph (PKG-01 to PKG-03), plus a search exceptions report beside it. Then a reply answering the request. |
| Node types interpreted | `search_report`, `local_land_charge`, `application`, `decision`, `authority`, `document`, `unresolved_reference` |
| Not emitted | `permission`, `condition` (PERM-02: search evidence is `search_report_only`) |
| Validation level claimed | `graph-valid` at most, plus `type-valid` once schemas exist |

## Workflow

### 1. Inventory and split components

- Record each document's identity and page count (PKG-07).
- Read each document with [@ kind="capability" id="document-reading" label="Document reading"], keeping the page layout. Extract text page by page first, and use the text wherever it keeps the structure. Inspect a page visually only where:
  - it has little or no text (image-only; HM Land Registry results often are);
  - it holds a map, plan, annotation, handwriting, ticks or stamps;
  - a table's columns or rows are lost in the text, or the text is ambiguous;
  - a row, block or value continues from or onto another page.

  Widen visual inspection to the neighbouring pages when a discrepancy appears: a count mismatch, a broken row, a split reference or an answer that does not fit its question. Identify boilerplate from its text and do not inspect it visually. Record the method used for each page in the manifest (PKG-02).
- If the authority is Welsh, extract the search as normal, set `jurisdiction: Wales` and add a `jurisdiction_not_validated` flag (CORE-01). Do not substitute English rules or Welsh-form question text from memory.
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
- Where the form prints a subpart's answer but not its question text (common for nested subparts), set `question_text` to `null` with `value_state: not_stated`, and keep the printed code. Never supply question wording from memory of the form.

### 4. Entries

Create an entry for each listed record, keeping its source field labels and values verbatim in `fields`. Observed shapes:

- **CON29 1.1 planning block:** `Reference`, description, `Date Decision Issued`, `Decision`, sometimes `Status`, `Site Address`, `Valid Date`.
- **CON29 1.1(j)–(l) building control:** a reference with a suffix such as `/BN`, `/NICEIC` or `/CORGI`; a status ("Plans Received", "Building works started"); a completion date. Use `entry_kind: building_control`, never planning.
- **Council LLC schedule row:** the charge description (procedure label, e.g. "Full Application"; "Conditional Planning Permission dated …"; the development description; `Application Number`; the statutory designation), originating authority, place of inspection, registration date, and an internal register reference (e.g. `TLC Ref`). Rows continue across pages, so rebuild each row from its columns before reading it. Where an address block could belong to the row before or the row after, decide from the layout image; if it is still unclear, attach the block to neither row and flag `continuation_uncertain`.
- **HM Land Registry charge block:** the labelled fields listed in LLC-04, one block per horizontal rule. Carry a value split by a page break (often `Authority reference`) onto the next page. Record that a map is present and its page; do not digitise geometry. Mark `Law`, `Creation date` and `Location` as register-supplied (LLC-05).
- **1.2 designations:** policy codes and designation names verbatim. Keep "(Within 200m)" items separate from those that apply to the property itself.

Data minimisation: omit applicants' personal names (e.g. "Issued to: …") unless the task needs them; record `personal_data_omitted: true`.

### 5. Build nodes

**Evidence model (PKG-04).** Each response and entry in `search_report.data` is the canonical evidence block: it holds the verbatim text, and one `source_reported` assertion gives its page, locator and a short quote. Derived nodes (`local_land_charge`, `application`, `decision`, `authority`, `unresolved_reference`) cite those blocks through `source_entry_ids` (or `source_response_ids`) and their assertions. They do not repeat the verbatim text, the page evidence or a new `source_reported` assertion. Add a field-level assertion only for a value that is normalised (procedure, outcome, an ISO date), inferred (with its basis) or conflicting.

- **`local_land_charge`**: one per LLC entry (LLC-02), holding its format, verbatim category or part, and fields.
- **`application`** and **`decision`**: one per distinct planning record (HAND, PERM-01). `procedure` comes from the label or reference suffix through the alias table (VOCAB-05: `FUL`, "Full Application" → `full`). `outcome` comes from "Approved" or "Conditional Planning Permission" (outcome alias table). `evidence_level: search_report_only`.
- **Existing nodes:** if the graph already holds the application (for example from reviewed decision notices), merge into it (PKG-05). Never lower an `evidence_level`.
- **Identity across components (RESP-07):** the same authority plus the same reference, e.g. a council LLC row "Application Number: 14/03676/FUL" and CON29 1.1(a) "Reference: 14/03676/FUL", gives one application with two entries. Without a reference (LLC-06), or on an address or description match alone, use `possibly_related_to` (REL-02).
- **Dates:** each date keeps its own label (EVID-06). An LLC "dated 24/05/1994" with registration 24/05/1994, alongside a CON29 "Date Decision Issued: 26/05/1994", gives three labelled dates and no conflict. Same label, different value is `conflicting` (EVID-05).
- **Addresses:** use `site_address_verbatim` for each entry (PERM-06). Variant addresses for one property, such as an old street name, are normal; flag `address_variant` and never normalise them into one.
- **`authority`**: one node per named body. Record the originating authority as named, including predecessor names and "Secretary of State" or "Historic England" (CUR-07).
- **Cited parents and conditions** (HAND-02): record the typed link stated by the search at `search_report_only` (REL-05, FAM-08), with the cited wording verbatim. No search creates a `permission` or `condition` node, so:
  - a link stated against a permission ("variation of condition 2 of 19/01234/FUL", "reserved matters pursuant to 19/01234/OUT") targets the cited record's `application` node if it is in the graph, otherwise an `unresolved_reference`;
  - a link stated against a condition ("details pursuant to condition 3 of 19/01234/OUT") always targets an `unresolved_reference` for that permission and condition number, holding `candidate_parent_node_id` where the cited application is in the graph;
  - the source is the decision where an outcome is stated, otherwise the application. A pending, withdrawn or undecided application carries the link as proposed (`attributes.proposed: true`).
- **Candidate links** (HAND-03): add `possibly_related_to` only where a reference pattern (`/RM`, `/NMA`, `/DIS`, `/DOC`, a shared stem with a procedure suffix) or substantive wording suggests a procedural connection. A shared address, the same search or the chronology alone does not need an edge; @permission-family generates those candidates when it maps a family.

### 6. Read components together

- Apply CON29R-03. A Part 3 "No" next to an LLC entry for the same matter is expected; record a `reads_with_llc` note on the response.
- Never turn a missing schedule, an unreadable page or an unselected optional question into a negative (RESP-06, CON29O-01).

### 7. Validate and reconcile

Run these checks once, over the finished extraction. Fix any failure, re-run only the checks it affects, then stop.

**Structural (required for `graph-valid`):**
- Every PKG-02 key is present, IDs are unique, and every link resolves to a node or an `unresolved_reference`.
- Every response and entry has a `source_reported` assertion with a page; every derived node cites a response or entry.
- Every page is assigned to a component, with none left over, and every continuation is joined or flagged.
- Every selected question has a response, including each nested subpart and each negative answer.
- Stated counts match the counts extracted (LLC-07), or the difference is a gap.
- No `permission` or `condition` nodes; every date keeps its label; no date is derived from another; no use class is translated (CUR-01).

**Factual (targeted):** go back to the page image only for flagged items, count mismatches, continuations and any answer classified `other` or `illegible`. Do not re-read every entry. Record what was checked in the manifest.

**Formal schema (`type-valid`):** not available until schemas exist. Do not claim it, and do not write schemas during a run.

The run is complete when the structural checks pass, or each failure is recorded as a gap or flag.

### 8. Emit

Write the graph revision (PKG-03), then the exceptions report (template T5): under one page, ordered by unreadable or missing material, count mismatches, flags needing review, and unmapped questions. Finish with one line stating the validation level reached. Do not add legal commentary. These two files are the whole output of an ordinary run; write diagnostics only in test mode.

## Respond

Answer only from the graph. Use the templates in [outputs.md](references/outputs.md), and check the reply against its examples before sending. Cite page numbers, and keep source wording for references, descriptions and outcomes. Do not state legal conclusions (CORE-04).

- **Search added, no question asked (T6):** a short completion reply with the headline counts, the most important exceptions and links to the graph and the exceptions report. Do not repeat their contents.

- **Planning history (T1):** merge appearances of the same record (RESP-07) and show every source. Order rows by the earliest labelled date, and say which label was used. Include refusals, withdrawals and pending items. List building control separately, and only on request.
- **Summary (T2):** go through each component in order. Mention every gap and exception.
- **Specific question (T3):** quote the relevant response or entry. A Part 3 answer is always read together with the LLC (CON29R-03).
- **Family questions:** do not call @permission-family; it calls this skill, and calls run one way only. Where the request asks about a permission's family or the documents missing from it, offer it in one line.
- **Conditions or operative wording** of a record known only from the search: say the search does not give them, and that @decision-notice reads the decision notice once supplied.
- **Every reply** ends with the footer (T4).

## When the user asks to test this skill

Use this section only when the user explicitly asks to test, evaluate or debug the skill. Run the skill on the supplied search for real. Keep the task output (Part A) separate from a diagnostic report (Part B). Part B covers:
- ambiguous, missing or misleading instructions, and the rule IDs involved;
- layouts the workflow did not anticipate;
- which validation checks were actually performed;
- which steps were limited by the environment, such as being unable to save the graph or read page images.

Propose concrete instruction changes, but do not change the skill unless asked. Report elapsed time by stage where the environment shows it, separating skill work from repository discovery and one-off setup.

## Search data

The graph envelope and assertion rules are in PKG-02 and PKG-04. `search_report.data`:
- `report_refs`, `provider`, `report_type`, `property_wording`, `plan_page`, `dates[]`, `caveats[]`;
- `components[]`: `component_id`, `kind`, `form_version`, `replying_body_id`, `pages`, `stated_count`;
- `responses[]`: `response_id`, `component_id`, `source_code`, `canonical_code`, `question_text`, `answer_verbatim`, `answer_classification`, `scope`, `informatives[]`, `entry_ids[]`, `shared_answer_with[]`, `assertion_ids`;
- `entries[]`: `entry_id`, `component_id`, `entry_kind`, `source_response_ids`, `verbatim_text`, `fields{}`, `references[]`, `dates[]`, `linked_node_ids[]`, `assertion_ids`.

Derived nodes add `source_entry_ids` or `source_response_ids` to their `data`; the verbatim text stays on the entry.

Flag codes: `address_variant`, `continuation_uncertain`, `count_mismatch`, `form_version_uncertain`, `conflicting_values`, `register_supplied_field`, `reference_missing`, `component_unrecognised`, `reads_with_llc`, `pre_2016_numbering`, `jurisdiction_not_validated`.

## Calibration

These public samples informed the patterns above. Use them as test inputs; never commit them.

- A council-issued LLC1 and CON29 (2016) from 2020, with a planning history duplicated across the LLC Part 3(b) and CON29 1.1(a), variant addresses and differing date labels.
- A personal search from 2012 on a pre-2016 CON29 (3.10 = conservation area), with an Article 4 direction and a conservation area found only in LLC Part 3.
- A regulated search from 2019 (87 pages, mostly boilerplate), with a per-part LLC summary and "See attached" at 1.1(a).
- An HM Land Registry official search example, image-only with one charge per block and the authority reference split across a page, plus the per-authority migration notes on GOV.UK.
