---
name: planning-permission-condition-consolidation
label: Planning Permission Condition Consolidation
description: >-
  Consolidate, track, reconcile, or build a tracker for the numbered conditions
  attached to a permission, consent, or approval that has an amendment
  mechanism (variations, non-material amendments) and/or a discharge/compliance
  mechanism (approval of details, discharge of condition). Not specific to UK
  planning law - the same shape recurs anywhere a numbered-condition regime has
  (a) a principal grant, (b) a variation mechanism, and (c) a
  compliance/discharge mechanism, including environmental permits with
  variation notices, licence conditions with modification notices, and other
  regulatory approvals with a documented amendment history. Produces a
  canonical conditions JSON (every condition, in original form, with full
  amendment history, discharge/compliance status where available, a structured
  taxonomy, and a one-line subject-matter summary) and a self-contained HTML
  tracker rendering that JSON as a filterable, sortable register with 
  click-to-expand detail and optional links to source pages. 
  
  Do NOT activate for a   one-off question about a single condition's current 
  wording, or for documents with no amendment or discharge/compliance mechanism
  at all.
---

# Skill: Planning Permission Condition Consolidation

## Activation

Activate when the user asks to consolidate, track, reconcile, or build a tracker for the numbered conditions attached to a permission/consent/approval that has an amendment mechanism (variations, non-material amendments) and/or a discharge/compliance mechanism (approval of details, discharge of condition). Not specific to UK planning law — the same shape recurs anywhere a numbered-condition regime has (a) a principal grant, (b) a variation mechanism, and (c) a compliance/discharge mechanism. Examples beyond UK Town & Country Planning Act permissions: environmental permits with variation notices, licence conditions with modification notices, and any regulatory approval with a documented amendment history.

## What this skill produces

1. **A canonical conditions JSON** — every condition in the permission, in original form, with full amendment history and (where available) discharge/compliance status, a structured taxonomy (trigger stage, scope, category), and a one-line subject-matter summary per condition.
2. **A self-contained HTML tracker** — a single-file, data-driven, no-external-dependency web page rendering that JSON as a filterable/sortable table, with click-to-expand detail and optionally if requested one-click links to the exact source page for the current wording and for every amendment/discharge entry.
3. Optionally if requested an **embedded-PDF variant** (source documents base64-encoded inline, fully portable).

## Operating levels

