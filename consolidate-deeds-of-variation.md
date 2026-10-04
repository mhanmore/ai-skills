---
name: deed-of-variation-agreement-consolidation
label: Deed of Variation / Amending-Instrument Agreement Consolidation
description: >-
  Consolidate a base agreement (a contract, deed, or agreement with clauses,
  definitions, and schedules - e.g. a section 106 planning obligations
  agreement, a facility agreement, a lease, a licence, a shareholders'
  agreement) against one or more deeds of variation, amendment deeds, side
  letters, or similar amending instruments that vary it by a delete/insert or
  restate-in-full convention. The primary deliverable is the base
  agreement's own text, reproduced in full and read in its native order,
  with each amendment woven into the running text at the point it occurs
  (not a separate log or explanation of the changes) - flags substantive
  versus housekeeping changes, sweeps for cross-reference and non-textual
  (plan/schedule) impact, and assembles as-amended text with each amending
  instrument's execution status stated plainly rather than gated on it -
  where more than one amending instrument exists, the output supports
  enabling/disabling individual instruments to reconstruct the agreement as
  it stood, or would stand, at a chosen point in time. The correlation
  register is supporting analysis behind that text, not the deliverable
  itself.

  Not specific to UK section 106 agreements or UK law - the same shape
  recurs anywhere a base agreement is varied by a separate amending deed
  using a "delete struck-through words, insert underlined words" or
  "restate this clause in full" convention: loan agreements varied by
  amendment letters, leases varied by deeds of variation, licences varied by
  side letters, articles of association amended by special resolution.

  Do NOT activate for a numbered-conditions regime with a statutory
  amendment/discharge mechanism (e.g. a planning permission's conditions
  varied by non-material amendments and discharged by approval-of-details
  applications) - use the planning-permission-condition-consolidation skill
  for that shape instead. Do NOT activate for a one-off question about a
  single clause's current wording, or for a document with no separate
  amending instrument at all.
---

# Skill: Deed of Variation / Amending-Instrument Agreement Consolidation

## Activation

Activate when the user asks to consolidate, correlate, redline, or reconcile
a base agreement against one or more separate amending instruments (a deed
of variation, amendment deed, side letter, supplemental agreement, or
similar) that vary clauses, definitions, or schedules of that base
agreement. This is distinct from a numbered-conditions regime (see the
planning-permission-condition-consolidation skill): here the unit of
analysis is a **clause or defined term**, not a numbered condition, and
there is usually no separate discharge/compliance mechanism layered on top -
the question is simply "what does the agreement say now, and how do we know."

Signals this skill is the right fit: the amending instrument uses a
delete/insert (strikethrough/underline) convention or restates clauses in
full; the base agreement has a definitions clause and numbered substantive
clauses/schedules; the user wants either a correlation register, a redline,
or a calculated as-amended version.

## What this skill produces

The primary deliverable is **the base agreement's own text, in full,
as amended** - not a report about the base agreement. Everything else
listed below is supporting analysis that makes that text trustworthy, not
a substitute for it. A reader opening the output should be reading the
agreement - every clause, every schedule, in the document's own order and
its own wording - with each amendment appearing as a change to that
wording at the exact point it occurs, the way a redline or a consolidated
copy would show it, not as a separate commentary layer describing what
changed.

1. **A canonical base-agreement JSON** - the full verbatim text of every
   defined term and every substantive clause/schedule of the base
   agreement (not headings or summaries - the actual wording), extracted
   with a completeness check against the document's own structure (see
   Stage 1). This is what makes it possible to reproduce the whole
   agreement later, not only the handful of clauses an amending instrument
   happens to touch.
2. **A correlation register** - for every clause or definition touched by
   an amending instrument, the original text, the proposed/executed text,
   a housekeeping-vs-substantive classification with a stated reason, and
   full citations into both documents. This is the supporting evidence
   for each amendment shown in the text at (4)/(5) below - it is not
   itself the deliverable the user reads to understand the agreement.
3. **A cross-reference impact list** - clauses not directly amended but
   which refer to, or are referred to by, an amended clause/definition,
   flagged as "indirectly affected, not independently verified" unless
   the user asks for that verification to be completed.
4. **Calculated as-amended text, as running agreement prose, for clauses
   that are genuinely text** - the full text of every clause and schedule
   from (1), with each enabled amending instrument's changes applied
   directly into that text (see Stage 6 for the redline/highlight
   convention), so the output reads as the agreement itself, not as a
   list of edits made to it. Non-textual schedule content (plans,
   drawings, inventories) is linked/attached at its position, never forced
   into flowing text. Where more than one amending instrument exists, each
   instrument is presented as an independently togglable layer over the
   base agreement rather than the skill deciding on the user's behalf
   which instruments are "in force" - so the as-amended text can be
   reconstructed for any chosen combination of instruments, e.g. a
   point-in-time view. An instrument's execution status (draft,
   engrossment, executed - and if executed, its date) is never a
   precondition for including it in this view; it is always shown plainly
   against that instrument's toggle so the reader knows exactly what they
   have switched on.
