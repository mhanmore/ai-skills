# Planning views

Load when the content comes from a planning matter graph (`planning-graph/graph.json`) or a view derived from it, such as `family-<root-ref>.json` or an exceptions report. Written for rules version 0.3-draft; rule IDs in brackets are for maintainers and need no other file.

## Stance

- **Read only.** Never write to the graph. The HTML file is a derived view (PKG-08): save it beside the graph as `planning-graph/<view>-r<revision>.html` and stamp the graph revision, rules version and validation level in the header and About panel.
- **Show, never decide.** The page records what documents say. Never compute or display whether a permission is implemented, lawful, extant, expired or operative, or whether a condition is discharged or complied with (CORE-04, COND-03). If the user asks for that, say the planning skills do not determine it.
- **No new facts.** Every value shown comes from a node, relationship, assertion, gap or flag in the graph. If something needed is missing, show it as missing; do not read it from the PDFs.
- **Personal data stays omitted** where the graph records `personal_data_omitted`.
- Footer on every view: "A view of the matter graph at revision [n]. It shows what the source documents say and is not legal advice."

## Documents and citations

- **Embedding:** for each entry in `graph.documents[]` that the view cites, embed the matter file whose SHA-256 equals the recorded `sha256`, passing it as `expected_sha256` to the build script. Use the `document_id` as the attachment id if it matches the id pattern; otherwise derive `doc-<first 12 hex of sha256>`. Use `page_count` as `pages`.
- **Identity without a checksum** (PKG-07 fallback): match by platform document identifier, or by file name plus page count, and say in the About panel which documents were matched that way.
- **No matching file:** never embed a different version. Show the citation without View or Download and list the document under limitations.
- **Citations:** one per assertion shown. `page` is the assertion's 1-based PDF page; `label` is the printed page label or the search question code where there is one (EVID-01); `quote` is the assertion's quote as recorded (at most 200 characters, PKG-04). Read the key names from the graph itself.
- A node derived from a search entry or instrument section cites that block's assertion; follow `source_entry_ids`, `source_response_ids` or `assertion_ids` to find it.

## Display vocabulary

Show plain labels, keeping the source label beside any normalised value (VOCAB-04).

| Graph value | Show |
| --- | --- |
| `evidence_level: search_report_only` | "Search report only" |
| `evidence_level: portal_record` | "Portal record" |
| `evidence_level: instrument_reviewed` | "Instrument reviewed" |
| `value_state: not_stated` | "Not stated in the source" |
| `value_state: illegible` | "Illegible in the source" |
| `value_state: conflicting` | "Sources conflict", with every value and its citation |
| `value_state: not_applicable` | "Not applicable" |
| `possibly_related_to` | "Possible link (inferred)", with its stated basis |
| `reported_as_related_to` | "Cited as related; type not stated" |
| `unresolved_reference` | "Cited, not in the documents reviewed" |
| `flags[]` | A review marker with the flag's message; the code in the About panel |

Relationships read as sentences: "21/0456/VAR varies 19/0123/OUT under s73 (instrument reviewed, p 1)".

Rules for values:
- **Dates:** always with their source label ("Decision date (as printed): 12/03/2020"). Never merge differently labelled dates, and never sort a mixed list by one date without naming which label it uses (EVID-06).
- **Conflicts:** show every competing value; never pick one (EVID-05).
- **Empty lists:** "None extracted within [scope]", never "None" (CORE-05). An empty "linked decisions" cell says nothing about discharge.
- **Use classes and descriptions:** exactly as recorded, never translated or tidied (CUR-01).
- **Unknown node or relationship types:** list them in the About panel with their raw type; do not style them as a known type.

## Views

Build the view the request asks for; offer others in one line.

| View | Content and form |
| --- | --- |
| Search | Register of every response in question order, including negative and "see attachment" answers; LLC and CON29 kept as separate sections (CON29R-03); entries expand under their response; gaps listed first |
| Instruments | One card per instrument in decision-date order: summary, decision wording, stated links; conditions as a register with number as printed, heading, trigger wording, text, reason and linked decisions |
| Family | Relationship map in inline SVG: root at top, each s73 branch as its own column, s96A, reserved-matters and condition decisions hung from the node they target. Stated links solid, candidates dashed, unresolved references as hollow nodes, evidence level shown on every node. Overlapping families are all drawn (FAM-04). A table view of the same nodes sits beside the map for keyboard and print use |
| Chronology | Timeline of decisions and applications by one named date label, with records lacking that date listed separately |
| Exceptions | The exceptions report as a checklist grouped by type, each item linked to its subject and citation |

Each view's header shows: matter, graph revision, rules version, validation level from the manifest, built-at time and the number of documents embedded.
