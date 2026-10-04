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
  amendment history, a substantively assessed discharge/compliance status
  where available, a structured taxonomy, and a one-line subject-matter
  summary) and a self-contained HTML tracker rendering that JSON as a
  filterable, sortable register with click-to-expand detail and links to
  source pages. Do NOT activate for a one-off question about a single
  condition's current wording, or for documents with no amendment or
  discharge/compliance mechanism at all.
---

# Skill: Planning Permission Condition Consolidation

## Activation

Activate when the user asks to consolidate, track, reconcile, or build a tracker for the numbered conditions attached to a permission/consent/approval that has an amendment mechanism (variations, non-material amendments) and/or a discharge/compliance mechanism (approval of details, discharge of condition). Not specific to UK planning law — the same shape recurs anywhere a numbered-condition regime has (a) a principal grant, (b) a variation mechanism, and (c) a compliance/discharge mechanism. Examples beyond UK Town & Country Planning Act permissions: environmental permits with variation notices, licence conditions with modification notices, and any regulatory approval with a documented amendment history.

## What this skill produces

1. **A canonical conditions JSON** — every condition in the permission, in original form, with full amendment history, a structured taxonomy (trigger stage, scope, category), a one-line subject-matter summary per condition, and (where discharge/compliance material exists) a substantively assessed discharge status per condition — not merely a transcription of a decision notice's own label, but the AI's own judgment of how much of the condition's actual requirements the discharge material satisfies.
2. **A self-contained HTML tracker** — a single-file, data-driven, no-external-dependency web page rendering that JSON as a filterable/sortable table, with click-to-expand detail, a discharge-assessment rationale per condition, and one-click links to the exact source page for the current wording and for every amendment/discharge entry.
3. Optionally, an **embedded-PDF variant** (source documents base64-encoded inline, fully portable) alongside a **linked-PDF variant** (smaller file, source documents shipped as siblings).

## Operating levels