This skill runs at three levels depending on what the user supplies. Determine the level from the inputs available before planning work — do not assume the highest level is always wanted, and do not silently downgrade if the user has supplied more than the minimum. If it is unclear which level the user wants (e.g. a document store is available but the user hasn't said whether to search it), ask.

**Level 1 — Documents only.** The user supplies the principal decision/permission document and all amendment/variation documents directly (attached, or named exactly). No search of a wider document store, no discharge/compliance data. Produces a complete, closed-family conditions JSON and tracker, scoped strictly to the supplied documents. State plainly in the `scope_note` that no independent check was made for missing amendments or discharge applications — this is a transcription-and-consolidation exercise, not a completeness audit.

**Level 2 — Documents + search, with gap-spotting.** As Level 1, plus the user has a searchable document store (project files, an organisation database, or another indexed corpus) that may contain the wider application family. Use search to confirm the supplied amendment documents are the *complete* set — do not simply trust that what was attached is everything. Actively look for signals of a missing amendment: a variation notice referencing a further amendment reference not supplied, a condition schedule referencing a plan revision that doesn't match any supplied notice, or a family/index document (a title register, a local authority search, a due diligence report) that lists applications not otherwise supplied. Report gaps found even if they cannot be resolved — a flagged gap is far more useful than a silent one.

**Level 3 — Documents + search + discharge/compliance data.** As Level 2, plus one or more sources of discharge/compliance information exist (a local-authority CON29/LLC1-style search, a compliance tracker, a register of approved-details applications) that let the tracker report live discharge status per condition, not just amendment history. This is the only level at which the tracker's Status column can be populated meaningfully — at Levels 1–2, render Status as "not assessed" rather than guessing or defaulting to "pending."

Name the achieved level explicitly in the final `scope_note` and to the user — e.g. "Level 2: consolidated from the supplied principal permission and two amendment notices, cross-checked against the project's document store; no discharge/compliance data was available so condition status is not assessed." This lets a reader (and a later re-run) know exactly what was and wasn't verified.

If a pre-built structured extract of the family history already exists for the matter (a prior run's family-index JSON, or a matter management system export), skip family-index construction and use that file as the source of truth for which amendments/discharges exist — it is faster and more accurate than re-deriving the list from free-text search. Building the family index from scratch is the one part of this pipeline where a documents-only start meaningfully costs more agent time than having the index already; if this skill will be run repeatedly for the same or related matters, persist the family-index output as a standing artifact rather than re-deriving it every time.

---

## Stage 1 — Build the family index

Goal: a definitive list of every application in the permission's family — the principal permission, every amendment/variation, and (Level 3 only) every discharge/compliance application — each with its reference, decision, and date, and (for amendments) which condition numbers it touches.

1. At Level 1, this stage is just reading the supplied documents' cover pages/headers for reference, decision, date, and (for amendments) the condition numbers touched — these are normally stated explicitly near the top of an amendment notice.
2. At Level 2–3, additionally search the document store for the principal permission reference and any documents whose filename or content mentions the jurisdiction's terms for decision notice, amendment/variation, and discharge/compliance. Run all discovery searches in parallel in one turn (different phrasings, or scoped to different subfolders) rather than search-then-read-then-search.
3. For each hit, read enough to capture: reference, application type, decision, decision date, and (for amendments) condition numbers touched. Record this as a small structured JSON, kept separate from condition-by-condition detail — a `family_index` with an `amendments_identified` list (and, at Level 3, a `discharge_applications_identified` list) is a reasonable default shape, reused as-is in Stage 3. Name the lists to fit the regime at hand (e.g. "non-material amendments" for a UK planning permission, "variation notices" for an environmental permit, "modification notices" for a licence) rather than copying a prior matter's field names verbatim.
4. **Gap-spotting (Level 2–3 only):** cross-check the amendment documents you were given against what the search turns up. If the search surfaces a reference to an amendment or discharge application not among the supplied documents, record it as a known-but-unread family member rather than silently ignoring it — flag whether its content could be read (and was incorporated) or only its existence/metadata is known.

---

## Stage 2 — Extract every condition from the principal document, verbatim

Goal: the full, original text of every numbered condition and its reason/rationale, with no amendments applied yet.

1. Identify the page range holding the conditions. Read in **large sequential page-range batches** (e.g. 15–20 pages per call) rather than page-by-page — this cuts tool calls by an order of magnitude and avoids losing continuation text where a condition spans a page boundary.
2. Transcribe each condition into a structured record as you go: `condition_number`, `heading` (write a short one if the source doesn't label it), `condition_group` (the document's own sub-heading, e.g. "Pre Commencement Condition(s) for Plot A" — a free, ready-made structural signal; capture it), `current_text`, `current_reason`.
3. Watch for **numbering defects** (duplicate numbers, skipped numbers, a condition that is clearly two run together). Do not silently renumber; record the defect, preserve the original numbering, and only apply a correction if a later amendment explicitly corrects it on its face (see Stage 3).
4. Sanity-check at the end: total condition count matches the last condition number (accounting for any known duplicate), and every `condition_group` captured is one of a small closed set — many one-off group strings signal a mis-transcribed sub-heading.

**Optimization:** don't design the trigger/category taxonomy before finishing extraction. Read all pages first (parallel batches), transcribe, and only then move to Stage 3 onward — interleaving tagging with transcription invites inconsistent tags as your understanding of the pattern set evolves partway through.

---

## Stage 3 — Layer in amendment history and discharge status

Goal: for every condition touched by an amendment, a complete `amendment_history` array (oldest first, ending with current); at Level 3, for every condition with a discharge/compliance application on record, a `discharge_applications` array.

1. For each amendment document identified in Stage 1, read it in full (same batching as Stage 2) and, for every condition number it lists as amended, capture the **complete new verbatim text** (amendment notices typically restate the condition in full, not as a diff) plus a plain-English `text_summary` of what changed versus the prior version — write the summary yourself by diffing the two texts; do not assume the notice states this.
2. Append one `amendment_history` entry per version: `{version, amending_reference, date, text_summary, source}`, `source` a citable pointer (document + page). The first entry is always "as originally granted" with `amending_reference: null`.
3. **Level 3 only:** attach a `discharge_applications` entry to the specific condition(s) each discharge/compliance application targets: `{reference, description, decision, date_decision_issued, note?}`. Where no decision is recorded yet, set `decision: null` rather than guessing — this null becomes the "pending" signal later. At Levels 1–2, omit this array entirely (or leave it absent) rather than populating it with assumptions.
4. **Numbering-defect corrections**: if an amendment's own face corrects a Stage 2 numbering defect, record that correction as an `amendment_history` entry on the corrected condition, with a `text_summary` explaining the correction is administrative (numbering/labelling only), not substantive — and keep a plain-language `condition_numbering_note` at the top level of the JSON explaining the defect and its resolution.
5. Cross-check every amendment/discharge target condition number against the Stage 2 extract — a reference to a condition number that doesn't exist is a strong signal either the family index or the extraction has an error.

**Optimization:** Stages 2 and 3's reads are fully independent per source document — read the principal document and every amendment document in the same batch of parallel tool calls at the start, rather than finishing all of Stage 2 before reading amendments. The only genuine sequencing constraint is needing the family index (Stage 1) before knowing *which* amendment documents to read.

---

## Stage 4 — Assemble and validate the consolidated JSON

Goal: one JSON file, structurally consistent, with no silent gaps.

1. Merge Stages 1–3 into: top-level metadata (site/subject, principal reference, decision date, the achieved operating level, any face-of-the-document discrepancies noticed and not reconciled), the family index, and a `consolidated_conditions` array covering **every** condition number from Stage 2 (not just the amended ones — a partial extract undermines the point of a canonical reference).
2. Validate programmatically before treating the file as done: every condition has all required fields; every amendment/discharge target condition number exists; the condition count matches the source; no condition's `amendment_history` is empty.
3. Write a `scope_note` describing exactly what the file does and does not cover — including the operating level achieved (see "Operating levels" above) and, at Levels 1–2, an explicit statement that condition status was not assessed — and a version number bumped on every structural change. This full `scope_note` is the authoritative methodological record and belongs in the JSON in complete form; it is not, by itself, what the tracker should show a user on first load — see Stage 7 point 4 for how to summarise it on the landing view without burying the register behind a wall of process detail.

**Optimization:** validate with a short script, not by re-reading the JSON by eye — checking dozens or hundreds of records for missing fields or dangling references is exactly the kind of task a script catches reliably and a manual scan does not.

---

## Stage 5 — Design and apply the taxonomy (trigger / category / scope)

Goal: every condition tagged with a structured, filterable classification, not just prose — and, critically, a scope model that supports "what applies to X" (inclusive of parent-level conditions with wider scope inclusive of X) as well as "what applies only to X." (narrowly scoped). For example X might be a building, that sits within a plot, that sits within a phase. Planning permissions commonly impose conditions that operate at each of those geospatial scopes, so the lower scoped targets 'inherit' conditions that sit above them.

1. **Design the trigger-stage set** from patterns observed while reading (don't design this abstractly before reading anything). Expect something like: pre-commencement, pre-above-ground/pre-superstructure, a mid-construction milestone if the regime has one, pre-occupation, pre-a-specific-use-commencing (distinct from pre-occupation generally), during-construction/ongoing monitoring, post-completion reporting, ongoing/perpetual compliance, event-triggered, and any pure time-limit condition. Give each a colour for later use in the report.

2. **Design the scope set as a hierarchy from the outset, not a flat list with hierarchy bolted on.** Graded permissions — site/phase/plot/building, or estate/unit/room, or programme/project/workstream — are the norm, not the exception, and overlap is structural: a site-wide condition binds every phase and every unit within it; a phase-wide condition binds every unit within that phase. **Treat the containment hierarchy as a required, first-class part of the scope model, not an optional enhancement**: derive the flat scope list and the `scopeAncestors` map from the actual document's own structure (e.g. for a scheme with an overall site divided into two phases, each phase containing two buildings, you might derive `site_wide`, `phase_1`, `phase_2`, `unit_1a`, `unit_1b`, `unit_2a`, `unit_2b`, with `scopeAncestors = {"unit_1a": ["phase_1", "site_wide"], "phase_1": ["site_wide"], ...}`) in the same pass, before tagging a single condition. Never copy a prior matter's scope names (its actual plot/building/unit labels) into a new matter — the hierarchy's *shape* generalises, its labels do not. This is not "cheap to add now, expensive later" — it is a design defect to omit, because every downstream question a reader actually asks ("what applies to this specific unit?") is inherently inclusive of parent scopes, and a flat scope model without the hierarchy silently gives the wrong answer to that question by construction.

3. **Design the category set** from patterns across the full read (not a generic textbook list) — expect one category per substantive topic area, plus a few genuinely one-off categories for bespoke conditions. Don't force a singleton into a bigger bucket just to avoid a short category list. As a loose sanity check, a large mixed-use scheme typically produces somewhere in the range of 15–40 categories; a much shorter or much longer list is worth a second look, though the actual count is genuinely regime/site-specific and should never be copied wholesale from a prior matter.

4. **Multi-trigger conditions**: represent triggers as an **array** per condition, each entry `{stage, scope, qualifier?}` — many real conditions have more than one distinct trigger point, and flattening to one trigger per condition loses this.

5. Tag every condition using a script holding a lookup table (condition number → tags), not by generating JSON by hand for dozens of records — a table is easier to review for gaps and consistency, and trivially lets you programmatically confirm 100% coverage.

6. Validate: every stage/scope/category value used on a condition exists in the corresponding option list (no ad hoc drift between the lookup table and the documented taxonomy); every condition has a non-empty `scope` array; every scope value used on a condition also exists as a key (or a valid leaf) in the `scopeAncestors` map.

---

## Stage 6 — Write subject-matter sentences (judgment task, not mechanical)

Goal: one short (roughly 8–15 word), high-level sentence per condition that lets a reader distinguish it from every other condition in a flat list without opening the full text.

1. This cannot be templated reliably — write each one by hand, informed by the condition's actual substance. The main risk is near-duplicate conditions (mirrored plot/building pairs, or a condition amended down to a narrower scope than its original): make sure the sentence itself carries the distinguishing fact rather than being identical for both halves of a mirrored pair.
2. Where an amendment changed *scope* (e.g. a condition originally covering two buildings now covers only one), say so directly in the subject-matter sentence.
3. Apply via the same lookup-table + script pattern as Stage 5, and validate 100% coverage the same way.

---

## Stage 7 — Build the HTML tracker (single self-contained file)

Goal: a No / Subject Matter / Trigger / Status table; filter/search; click-to-expand detail; embedded data if requested; no external requests; reusable for a different site's schema.

1. **Structure as three concerns, not one file typed linearly**: (a) CSS in `<style>`, using CSS variables for the colour palette and status colours; (b) the data, embedded as `<script type="application/json" id="...">` (not inlined into a JS object literal — a dedicated JSON script tag is trivially re-parseable and swappable for a different dataset later); (c) a JS `CONFIG` object naming every field/taxonomy-list/status-derivation rule the rendering logic depends on, followed by rendering logic that refers only to `CONFIG` and never hardcodes a field name inline.
2. **`table-layout: fixed` is required** the moment column widths are declared as percentages on `<th>` — without it, browsers auto-size every column to its widest content regardless of the declared width. Set it from the start.
3. **Visual identity is deliberately unspecified beyond the structural requirements in this stage** (CSS variables, `table-layout: fixed`, colour-coded stages/status, three-concerns separation). Choose typography, palette, and layout metaphor based on the artefact's actual context — a statutory conditions register for a property transaction reads differently from a general project dashboard, and a licence-condition tracker for a different regime again. Treat this as a fresh design decision on every run, the same way the category list and scope hierarchy are re-derived from the actual document set rather than copied from a prior matter — do not default to a previous run's visual style out of habit.

4. **The landing view is user-facing output, not a specification recital.** The page the user first sees (title, subtitle, any introductory note) should read like a finished work product — plain statements of what the register covers and its source (site/subject, principal reference, decision date, amendments applied) — not a restatement of this skill's internal mechanics. In particular, the `scope_note` field belongs in the JSON as the authoritative record of what was and wasn't verified, but do not paste its full text verbatim onto the tracker's landing page: summarise it in one or two short, plain-English sentences a solicitor or client could read at a glance (e.g. what level was achieved and the one or two most material gaps), and put the complete methodological detail — the full family index, any excluded-family notes, the per-condition discrepancy notes — behind a click (a collapsible "About this register" section, or inside the row-level detail) rather than in the page header. A cluttered landing view undermines the same goal as a wrongly-defaulted filter: it makes the single most important question ("what does this tell me, right now") harder to answer at a glance.

5. **Status must be derived, not hand-entered**, from whatever discharge/compliance data exists — write the derivation as a single named function in `CONFIG`. At Levels 1–2 (no discharge data), this function should return a fixed "not assessed" status for every condition rather than fabricating a status from absence of data. **Internal status keys (`not_assessed`, `not_triggered`, `pending_discharge`, `partially_discharged`, `discharged`) are for the data model only — do not render them verbatim as user-facing labels.** `not_triggered` in particular reads, in plain English, as a claim about what stage construction has reached ("this hasn't been triggered yet"), which a document-consolidation exercise has no basis to assert — it can only report what discharge/compliance record it did or didn't find. Render it with reporting-neutral wording instead, e.g. "No discharge data found," and add a one-line caption near the status legend clarifying that absence of a discharge record is not confirmation that no discharge is required and is not a statement about construction progress. Apply the same discipline to any other status label that could be misread as a substantive finding rather than a report of what was found in the sources reviewed.

6. **Filter design, scope-inclusive by default:**
   - Free-text search across the fields that matter (subject, heading, full text, ID).
   - Independent dropdowns per taxonomy axis (category, scope, stage, status).
   - A colour-coded legend that doubles as a one-click filter shortcut.
   - **The scope filter must default to answering "what applies to X" (inclusive of ancestor scopes), not "what applies only to X."** Implement scope matching as one small pure function, `scopeMatches(conditionScopes, selectedScope, exclusiveOnly)`, where the default (`exclusiveOnly = false`) matches a condition if the selected scope is in its scope list **or** the *condition's own scope* appears in `scopeAncestors[selectedScope]` — i.e. you walk up from the **selected** scope to see what it descends from, not up from the condition. For example, with a hierarchy of `site_wide` > `phase_1` > `unit_1a`, selecting `unit_1a` must also return every `phase_1` and `site_wide` condition, because `phase_1` and `site_wide` are `unit_1a`'s ancestors. This direction is easy to invert by accident because both phrasings ("X is an ancestor of Y" vs "Y is an ancestor of X") sound similar; after implementing, sanity-check with the actual scope hierarchy derived for this matter before moving on: select the narrowest scope and confirm every wider (ancestor) condition appears, and confirm a sibling scope's conditions do NOT appear. Provide an explicit toggle — labelled something like "this scope only" — to switch to the narrower, exclusive-match behaviour, rather than the reverse. Getting the default backwards silently produces a materially incomplete answer to the single question this filter exists to answer, so treat the direction of the default as a correctness requirement, not a UX preference. Because inclusive is now the default, this control should always be visible when a scope taxonomy with any hierarchy exists — it does not need to hide itself for scopes with no ancestors, since the toggle in that case is simply inert.

7. **Expand-on-click** via a sibling `<tr class="detail-row">` per row, toggled by a Set of expanded IDs re-rendered on every filter/sort change. Guard the row's click handler so clicks on any interactive control nested inside the row (a source link, a button) do not also toggle the row — check `event.target.closest("[data-stop-row-toggle]")` and bail early.

8. **Validate before every save**: HTML has a DOCTYPE and balanced tags; script tag count is even; no external `src="http`, no `rel="stylesheet"`, no CDN references; the embedded JSON parses and has the expected record count; extract the logic `<script>` block and run it through a JS syntax checker (e.g. `node --check`).

**Optimization:** build the CSS/body/script as separate text fragments assembled by a small script, rather than one giant literal — this makes later patches targeted string replacements against a known fragment rather than a full re-generation.

---

## Stage 8 — Link (or embed) the source documents

Trigger: only apply stage 8 if the user has requested linked documents. Default mode will be free-standing report that refers to but does not include source documents. 

Goal: a link at summary level to the condition's *current* source page, and a link on every amendment/discharge entry to *its own* source page, opening in a new tab without triggering a download.

Options: files can be structured either as resources saved alongside the html, or as base64 encoded embeddings. Side car resources are fragile in serverless html deployment, so user references to embedding, self-containment etc. sohould be treated as a preferences sfor a base64 encoded html file despite resulting in large download sizes.

1. Parse each `source` reference already captured in Stage 3 into a structured `{docId, page}` reference; treat a source with no exact page as `page: null` and fall back to page 1 in the eventual open-document call. Do this as a script pass over the whole JSON, adding `source_ref` to every `amendment_history` entry, `discharge_applications` entry, and a top-level `current_source_ref` per condition (= the `source_ref` of the *last* `amendment_history` entry).

2. **Two variants, same template, usuall base64 embedded unless both are requested.** An embedded variant base64-encodes source documents into a JS object (`DOC_REGISTRY`); a linked variant uses the same registry shape holding relative filenames instead, shipped as sibling files. Build both from one shared HTML template with placeholder tokens substituted per variant — do not maintain two hand-diverged copies. If the user has asked for only one variant, build only that one; there is no need to engineer the dual-template abstraction for a single-variant request.

3. **Opening a PDF from a `blob:` source without triggering a download is the one real cross-browser trap.** Navigating a new tab directly to a `blob:` URL for an `application/pdf` Blob renders inline in some Chromium browsers but is treated as a download in others. The robust fix: open the blank tab **synchronously** in the click handler (so it isn't caught by a popup blocker), then `document.write` a minimal same-origin wrapper page into that tab containing `<embed src="[blob url]" type="application/pdf">`. For the linked (real file URL) variant this trap generally does not apply, but consider the same wrapper approach anyway for consistency.

4. Sanitize anything interpolated into a `document.write` call (strip `<`, `>`, `&` from a filename before using it as a wrapper-page `<title>`).

5. Guard summary-level and per-entry link buttons against the row's own click-to-expand handler (Stage 7, point 7).

6. Validate the base64 registry round-trips exactly: decode every entry back to bytes and diff against the original file bytes before saving.

---

## Stage 9 — Save discipline

1. **Validate structurally before every `suggest-save`**, not after.

2. **Use `mode: "overwrite"` for every revision to a file that already exists at that destination.** Using `create` for a fix to a file already saved produces a second, differently-named file rather than replacing it.

3. **One pending save per destination at a time.**

4. **Scratch does not reliably persist between conversation turns.** Do not assume a file written to scratch in an earlier turn is still there in a later turn — either keep dependent work within one turn, or re-derive/re-copy what's needed at the start of a new turn. If a scratch file has disappeared, re-fetch its source rather than assuming an in-memory record of "what's in scratch" is still accurate.

---

## Quick-reference: taxonomy schema shape

```
trigger_stage_options: pre_commencement | pre_above_ground | pre_completion_of_frame |
  pre_occupation | pre_use_commencing | during_construction | post_completion |
  ongoing_perpetual | event_triggered | time_limited

scope model (hierarchy is required, not optional):
  scope_options: <flat list reflecting the actual site/programme/regime structure>
  scopeAncestors: <map from each scope to every broader scope that contains it,
    e.g. unit_1a -> [phase_1, site_wide]; phase_1 -> [site_wide]; site_wide -> []>

scope filter matching (default = inclusive):
  scopeMatches(conditionScopes, selectedScope, exclusiveOnly):
    if exclusiveOnly: selectedScope is literally in conditionScopes
    else (default):   selectedScope is in conditionScopes
                       OR some scope in conditionScopes appears in scopeAncestors[selectedScope]
  # Direction check: selecting a NARROW scope must pull in conditions scoped to ITS OWN
  # ancestors (the broader scopes that contain it) — walk UP from the selection, not from
  # the condition. Concretely: scopeAncestors[selectedScope].some(a => conditionScopes.includes(a))
  # Worked example: with scopeAncestors = {unit_1a: [phase_1, site_wide], phase_1: [site_wide], ...},
  # selecting unit_1a must match a condition scoped only to ["phase_1"] or only to ["site_wide"] —
  # a condition scoped to ["unit_1b"] (a sibling) must NOT match. If your implementation matches
  # the reverse case (a wide selection pulling in narrow conditions), the direction is inverted —
  # fix it before shipping the tracker, this is a correctness bug, not a style choice.

status derivation (Level 3 only; Level 1-2 -> fixed "not_assessed"):
  no discharge_applications entries                           -> no_discharge_data
  all entries decision == "Granted" or "Approved" or similar  -> discharged
  some (not all) "Granted" etc.                               -> partially_discharged
  any entry decision is null                                  -> pending_discharge
```

Re-derive the category list, the flat scope list, and the scope-ancestor map from the actual document set every time — all three are genuinely regime/site-specific, and a fixed list or map from a prior matter should only be a starting hypothesis, never copied wholesale.