5. **A self-contained HTML rendering of (4)** - the full agreement text,
   reproduced clause by clause and schedule by schedule in the document's
   own structure and order, navigable via a sidebar so a reader can go
   straight to "Schedule 3, paragraph 12" or "Schedule 8" and read the
   actual current wording there, with amendments shown inline in that
   wording (see Stage 7's redline convention) and available to expand for
   the underlying original text, classification, and citations - never a
   flat table or log standing in place of the text itself. See Stage 7 -
   do not default to a generic filterable-table template or to an
   annotation-only view; derive the navigation from the document's real
   structure and render the document's real words.

## Operating levels

**Level 1 - Documents only.** The user supplies the base agreement and all
amending instruments directly. No search of a wider document store. State
in the `scope_note` that no independent check was made for a missing
amending instrument, a superseding version, or a later completion/execution
copy - this is a closed-family, documents-only exercise.

**Level 2 - Documents + search.** As Level 1, plus a searchable document
store exists that may hold further amending instruments, an executed
(rather than draft/engrossment) version, or a related family member (e.g.
a collateral deed, a licence to assign referencing the same base
agreement). Use search to confirm the supplied amending instruments are
complete; report gaps found even if unresolved.

**Level 3 - Documents + search + a live/executed source of truth.** As
Level 2, plus a register, data room index, or completion checklist exists
that confirms which amending instruments have actually been executed
(as opposed to drafts) and in what order. This is the level at which a
default "as the agreement currently stands" view (i.e. only executed
instruments switched on, in execution order) can be asserted with
confidence - at Levels 1-2, an as-amended view can still be produced (see
Stage 6), but state plainly that the amending instrument's
execution/effective status is unconfirmed if the document itself is
undated, unsigned, or marked as a draft/engrossment, and let the reader
choose which instruments to switch on rather than presenting one
combination as "current" by default.

Name the achieved level explicitly in the `scope_note`, exactly as in the
planning-permission-condition-consolidation skill.

---

## Stage 1 - Build the canonical base-agreement JSON, with a completeness gate

Goal: every defined term and every substantive clause/schedule of the base
agreement, extracted once, checked for completeness before any correlation
work starts.

1. Read the base agreement's definitions clause in full, in large
   sequential page-range batches. Do not assume the definitions clause
   starts on a particular page from a prior sample read - read from its
   opening words ("Unless the context otherwise requires... the following
   defined terms...") through to its closing words, inclusive.
2. **Mandatory completeness check, scripted, not eyeballed:** extract every
   quoted defined term (e.g. every `"Xxx" means...` pattern) from the raw
   text and diff that list against the JSON's key list. A definition
   present in the base document's own text but missing from the JSON is a
   Stage 1 defect, not a Stage 3 finding - fix it before moving on. This
   check exists because a single linear read can silently start one page
   too late or too early and produce a false "this term does not appear in
   the base agreement" conclusion downstream.
3. Extract the full verbatim text of every substantive numbered clause
   and schedule the same way as point 1 - not only its heading or title,
   and not only its numbered/lettered sub-paragraphs. Apply the identical
   "opening words through closing words, inclusive" discipline from point
   1 to every clause and schedule: start from the clause or schedule's own
   heading and its first words of running text - including any chapeau,
   preamble, or lead-in sentence that precedes its enumerated sub-items
   (e.g. a clause heading followed by "Unless otherwise agreed..." or
   "The Owner covenants to:" before a lettered list, or a schedule's own
   introductory paragraph before its numbered paragraphs begin) - through
   to its last words, inclusive. **A clause's or schedule's "full text"
   means every sentence a reader would encounter reading that clause or
   schedule from its heading to its end, in order, including connective
   and scene-setting prose that is not itself numbered or lettered** - it
   is not limited to the enumerated sub-paragraphs that happen to carry
   their own numbers or letters. Treat any hand-written summary,
   paraphrase, or "connective sentence" substituted for the document's own
   wording anywhere in this range as a Stage 1 defect on the same footing
   as a missing sub-paragraph - do not assume a lead-in sentence is safe to
   compress or reword because it precedes the "real" numbered content.
   The downstream deliverable is the agreement's actual wording reproduced
   in full (see "What this skill produces"), so a clause or schedule
   recorded only as a heading/title, or with its lead-in prose paraphrased
   even if its numbered sub-items are captured verbatim, is an incomplete
   extraction, not a shortcut - go back and capture its text before
   treating Stage 1 as done. Build the closed, enumerable set of clause
   numbers and schedule numbers (clause 1, 2, 3... schedule 1, 2, 3...) as
   part of this pass, so that a later "clause 47 referenced but not found"
   check is possible, but the set of numbers is a by-product of extracting
   the text, not a substitute for it. For a long base agreement, budget for
   this being the largest single piece of work in the exercise - there is
   no shortcut that preserves a full-text deliverable.
4. Record each defined term and clause as `{id, heading, current_text,
   source}` with `source` a citable pointer (document + page/block), and
   `current_text` holding the clause's actual words, not a paraphrase or a
   "not amended" placeholder.

**Optimization:** run the definition-count and clause-count completeness
checks as a short script immediately after extraction, not as a final
Stage 4 validation - catching a missed page here is far cheaper than
discovering it after correlation and taxonomy work has already been done
against an incomplete base.

---

## Stage 2 - Correlate the amending instrument's own change list

Goal: for every clause/definition the amending instrument itself identifies
as changed, a verified original-versus-proposed pair.

1. Read the amending instrument in full. Identify its own convention for
   marking changes - most commonly a schedule listing "delete the words
   which are struck through; insert the words which are underlined" against
   quoted base-document text, or a "restate clause X in full as follows"
   instruction. Where any part of the amending instrument's own text is
   going to be quoted, summarised, or relied on anywhere in the output
   (its recitals as background for a change, its operative clauses stating
   when the variation takes effect, its own defined terms, or its
   execution block for dating/status purposes), apply the same "opening
   words through closing words, inclusive" verbatim discipline from Stage
   1 to that text - do not paraphrase the instrument's recitals or
   operative wording any more than the base agreement's. This does not
   require extracting boilerplate the output will never surface (e.g. a
   standard execution block's full formal wording, if only the execution
   date and signatory are being reported) - scope the verbatim requirement
   to whatever of the instrument's own text ends up quoted or relied upon,
   but wherever it does, quote it exactly rather than describing it.
2. For each item the amending instrument lists, capture: the clause/
   definition identifier, the instrument's own quoted "before" text, the
   instrument's own "after" text, and source references into both
   documents.
3. **Verify the instrument's own "before" text against the Stage 1
   canonical JSON, not just against itself.** An amending instrument's
   recitation of the clause it is amending is not guaranteed to match the
   base agreement's actual current text - it may already be out of date
   (e.g. quoting an earlier draft, or a superseded prior amendment), or the
   correlation exercise's own extraction of the base agreement may be the
   one at fault. Treat any mismatch as a first-class flag requiring
   resolution, not a data-entry nuisance: state which of the two documents
   the mismatch was found in, quote both versions, and do not silently
   prefer one over the other.
4. Classify each change as **housekeeping** (a plan revision suffix, a
   date, a formatting-only restatement with no wording change, an
   administrative correction) or **substantive** (an amount, a party's
   obligation, a scope of a defined space/area, a condition precedent, a
   release/liability provision, a right to assign or sublet, anything a
   reasonable lawyer would want specifically drawn to their attention).
   Write one sentence justifying the classification for every item, not
   just for the substantive ones - a housekeeping classification is itself
   a claim that needs a stated reason ("plan revision suffix only, no
   wording or figure change") to be checkable.

---

## Stage 3 - Sweep for indirect impact beyond the amending instrument's own list

Goal: catch changes the amending instrument's own change list would not
surface - a definition change altering the meaning of a clause whose own
text is untouched, or a clause renumbering that orphans a cross-reference
elsewhere in the base agreement.

1. For every clause or definition changed in Stage 2, search the full base
   agreement (not just the amending instrument) for every other place that
   clause number or defined term is used. A capitalised defined term is
   textually searchable; do this as a systematic grep-style pass across
   the whole base agreement, not a memory-based guess.
2. For every hit outside the clauses/definitions Stage 2 already covers,
   add an entry to the cross-reference impact list: `{referencing_clause,
   referenced_item, verified: false}`. Do not silently resolve whether the
   cross-reference still makes sense after the change - record it as
   flagged, and only mark `verified: true` if the user has asked for that
   check to be completed and it has been read and confirmed sound.
3. Pay particular attention to any amended clause that changes its own
   internal structure (e.g. unlettered prose restated into lettered
   sub-paragraphs, or a clause split in two) - every cross-reference to
   that clause by number anywhere in the base agreement is now a candidate
   for a broken or ambiguous reference and must appear on the impact list.
4. Note any amendment that touches a defined term used inside another
   *definition* (not just inside a substantive clause) - definition-to-
   definition dependencies are easy to miss because they don't read as
   "clauses" and are commonly skipped by a clause-only sweep.

---

## Stage 4 - Validate and assemble the correlation register

Goal: one JSON file, structurally consistent, with every item traceable and
every known gap stated rather than silently absorbed.

1. Merge Stages 1-3 into: top-level metadata (parties, base agreement
   date, achieved operating level, and - for every amending instrument in
   play, not just one - its date and execution status: draft, engrossment,
   or executed, with the executed date if known), the correlation register
   (every Stage 2 item, tagged with which amending instrument it comes
   from, its housekeeping/substantive classification and justification),
   and the cross-reference impact list from Stage 3. Where more than one
   amending instrument exists, record enough on each correlation item to
   support later toggling by instrument (Stage 6/7) - at minimum an
   `instrument_id` per item and, where two instruments touch the same
   clause, their relative order (by date if both executed, or flagged as
   ambiguous per Stage 6 point 4 if not).
2. Validate programmatically: every correlation item has both an original
   and proposed text field (or an explicit note if one side genuinely does
   not exist, e.g. a brand-new definition being introduced); every
   source reference resolves to a real page/block in the cited document;
   every cross-reference impact item names a real clause that exists in
   the Stage 1 canonical JSON.
3. Write a `scope_note` stating the operating level achieved, the
   execution status of every amending instrument in play, and - critically
   - whether the cross-reference sweep (Stage 3) was completed and, if so,
   whether any flagged items remain unverified. Any as-amended text view
   (Stage 6) must carry this note, and every amending instrument's status
   from it, alongside the view itself - not as a gate on producing the
   view, but as the information the reader needs to judge what they are
   looking at.

---

## Stage 5 - Classify and tag (optional taxonomy layer)

Goal: where the correlation register is large enough to need filtering, tag
each item the same way the planning-permission-condition-consolidation
skill's Stage 5 tags conditions - but the taxonomy here is about the nature
of the change, not a trigger-stage/scope hierarchy, because a deed of
variation has no construction-milestone or spatial-scope structure to
inherit from.

1. Design a change-category set from the actual amending instrument (e.g.
   "financial contribution amount", "plan/drawing reference", "defined
   space/area", "liability/release", "party/assignment", "lease/tenancy
   term", "procedural/administrative") - re-derive this per matter, the
   same discipline as the conditions skill's category taxonomy.
2. Tag every item via a lookup-table script, not by hand-typing dozens of
   JSON records, and validate 100% coverage the same way.

---

## Stage 6 - Calculate as-amended text (as an instrument-togglable view, only where genuinely textual)

Goal: the actual text of the base agreement, read in full, with each
enabled amending instrument's changes shown inline in that text - a
redline or consolidated copy, not a report about the changes. A reader
should be able to read Clause 11 from top to bottom in one place and see
the current wording (including any struck-through/inserted convention
that shows what changed), rather than reading a paraphrase or a badge that
sends them elsewhere to see the actual clause. This applies for any chosen
combination of amending instruments - never gated on any instrument's
execution status, but always presented so the reader can see and control
exactly which instruments are contributing to the text in front of them.

**The deliverable at this stage is the clause's real wording, not a
description of the edit made to it.** For every clause/definition touched
by an enabled instrument, render the current text of that clause exactly
as it would read once the instrument's changes are applied, using a
redline convention within the running sentence - struck-through text for
words the instrument deletes, inserted text visually distinguished (e.g.
underlined) for words the instrument adds - so the paragraph as a whole
still reads as the clause, just marked up, the same way a lawyer would
read a solicitor's redline. Do not replace the clause's own wording with a
label, badge, or one-line summary of what changed ("financial contribution
amount increased") as the primary content - that belongs in expandable
supporting detail (Stage 7), never standing in for the text itself.

**Use colour, not only strikethrough/underline, to separate original text
from changed text at a glance.** Strikethrough and underline alone are
sufficient in monochrome print but are easy to miss when a reader is
scanning a long clause on screen; colour is a second, faster signal
running alongside them, the same convention word processors use for
tracked changes. Unless the base agreement's own document conventions
already assign a specific colour meaning (in which case follow those), use
a consistent, matter-wide scheme: deleted (struck-through) text in one
colour (e.g. a muted red), inserted (underlined) text in a second colour
(e.g. a distinguishable green), and unamended surrounding text left in the
document's normal ink colour, never highlighted - so an amended sentence
reads as mostly ordinary text with the specific words that changed
standing out in colour, not as a wall of colour that is as hard to parse
as no colour at all. Apply this colour scheme consistently at every
amendment across the whole document and state the scheme once, visibly
(e.g. a small legend near the top of the view or document), rather than
leaving the reader to infer what each colour means from context.

**An amending instrument's execution status (draft, engrossment, executed,
undated, unsigned) is never a precondition for calculating or displaying
as-amended text.** A draft or engrossment is frequently exactly what a
user needs to see the effect of - e.g. reviewing a proposed deed of
variation before it is signed. Gating the calculation on execution status
would block that legitimate use. Instead, execution status is surfaced
as data attached to each instrument, prominently and unavoidably, wherever
that instrument's contribution is switched on (see Stage 7) - the
reader's informed choice replaces the skill's gate.

**What remains a genuine quality gate (about the analysis, not the
instrument's status) - do not proceed to calculate a given instrument's
contribution unless:**
- Stage 1's completeness check passed for the base agreement (no missing
  defined terms/clauses to substitute into);
- that instrument's Stage 2 correlation items have been verified per
  Stage 2 point 3 (the instrument's own "before" text checked against the
  Stage 1 canonical JSON, mismatches resolved or flagged);
- Stage 3's cross-reference sweep has at least been run for that
  instrument's changes (it can surface open, unverified flags - the gate
  is that the sweep happened, not that every flag is resolved).

1. For every clause/definition in the Stage 1 canonical JSON, render its
   full current text. Where it is untouched by any currently enabled
   instrument, this is simply the Stage 1 verbatim text, unmarked. Where
   it is touched by a currently enabled instrument, build the redline
   version of that same running text - a word-level or phrase-level
   diff between the Stage 1 original and the Stage 2 proposed text, marked
   with the struck-through/inserted convention - rather than replacing the
   clause with the proposed text alone or with a summary of the change; a
   reader must be able to see both what the clause used to say and what it
   now says without leaving that clause. Treat "which instruments are
   enabled" as a live input the reader controls (Stage 7's toggles), not a
   fixed answer the skill decides once - disabling an instrument reverts
   that clause to its Stage 1 original wording, unmarked.
2. For any amended clause whose Stage 3 cross-reference sweep found other
   clauses referring to it by number or structure, insert an inline
   editorial flag at the point of calculated text (e.g. "[check: clause
   X.Y refers to this clause by its former structure - confirm the
   reference still resolves correctly after restructuring]") rather than
   silently leaving a stale reference uncommented. Recompute which flags
   apply when the enabled set of instruments changes - a flag tied to an
   instrument that is switched off should not appear in that view.
3. **Do not attempt to render non-textual schedule content (plans,
   drawings, inventories, specifications) as text.** Where a schedule is
   substituted by drawings, the as-amended output must say so and link to
   or attach the actual substituted drawings, exactly as they appear in the
   relevant amending instrument - never summarise, describe, or omit them
   as if the schedule were resolved by the surrounding textual changes.
4. Where more than one enabled amending instrument touches the same
   clause, apply them in date/execution order recorded at Stage 4. If
   execution order is ambiguous for two *currently enabled* instruments
   (e.g. two undated drafts touching the same clause), do not guess -
   flag the ambiguity inline at that clause and decline to resolve a
   single text for it until the reader either disables one of the
   conflicting instruments or the order is confirmed; this does not block
   calculating the rest of the document.
5. Present the calculated as-amended text with a header banner stating
   the Stage 4 `scope_note` in short form (operating level, which
   instruments are currently enabled and each one's execution status, and
   whether any cross-reference flags remain open for the enabled set) -
   the same "landing view must not bury material caveats" discipline as
   the planning-permission-condition-consolidation skill's Stage 7 point
   4. This banner must update live as the reader toggles instruments on
   or off (see Stage 7), never left showing a stale combination.

---

## Stage 7 - Build the HTML consolidated view (if requested)

Goal: make the base agreement as varied genuinely **accessible to read**,
not just log every change in a table. HTML output is desirable specifically
because it can present the document the way a reader actually wants to use
it - navigate to a part of the agreement and read it there, amendments
shown in place - rather than because a filterable-table template exists to
be filled in. Treat the planning-permission-condition-consolidation
skill's Stage 7 as a source of reusable *technical* conventions, not as a
template whose table-and-filter shape must be reproduced here. The two
skills produce different documents for different reading tasks: a
conditions tracker is inherently a log of many similar, independent items
best scanned as rows; a deed of variation's output is fundamentally one
document, read in order, with a handful of amendments embedded in it - the
UI should follow that.

1. **Derive the navigation structure from the base agreement itself,
   every time - do not assume it is schedule-based.** Most agreements
   varied by a separate instrument have some clear internal structure
   (clauses and schedules, articles and appendices, sections and annexes,
   parts and exhibits) - inspect the Stage 1 canonical JSON's own headings
   and titles and build the sidebar/contents navigation from whatever that
   structure actually is. Do not force a schedule-based tree onto an
   agreement organised by article, or a clause-based tree onto one
   organised by numbered parts.
2. **Build a full-document reading view that reproduces the agreement's
   actual text throughout, not a changes-only or annotation-only view.**
   Render the full verbatim text of every clause/definition/schedule from
   the Stage 1 canonical JSON, in its native order and structure - every
   section, not only the ones touched by an amending instrument, and not
   as a "not amended, see page N" placeholder. A reader must be able to
   read the whole agreement in this view exactly as they would read the
   base document itself; a placeholder standing in for a clause's real
   words is an incomplete render, not an acceptable shortcut, however long
   the base agreement is. Within that full text:
   - Where a clause/definition is touched by a currently enabled
     instrument, show the **redlined running text** built in Stage 6 -
     the clause's actual wording with struck-through deletions and
     inserted additions marked inline, each also coloured per Stage 6's
     colour scheme (e.g. muted red strikethrough for deletions,
     distinguishable green underline for insertions, unamended text left
     in the document's normal colour) - as the primary content at that
     point in the document. The reader's first and main view of an
     amendment is the amended sentence itself, not a badge, summary, or
     separate log entry standing in for it. Keep this original/changed
     colour scheme visually distinct from the change-category colours
     used on the secondary badge below - a reader must not confuse "this
     text was deleted" with "this is a financial contribution change";
     reserve saturated/bold colour for the redline itself and a
     smaller, muted badge palette for category wayfinding.
   - A small, secondary badge or marker beside the redlined text (colour-
     coded by the Stage 5 change-category tag, e.g. "financial
     contribution amount", "liability/release") signals that a change
     exists and, where more than one amending instrument is in play,
     which instrument it comes from - this is a wayfinding aid layered on
     top of the redlined text, not a replacement for showing that text.
   - Expand/collapse triggered from that badge reveals *supporting*
     detail only: the housekeeping/substantive classification, its stated
     reason, any verification note, and citations into both documents -
     collapsed by default so the base reading experience is not
     overwhelmed, expandable on click. This detail supplements the
     redlined text already visible; it does not carry content (like the
     clause's actual wording) that belongs in the main text itself.
   - Unamended clauses/definitions/schedules render as their full plain
     verbatim text, with no badge and no expand affordance - do not
     manufacture an expand control, a badge, or a placeholder where there
     is nothing to disclose and nothing has changed.
   - Non-textual schedule content (plans, drawings, inventories) is
     represented as a clearly labelled placeholder/link at its position
     in the structure, per Stage 6 point 3 - never rendered as if it were
     flowing text, never silently omitted from the navigation tree. This
     is the one legitimate use of a placeholder in this view - standing in
     for content that genuinely cannot be flowing text, not for text that
     simply has not been extracted yet.
3. **Sidebar navigation plus in-page anchors.** A persistent sidebar
   listing the document's top-level structural units (however Stage 1
   found them - clauses, schedules, articles, parts) lets a reader jump
   directly to a section; within a long section, in-page anchors or a
   secondary-level list (e.g. schedule paragraphs) are appropriate once a
   section is long enough to need them - judge this from the actual
   document, do not add a navigation level that has nothing to organise.
4. **Surface the cross-reference impact list and scope note prominently,
   not buried in a footer.** A dedicated, always-visible panel or banner
   states the Stage 4 `scope_note` (operating level, every amending
   instrument's execution status, cross-reference sweep completeness) and
   links each open cross-reference flag to the point in the document it
   concerns - the same "landing view must not bury material caveats"
   discipline as the planning-permission-condition-consolidation skill's
   Stage 7 point 4 and this skill's Stage 6 point 5.
5. **Give every amending instrument its own enable/disable control, with
   its status shown on the control itself - this is what replaces the
   old execution-status gate.** Where the base agreement has one or more
   amending instruments, render a control per instrument (e.g. a toggle
   or checkbox in the scope panel from point 4, one per instrument) that:
   - Is labelled with the instrument's own identifying details (title,
     date if any) and its execution status (draft / engrossment /
     executed, with the executed date if known) directly on or beside the
     control - a reader must be able to see what a toggle represents and
     whether it is signed and dated without opening anything else.
   - Lets the reader switch that instrument's contribution on or off and
     see the full-document view (Stage 7 point 2) and the calculated
     as-amended text (Stage 6) update live to reflect only the currently
     enabled instruments - this is what makes a genuine point-in-time
     view possible: e.g. switch on only instruments executed before a
     chosen date, or switch on a single draft in isolation to see its
     standalone effect, or switch on all instruments to see the fullest
     currently-available position.
   - Defaults to a sensible starting state stated explicitly in the UI
     (e.g. "showing: all executed instruments, in execution order" or,
     where execution order is unknown, "showing: base agreement only -
     enable instruments below to see their effect") - the default is a
     starting point for the reader to change, never a silent substitute
     for the reader's own judgment about which combination they need.
   - Where two enabled instruments conflict on the same clause (Stage 6
     point 4), shows that conflict inline at the clause rather than
     silently picking one instrument's version - resolvable by the reader
     disabling one of the conflicting instruments.
   - Does not remove or grey out a control for an unexecuted instrument -
     every instrument supplied is togglable regardless of status; status
     is information for the reader's decision, never a reason to disable
     the toggle itself.
6. **Free-text search remains useful even in a document-shaped view** -
   keep it, scoped across the full rendered text (not only the amendment
   log), so a reader can jump to a defined term or figure without knowing
   which structural unit it sits in. A change-category filter and a
   housekeeping/substantive toggle are useful additions that can dim or
   highlight matching amendments within the full-document view, rather
   than replacing the view with a filtered table.
7. **Reuse proven technical conventions from the planning-permission-
   condition-consolidation skill's Stage 7 where they still fit**:
   three-concerns separation (CSS in `<style>`, data as a dedicated
   `<script type="application/json">` tag, a CONFIG-driven rendering
   script), guarded click-to-expand behaviour, and the same pre-save
   validation checklist (balanced tags, no external requests, embedded
   JSON parses, JS syntax checked). Include two distinct legends, not one
   merged legend: a redline legend for the Stage 6 original/changed colour
   scheme (what "struck-through red" and "underlined green" mean), and a
   separate colour-coded legend for the Stage 5 change-category badges -
   keeping them visually and spatially separate reinforces that these are
   two different signals (what changed vs. what kind of change it is), not
   one. Drop only the parts of that convention that presuppose a flat
   table is the right shape for the content - `table-layout: fixed` and
   per-row table filtering apply to a log of similar items, not to a
   rendered document with embedded expandable amendments.
8. **When the base agreement's structure genuinely does not support a
   useful navigation tree** (e.g. a very short agreement with no
   schedules and few clauses), say so and drop the sidebar rather than
   forcing an empty or trivial one - the goal is accessibility, not a
   mandatory UI pattern. Still render the agreement's full redlined text
   as the primary content even without a sidebar; only the navigation
   affordance is optional, the full-text-with-redlines deliverable from
   point 2 is not. A correlation-register table is a fallback for the
   supporting analysis only, never a substitute for showing the agreement's
   actual words. The instrument enable/disable controls from point 5 still
   apply regardless of whether a sidebar is present.

---

## Stage 8 - Save discipline

Same as the planning-permission-condition-consolidation skill's Stage 9:
validate before every `suggest-save`; use `overwrite` for a revision to a
file already saved at that destination; one pending save per destination
at a time; do not assume scratch persists between conversation turns.

---

## Quick-reference: failure modes this skill's gates exist to prevent

These are generic risk patterns, not tied to any one matter or agreement -
each maps to the stage/gate designed to catch it. Treat any of these as a
live risk on every run, not just when they happen to surface:

- **Extraction boundary error**: a linear read of the definitions clause
  starting or ending on the wrong page silently drops a defined term,
  producing a false "this term does not appear in the base agreement" or
  "new term being introduced" conclusion. Stage 1's scripted completeness
  check (diffing every quoted defined term in the raw text against the
  JSON key list) exists specifically to catch this before it propagates.
- **Cosmetic-looking change that is actually substantive**: an edit that
  reads like a drawing revision, date update, or formatting tweak on its
  face can in fact change a defined space's location, an amount, a scope,
  or a party's obligation. Stage 2's requirement to justify every
  classification - including "housekeeping" ones - in one sentence exists
  to force that judgment to be made actively, not assumed from the surface
  form of the edit.
- **Structural change that orphans a cross-reference**: a clause restated
  with a different internal structure (e.g. flowing prose replaced by
  lettered sub-paragraphs, or a clause split in two) can be correlated
  correctly in isolation while breaking or destabilising another clause
  elsewhere in the agreement that refers to it by number or structure.
  Stage 3's mandatory cross-reference sweep exists to catch this before
  any as-amended text is calculated, not after.
- **An unexecuted instrument's effect is either withheld or mistaken for
  the current position**: gating as-amended text on execution status
  blocks the legitimate case of a user wanting to see the effect of a
  draft before signing it, while removing that gate without a clear status
  indicator risks the opposite failure - a reader mistaking a draft's
  calculated text for the agreement's current operative wording. Stage 6's
  removal of the execution-status gate, paired with Stage 7 point 5's
  requirement that every instrument's status is shown directly on its own
  enable/disable control (not just once in a general scope note), exists
  to give the reader the choice and the information together, rather than
  the skill silently deciding for them or silently failing to warn them.
- **A point-in-time toggle combination silently hides an unresolved
  conflict**: when two enabled instruments touch the same clause and their
  relative order is genuinely ambiguous (e.g. two undated drafts), picking
  one silently to produce a clean-looking as-amended text would misstate
  the position. Stage 6 point 4 and Stage 7 point 5's conflict-surfacing
  requirement exist to force that ambiguity to stay visible at the clause
  itself for whatever combination of instruments the reader has currently
  enabled, rather than being resolved by an arbitrary tie-break the reader
  never sees.
- **Non-textual schedule content misread as resolved or absent**: a
  schedule of substituted plans, drawings, inventories, or specifications
  can be misread as empty or as fully resolved by the surrounding textual
  changes when it actually contains real substituted content that a
  text-only pipeline cannot represent. Stage 6 point 3 exists to stop this
  content from being silently dropped, misdescribed, or forced into a
  text-only as-amended draft.
- **A template built for a different document shape gets reused
  uncritically**: a filterable-table tracker is the right reading tool for
  a long list of similar, independent items (e.g. numbered planning
  conditions), but a deed of variation's output is fundamentally one
  document read in order, with a handful of amendments embedded in it -
  forcing the same flat-table UI onto it produces an HTML artefact that is
  technically well-built but hard to actually read the agreement from.
  Stage 7's requirement to derive the navigation structure from the base
  agreement's own structure, and to render the full document (not only
  the changes), exists to stop a proven technical pattern from being
  copied past the point where it still fits the content.
- **The output explains the document instead of being the document**: it
  is easy to build a technically impressive view that describes every
  change accurately - badges, categories, expandable panels, an
  instrument toggle - while never actually reproducing the clause's own
  words as the thing the reader reads. That produces a report about the
  agreement, not a copy of it, and fails the basic purpose of a
  consolidation exercise: giving the reader the agreement's current text.
  This is easiest to fall into with a long base agreement, where fully
  extracting and rendering every clause is more work than summarising the
  handful that changed - watch specifically for "not amended, see page N"
  placeholders creeping back in for untouched sections under time
  pressure; a placeholder is only legitimate for genuinely non-textual
  content (Stage 6 point 3), never as a stand-in for text that simply has
  not been extracted yet, however long the base agreement runs. Stage 1
  point 3's requirement to extract full verbatim text (not headings) for
  every clause and schedule, and Stage 6 and Stage 7 point 2's requirement
  that the redlined clause text itself - not a badge or summary - is the
  primary content at each amendment, exist specifically to keep the
  deliverable being the agreement, with the analysis as support underneath
  it, rather than the other way around.
- **Strikethrough/underline alone under-signals what changed**: a reader
  scanning a long clause on screen can miss a small strikethrough or
  underline mark, especially where the surrounding unamended text is
  visually similar. Relying on typographic marks alone reproduces the
  weakest part of a black-and-white paper redline in a medium (HTML) that
  can do better. Stage 6's colour scheme (distinct colours for deleted and
  inserted text, unamended text left in the document's normal colour) and
  Stage 7 point 2's requirement to keep that colour scheme visually
  distinct from the separate change-category badge colours exist to make
  "what changed" immediately visible without the reader needing to expand
  anything or read closely - and to stop the two different colour signals
  (redline vs. category) from bleeding into one confusing scheme.
- **A clause's own lead-in prose gets silently rewritten while its
  numbered sub-items are extracted faithfully**: a clause or schedule
  heading is very often followed by unnumbered connective or scene-setting
  text before its lettered/numbered sub-items begin (a chapeau such as
  "Unless the context otherwise requires, where in this Deed the following
  defined terms and expressions are used they shall have the following
  respective meanings:", or "The Owner covenants to:" before a lettered
  list). Because this text carries no number or letter of its own, it is
  easy to treat as connective scaffolding safe to paraphrase or summarise
  while still believing the clause has been extracted "in full" because
  every numbered sub-item is intact. This produces exactly the kind of
  silent substitution of the model's own words for the document's own
  words that this skill exists to prevent, and it is easy to miss because
  the numbered content - the part completeness checks naturally focus on -
  is genuinely fine. Stage 1 point 3's express instruction to extract a
  clause's or schedule's full text "from its own opening words... through
  to its closing words, inclusive" - mirroring the same discipline point 1
  already required for the definitions clause's own chapeau - and its
  express statement that a clause's "full text" includes unnumbered
  connective and scene-setting prose, exist specifically to close this
  gap. The same risk and the same discipline apply to an amending
  instrument's own recitals and operative wording wherever they are
  quoted or relied upon (Stage 2 point 1).