This skill runs at three levels depending on what the user supplies. Determine the level from the inputs available before planning work — do not assume the highest level is always wanted, and do not silently downgrade if the user has supplied more than the minimum. If it is unclear which level the user wants (e.g. a document store is available but the user hasn't said whether to search it), ask.

**Level 1 — Documents only.** The user supplies the principal decision/permission document and all amendment/variation documents directly (attached, or named exactly). No search of a wider document store, no discharge/compliance review. Produces a complete, closed-family conditions JSON and tracker, scoped strictly to the supplied documents. State plainly in the `scope_note` that no independent check was made for missing amendments or discharge applications, and that discharge status was not assessed — this is a transcription-and-consolidation exercise, not a completeness or compliance audit.

**Level 2 — Documents + search, with gap-spotting.** As Level 1, plus the user has a searchable document store (project files, an organisation database, or another indexed corpus) that may contain the wider application family. Use search to confirm the supplied amendment documents are the *complete* set — do not simply trust that what was attached is everything. Actively look for signals of a missing amendment: a variation notice referencing a further amendment reference not supplied, a condition schedule referencing a plan revision that doesn't match any supplied notice, or a family/index document (a title register, a local authority search, a due diligence report) that lists applications not otherwise supplied. Report gaps found even if they cannot be resolved — a flagged gap is far more useful than a silent one. Discharge status is still not assessed at this level unless discharge/compliance source material is actually found and read (see Level 3).

**Level 3 — Documents + search + discharge/compliance review.** As Level 2, plus one or more sources of discharge/compliance material exist (a local-authority CON29/LLC1-style search, a compliance tracker, a register of approved-details applications, or the discharge/approval-of-details submissions and decisions themselves) that let the AI actively review, condition by condition, whether and to what extent each has been discharged. This is not a pass-through of a status label already assigned by someone else: the AI reads the discharge/compliance material itself and forms its own judgment of full, partial, or no discharge against the condition's actual requirements (see Stage 4). This is the only level at which the tracker's Status column can be populated meaningfully — at Levels 1–2, render Status as "not assessed" rather than guessing or defaulting to "pending."

Name the achieved level explicitly in the final `scope_note` and to the user — e.g. "Level 3: consolidated from the supplied principal permission and two amendment notices, cross-checked against the project's document store, with discharge status substantively assessed against three approval-of-details decisions; two conditions have no discharge material on record." This lets a reader (and a later re-run) know exactly what was and wasn't verified.

If a pre-built structured extract of the family history already exists for the matter (a prior run's family-index JSON, or a matter management system export), skip family-index construction and use that file as the source of truth for which amendments/discharges exist — it is faster and more accurate than re-deriving the list from free-text search. Building the family index from scratch is the one part of this pipeline where a documents-only start meaningfully costs more agent time than having the index already; if this skill will be run repeatedly for the same or related matters, persist the family-index output — and, once built, the discharge-assessment output from Stage 4 — as a standing artifact rather than re-deriving it every time.

---

## Stage 1 — Build the family index

Goal: a definitive list of every application in the permission's family — the principal permission, every amendment/variation, and (Level 3 only) every discharge/compliance application — each with its reference, decision, and date, and (for amendments) which condition numbers it touches.

1. At Level 1, this stage is just reading the supplied documents' cover pages/headers for reference, decision, date, and (for amendments) the condition numbers touched — these are normally stated explicitly near the top of an amendment notice.
2. At Level 2–3, additionally search the document store for the principal permission reference and any documents whose filename or content mentions the jurisdiction's terms for decision notice, amendment/variation, and discharge/compliance. Run all discovery searches in parallel in one turn (different phrasings, or scoped to different subfolders) rather than search-then-read-then-search.
3. For each hit, read enough to capture: reference, application type, decision, decision date, and (for amendments) condition numbers touched. For a discharge/compliance-type hit, also note the condition number(s) it targets — full substantive reading of its content happens in Stage 4, not here. Record this as a small structured JSON, kept separate from condition-by-condition detail — a `family_index` with an `amendments_identified` list and, at Level 3, a `discharge_applications_identified` list is a reasonable default shape, reused as-is in Stage 3. Name the lists to fit the regime at hand (e.g. "non-material amendments" for a UK planning permission, "variation notices" for an environmental permit, "modification notices" for a licence) rather than copying a prior matter's field names verbatim.
4. **Gap-spotting (Level 2–3 only):** cross-check the amendment and discharge documents you were given against what the search turns up. If the search surfaces a reference to an amendment or discharge application not among the supplied documents, record it as a known-but-unread family member rather than silently ignoring it — flag whether its content could be read (and was incorporated) or only its existence/metadata is known. A discharge application whose existence is known but whose content cannot be read must not be assessed in Stage 4 — record it as an unresolved gap instead.

---

## Stage 2 — Extract every condition from the principal document, verbatim

Goal: the full, original text of every numbered condition and its reason/rationale, with no amendments applied yet.

1. Identify the page range holding the conditions. Read in **large sequential page-range batches** (e.g. 15–20 pages per call) rather than page-by-page — this cuts tool calls by an order of magnitude and avoids losing continuation text where a condition spans a page boundary.
2. Transcribe each condition into a structured record as you go: `condition_number`, `heading` (write a short one if the source doesn't label it), `condition_group` (the document's own sub-heading, e.g. "Pre Commencement Condition(s) for Plot A" — a free, ready-made structural signal; capture it), `current_text`, `current_reason`.
3. Watch for **numbering defects** (duplicate numbers, skipped numbers, a condition that is clearly two run together). Do not silently renumber; record the defect, preserve the original numbering, and only apply a correction if a later amendment explicitly corrects it on its face (see Stage 3).
4. Sanity-check at the end: total condition count matches the last condition number (accounting for any known duplicate), and every `condition_group` captured is one of a small closed set — many one-off group strings signal a mis-transcribed sub-heading.

**Optimization:** don't design the trigger/category taxonomy before finishing extraction. Read all pages first (parallel batches), transcribe, and only then move to Stage 3 onward — interleaving tagging with transcription invites inconsistent tags as your understanding of the pattern set evolves partway through.

---

## Stage 3 — Layer in amendment history

Goal: for every condition touched by an amendment, a complete `amendment_history` array (oldest first, ending with current).

1. For each amendment document identified in Stage 1, read it in full (same batching as Stage 2) and, for every condition number it lists as amended, capture the **complete new verbatim text** (amendment notices typically restate the condition in full, not as a diff) plus a plain-English `text_summary` of what changed versus the prior version — write the summary yourself by diffing the two texts; do not assume the notice states this.
2. Append one `amendment_history` entry per version: `{version, amending_reference, date, text_summary, source}`, `source` a citable pointer (document + page). The first entry is always "as originally granted" with `amending_reference: null`.
3. **Numbering-defect corrections**: if an amendment's own face corrects a Stage 2 numbering defect, record that correction as an `amendment_history` entry on the corrected condition, with a `text_summary` explaining the correction is administrative (numbering/labelling only), not substantive — and keep a plain-language `condition_numbering_note` at the top level of the JSON explaining the defect and its resolution.
4. Cross-check every amendment target condition number against the Stage 2 extract — a reference to a condition number that doesn't exist is a strong signal either the family index or the extraction has an error. The current version of a condition (the latest `amendment_history` entry) is what Stage 4 assesses discharge against, not the original wording — always assess a condition as currently amended, never as originally granted, unless no amendment has touched it.

**Optimization:** Stages 2 and 3's reads are fully independent per source document — read the principal document and every amendment document in the same batch of parallel tool calls at the start, rather than finishing all of Stage 2 before reading amendments. The only genuine sequencing constraint is needing the family index (Stage 1) before knowing *which* amendment documents to read.

---

## Stage 4 — Review and assess discharge/compliance status (Level 3 only)

Goal: for every condition with discharge/compliance material identified in Stage 1, a `discharge_applications` array recording what was submitted and decided, **plus an AI-formed judgment of whether that material discharges the condition in full, in part, or not at all**, tested against the condition's actual current requirements (Stage 3's current text) rather than taken at face value from a decision label alone. Skip this stage entirely at Levels 1–2 and state plainly that discharge was not assessed.

1. **Read the discharge/compliance material itself, in full**, for every application identified in Stage 1 (same large-batch page-range approach as Stages 2–3) — the application/submission, any conditions or caveats attached to its approval, and the decision notice or equivalent. A decision notice's own headline outcome ("approved," "granted," "discharged") is a starting signal, not the answer: read what was actually submitted and approved before relying on that label.
2. **Decompose multi-part conditions before assessing.** Many conditions impose several distinct requirements in one numbered clause (e.g. a landscaping condition requiring a scheme, an implementation timetable, and a maintenance regime). Identify the condition's distinct sub-requirements from its current text, and check each one separately against what the discharge material actually covers — a decision that discharges only some sub-requirements is a partial discharge even if its own covering letter describes itself as approving "the details submitted," because "the details submitted" may not have addressed every sub-requirement the condition imposes.
3. **Classify the outcome per condition** using a small closed set, and always give a `discharge_rationale` — a short plain-English explanation of *why* that classification was reached, naming which sub-requirements are and are not covered — never just the label on its own:
   - `discharged` — the discharge material, on its face, addresses every sub-requirement of the condition's current text, and the decision approves it without a caveat that leaves any part outstanding.
   - `partially_discharged` — the discharge material addresses some but not all of the condition's sub-requirements, or approves part of the scope (e.g. one plot of several) while the rest remains outstanding, or approves subject to a further condition/caveat that itself amounts to an outstanding requirement.
   - `pending_discharge` — an application has been submitted but no decision is recorded, or the decision itself is provisional/deferred.
   - `no_discharge_data` — no discharge/compliance application was identified for this condition at all (this is a report of absence, not a finding that discharge is unnecessary — see Stage 7 point 5 on rendering this without overclaiming).
   - If a condition has been the subject of more than one discharge application (e.g. a part-discharge followed by a further application covering the remainder), assess the **combined effect** of all applications together, and record each application separately in the array with its own individual outcome as well as the condition-level combined outcome.
4. Append one `discharge_applications` entry per application: `{reference, description, decision, date_decision_issued, sub_requirements_covered, sub_requirements_outstanding, note?, source}`, `source` a citable pointer (document + page), consistent with the `amendment_history` sourcing pattern in Stage 3. Where no decision is recorded yet, set `decision: null` rather than guessing.
5. Set a condition-level `discharge_status` (one of the four values in point 3) and `discharge_rationale` on the `consolidated_conditions` record itself, derived from the `discharge_applications` array — this is what Stage 7's Status column renders, so it must be present on every condition assessed at this level, not buried only inside the array.
6. Cross-check every discharge target condition number against the Stage 2/3 extract, exactly as Stage 3 does for amendments — a reference to a condition number that doesn't exist signals an error upstream.
7. **This is a judgment task, not a mechanical lookup** — treat it with the same care as Stage 6's subject-matter sentences. Do not default to "discharged" merely because a decision notice uses an approving word, and do not default to "partially_discharged" merely because a condition has several sub-clauses if the discharge material in fact covers all of them. Where the discharge material is genuinely ambiguous about coverage, say so in the `discharge_rationale` rather than forcing a confident classification the source doesn't support.

**Optimization:** batch the discharge-material reads together with the amendment reads from Stage 3 wherever both are already known from Stage 1 — the only hard sequencing constraint is that Stage 4's assessment needs each condition's *current* (post-amendment) text from Stage 3, so finish capturing current text for a condition before assessing its discharge status, even if the underlying documents were fetched in the same batch.

---

## Stage 5 — Assemble and validate the consolidated JSON

Goal: one JSON file, structurally consistent, with no silent gaps.

1. Merge Stages 1–4 into: top-level metadata (site/subject, principal reference, decision date, the achieved operating level, any face-of-the-document discrepancies noticed and not reconciled), the family index, and a `consolidated_conditions` array covering **every** condition number from Stage 2 (not just the amended or discharge-assessed ones — a partial extract undermines the point of a canonical reference).
2. Validate programmatically before treating the file as done: every condition has all required fields; every amendment/discharge target condition number exists; the condition count matches the source; no condition's `amendment_history` is empty; at Level 3, every condition has a `discharge_status` value from the closed set and, where that value is not `no_discharge_data`, a non-empty `discharge_rationale`.
3. Write a `scope_note` describing exactly what the file does and does not cover — including the operating level achieved (see "Operating levels" above), and, at Level 3, a summary of how many conditions were reviewed for discharge and how many had no discharge material found, and, at Levels 1–2, an explicit statement that discharge status was not assessed — and a version number bumped on every structural change. This full `scope_note` is the authoritative methodological record and belongs in the JSON in complete form; it is not, by itself, what the tracker should show a user on first load — see Stage 8 point 4 for how to summarise it on the landing view without burying the register behind a wall of process detail.

**Optimization:** validate with a short script, not by re-reading the JSON by eye — checking dozens or hundreds of records for missing fields or dangling references is exactly the kind of task a script catches reliably and a manual scan does not.

---

## Stage 6 — Design and apply the taxonomy (trigger / category / scope)

Goal: every condition tagged with a structured, filterable classification, not just prose — and, critically, a scope model that supports "what applies to X" (inclusive of parent-level conditions with wider scope inclusive of X) as well as "what applies only to X." (narrowly scoped). For example X might be a building, that sits within a plot, that sits within a phase. Planning permissions commonly impose conditions that operate at each of those geospatial scopes, so the lower scoped targets 'inherit' conditions that sit above them.

1. **Design the trigger-stage set** from patterns observed while reading (don't design this abstractly before reading anything). Expect something like: pre-commencement, pre-above-ground/pre-superstructure, a mid-construction milestone if the regime has one, pre-occupation, pre-a-specific-use-commencing (distinct from pre-occupation generally), during-construction/ongoing monitoring, post-completion reporting, ongoing/perpetual compliance, event-triggered, and any pure time-limit condition. Give each a colour for later use in the report.

2. **Design the scope set as a hierarchy from the outset, not a flat list with hierarchy bolted on.** Graded permissions — site/phase/plot/building, or estate/unit/room, or programme/project/workstream — are the norm, not the exception, and overlap is structural: a site-wide condition binds every phase and every unit within it; a phase-wide condition binds every unit within that phase. **Treat the containment hierarchy as a required, first-class part of the scope model, not an optional enhancement**: derive the flat scope list and the `scopeAncestors` map from the actual document's own structure (e.g. for a scheme with an overall site divided into two phases, each phase containing two buildings, you might derive `site_wide`, `phase_1`, `phase_2`, `unit_1a`, `unit_1b`, `unit_2a`, `unit_2b`, with `scopeAncestors = {"unit_1a": ["phase_1", "site_wide"], "phase_1": ["site_wide"], ...}`) in the same pass, before tagging a single condition. Never copy a prior matter's scope names (its actual plot/building/unit labels) into a new matter — the hierarchy's *shape* generalises, its labels do not. This is not "cheap to add now, expensive later" — it is a design defect to omit, because every downstream question a reader actually asks ("what applies to this specific unit?") is inherently inclusive of parent scopes, and a flat scope model without the hierarchy silently gives the wrong answer to that question by construction.

3. **Design the category set** from patterns across the full read (not a generic textbook list) — expect one category per substantive topic area, plus a few genuinely one-off categories for bespoke conditions. Don't force a singleton into a bigger bucket just to avoid a short category list. As a loose sanity check, a large mixed-use scheme typically produces somewhere in the range of 15–40 categories; a much shorter or much longer list is worth a second look, though the actual count is genuinely regime/site-specific and should never be copied wholesale from a prior matter.

4. **Multi-trigger conditions**: represent triggers as an **array** per condition, each entry `{stage, scope, qualifier?}` — many real conditions have more than one distinct trigger point, and flattening to one trigger per condition loses this.

5. Tag every condition using a script holding a lookup table (condition number → tags), not by generating JSON by hand for dozens of records — a table is easier to review for gaps and consistency, and trivially lets you programmatically confirm 100% coverage.

6. Validate: every stage/scope/category value used on a condition exists in the corresponding option list (no ad hoc drift between the lookup table and the documented taxonomy); every condition has a non-empty `scope` array; every scope value used on a condition also exists as a key (or a valid leaf) in the `scopeAncestors` map.

---

## Stage 7 — Write subject-matter sentences (judgment task, not mechanical)

Goal: one short (roughly 8–15 word), high-level sentence per condition that lets a reader distinguish it from every other condition in a flat list without opening the full text.

1. This cannot be templated reliably — write each one by hand, informed by the condition's actual substance. The main risk is near-duplicate conditions (mirrored plot/building pairs, or a condition amended down to a narrower scope than its original): make sure the sentence itself carries the distinguishing fact rather than being identical for both halves of a mirrored pair.
2. Where an amendment changed *scope* (e.g. a condition originally covering two buildings now covers only one), say so directly in the subject-matter sentence.
3. Where a condition is only partially discharged, the subject-matter sentence should still describe the condition's substance, not its discharge state — discharge status belongs in the dedicated `discharge_status`/`discharge_rationale` fields and the tracker's Status column, not folded into the subject-matter sentence.
4. Apply via the same lookup-table + script pattern as Stage 6, and validate 100% coverage the same way.

---

## Stage 8 — Build the HTML tracker (single self-contained file)

Goal: a No / Subject Matter / Trigger / Status table; filter/search; click-to-expand detail; embedded data; no external requests; reusable for a different site's schema.

1. **Structure as three concerns, not one file typed linearly**: (a) CSS in `<style>`, using CSS variables for the colour palette and status colours; (b) the data, embedded as `<script type="application/json" id="...">` (not inlined into a JS object literal — a dedicated JSON script tag is trivially re-parseable and swappable for a different dataset later); (c) a JS `CONFIG` object naming every field/taxonomy-list/status-derivation rule the rendering logic depends on, followed by rendering logic that refers only to `CONFIG` and never hardcodes a field name inline.
2. **`table-layout: fixed` is required** the moment column widths are declared as percentages on `<th>` — without it, browsers auto-size every column to its widest content regardless of the declared width. Set it from the start.
3. **Visual identity is deliberately unspecified beyond the structural requirements in this stage** (CSS variables, `table-layout: fixed`, colour-coded stages/status, three-concerns separation). Choose typography, palette, and layout metaphor based on the artefact's actual context — a statutory conditions register for a property transaction reads differently from a general project dashboard, and a licence-condition tracker for a different regime again. Treat this as a fresh design decision on every run, the same way the category list and scope hierarchy are re-derived from the actual document set rather than copied from a prior matter — do not default to a previous run's visual style out of habit.

4. **The landing view is user-facing output, not a specification recital.** The page the user first sees (title, subtitle, any introductory note) should read like a finished work product — plain statements of what the register covers and its source (site/subject, principal reference, decision date, amendments applied, how many conditions were reviewed for discharge) — not a restatement of this skill's internal mechanics. In particular, the `scope_note` field belongs in the JSON as the authoritative record of what was and wasn't verified, but do not paste its full text verbatim onto the tracker's landing page: summarise it in one or two short, plain-English sentences a solicitor or client could read at a glance (e.g. what level was achieved and the one or two most material gaps), and put the complete methodological detail — the full family index, any excluded-family notes, the per-condition discrepancy notes, the discharge-assessment rationale — behind a click (a collapsible "About this register" section, or inside the row-level detail) rather than in the page header. A cluttered landing view undermines the same goal as a wrongly-defaulted filter: it makes the single most important question ("what does this tell me, right now") harder to answer at a glance.

5. **Status must be derived from the Stage 4 assessment, not hand-entered or re-guessed at render time** — write the derivation as a single named function in `CONFIG` that simply reads each condition's stored `discharge_status`. At Levels 1–2 (no discharge review performed), this function should return a fixed "not assessed" status for every condition rather than fabricating a status from absence of data. **Internal status keys (`not_assessed`, `no_discharge_data`, `pending_discharge`, `partially_discharged`, `discharged`) are for the data model only — do not render them verbatim as user-facing labels.** `no_discharge_data` in particular must be rendered with reporting-neutral wording (e.g. "No discharge data found") and not as a claim about construction progress or a statement that discharge is unnecessary — add a one-line caption near the status legend clarifying that absence of a discharge record is not confirmation that no discharge is required. For `partially_discharged`, surface the `discharge_rationale` prominently in the row's expanded detail — a partial-discharge finding is only useful to a reader if they can immediately see which sub-requirements remain outstanding, not just the label. Apply the same discipline to any other status label that could be misread as a substantive finding rather than a report of what was found and assessed in the sources reviewed.
6. **Filter design, scope-inclusive by default:**
   - Free-text search across the fields that matter (subject, heading, full text, ID).
   - Independent dropdowns per taxonomy axis (category, scope, stage, status).
   - A colour-coded legend that doubles as a one-click filter shortcut.
   - **The scope filter must default to answering "what applies to X" (inclusive of ancestor scopes), not "what applies only to X."** Implement scope matching as one small pure function, `scopeMatches(conditionScopes, selectedScope, exclusiveOnly)`, where the default (`exclusiveOnly = false`) matches a condition if the selected scope is in its scope list **or** the *condition's own scope* appears in `scopeAncestors[selectedScope]` — i.e. you walk up from the **selected** scope to see what it descends from, not up from the condition. For example, with a hierarchy of `site_wide` > `phase_1` > `unit_1a`, selecting `unit_1a` must also return every `phase_1` and `site_wide` condition, because `phase_1` and `site_wide` are `unit_1a`'s ancestors. This direction is easy to invert by accident because both phrasings ("X is an ancestor of Y" vs "Y is an ancestor of X") sound similar; after implementing, sanity-check with the actual scope hierarchy derived for this matter before moving on: select the narrowest scope and confirm every wider (ancestor) condition appears, and confirm a sibling scope's conditions do NOT appear. Provide an explicit toggle — labelled something like "this scope only" — to switch to the narrower, exclusive-match behaviour, rather than the reverse. Getting the default backwards silently produces a materially incomplete answer to the single question this filter exists to answer, so treat the direction of the default as a correctness requirement, not a UX preference. Because inclusive is now the default, this control should always be visible when a scope taxonomy with any hierarchy exists — it does not need to hide itself for scopes with no ancestors, since the toggle in that case is simply inert.
7. **Expand-on-click** via a sibling `<tr class="detail-row">` per row, toggled by a Set of expanded IDs re-rendered on every filter/sort change. Guard the row's click handler so clicks on any interactive control nested inside the row (a source link, a button) do not also toggle the row — check `event.target.closest("[data-stop-row-toggle]")` and bail early.
8. **Validate before every save**: HTML has a DOCTYPE and balanced tags; script tag count is even; no external `src="http`, no `rel="stylesheet"`, no CDN references; the embedded JSON parses and has the expected record count; extract the logic `<script>` block and run it through a JS syntax checker (e.g. `node --check`).

**Optimization:** build the CSS/body/script as separate text fragments assembled by a small script, rather than one giant literal — this makes later patches targeted string replacements against a known fragment rather than a full re-generation.

---

## Stage 9 — Link (or embed) the source documents

Goal: a link at summary level to the condition's *current* source page, and a link on every amendment/discharge entry to *its own* source page, opening in a new tab without triggering a download.

1. Parse each `source` reference already captured in Stages 3 and 4 into a structured `{docId, page}` reference; treat a source with no exact page as `page: null` and fall back to page 1 in the eventual open-document call. Do this as a script pass over the whole JSON, adding `source_ref` to every `amendment_history` entry, `discharge_applications` entry, and a top-level `current_source_ref` per condition (= the `source_ref` of the *last* `amendment_history` entry).
2. **Two variants, same template, only if both are requested.** An embedded variant base64-encodes source documents into a JS object (`DOC_REGISTRY`); a linked variant uses the same registry shape holding relative filenames instead, shipped as sibling files. Build both from one shared HTML template with placeholder tokens substituted per variant — do not maintain two hand-diverged copies. If the user has asked for only one variant, build only that one; there is no need to engineer the dual-template abstraction for a single-variant request.
3. **Opening a PDF from a `blob:` source without triggering a download is the one real cross-browser trap.** Navigating a new tab directly to a `blob:` URL for an `application/pdf` Blob renders inline in some Chromium browsers but is treated as a download in others. The robust fix: open the blank tab **synchronously** in the click handler (so it isn't caught by a popup blocker), then `document.write` a minimal same-origin wrapper page into that tab containing `<embed src="[blob url]" type="application/pdf">`. For the linked (real file URL) variant this trap generally does not apply, but consider the same wrapper approach anyway for consistency.
4. Sanitize anything interpolated into a `document.write` call (strip `<`, `>`, `&` from a filename before using it as a wrapper-page `<title>`).
5. Guard summary-level and per-entry link buttons against the row's own click-to-expand handler (Stage 8, point 7).
6. Validate the base64 registry round-trips exactly: decode every entry back to bytes and diff against the original file bytes before saving.

---

## Stage 10 — Save discipline

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

discharge status (Level 3 only; Level 1-2 -> fixed "not_assessed"):
  no discharge_applications entries identified                        -> no_discharge_data
  discharge material read and found to cover every sub-requirement    -> discharged
  discharge material covers some but not all sub-requirements,
    or covers only part of the condition's scope, or is approved
    subject to an outstanding caveat                                  -> partially_discharged
  an application is on record but no decision has been issued yet     -> pending_discharge
  # Every non-"no_discharge_data" classification must carry a discharge_rationale explaining,
  # in plain English, which sub-requirements are covered and which (if any) are not — a bare
  # label without a rationale is not an acceptable Stage 4 output.
```

Re-derive the category list, the flat scope list, and the scope-ancestor map from the actual document set every time — all three are genuinely regime/site-specific, and a fixed list or map from a prior matter should only be a starting hypothesis, never copied wholesale.
