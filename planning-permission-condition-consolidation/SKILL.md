---
name: planning-permission-condition-consolidation
description: Consolidate the numbered conditions of a planning permission or comparable regulatory approval across amendments and, when evidence is available, assess documentary discharge or compliance coverage. Use for whole-register reconciliation and tracking, not a one-off question about one condition or an instrument with no amendment history.
---

# Planning Permission Condition Consolidation

Build an evidence-traceable canonical record of every condition, its wording history, taxonomy, and the limits of the review. Preserve source language and distinguish formal source outcomes from analytical assessments.

Read [consolidation-spec.md](consolidation-spec.md) completely before performing the consolidation. HTML is not an automatic output. When the user requests HTML or an interactive tracker, also use the standalone `render-response-as-html` skill and read [html-runtime-instructions.md](html-runtime-instructions.md) completely. For other output formats, use the same canonical data without invoking the HTML renderer.

## Establish the review scope

Do not use or expose tier labels. Determine the work from the user's request, the readable documents, and the authorized sources actually available. Treat these as independent evidence dimensions:

- **Core consolidation:** always identify the principal instrument, extract every condition, and apply every supplied/readable amendment.
- **Family completeness check:** when an authorized searchable corpus is available and the request warrants it, search for missing amendments and family members. Otherwise state plainly that no independent family search was performed.
- **Documentary compliance assessment:** when readable discharge/compliance submissions and decisions are available and within scope, assess their coverage against current condition wording. Otherwise use `not_assessed`; do not infer status from absence.

Record the dimensions in structured `review_scope` fields and write a short `review_summary` in ordinary language. Never ask the user to choose an internal tier, and never show internal scope keys as unexplained labels. A prior family index may accelerate discovery, but verify its provenance, date, and coverage before relying on it.

## Workflow

1. Build a family index of the principal instrument, every identified amendment, and any discharge/compliance applications within the review scope. Record references, dates, outcomes, targets, readable/unreadable status, and sources.
2. Extract every numbered condition and reason from the principal instrument verbatim. Preserve numbering defects and document group headings; never silently renumber.
3. Apply amendments in chronological or legally established order. Keep every version in `amendment_history`, with the complete text, a concise change summary, and a page-specific source.
4. When documentary compliance assessment is in scope, assess coverage against the latest amended wording using the consolidation specification.
5. Assemble and programmatically validate the canonical JSON using the consolidation specification.
6. Derive trigger, category, and hierarchical scope taxonomies from the actual document family using the consolidation specification.
7. Write one short, discriminating subject-matter sentence per condition. Describe the obligation, not its discharge status.
8. Deliver the canonical data and a concise explanation of sources reviewed, searches performed, assessments performed, gaps, and material uncertainties. If HTML was requested, render from the canonical data without re-deriving legal or factual judgments in the presentation layer.

## Essential constraints

- Cover every condition, not only amended or discharged conditions.
- Assess the current amended text, never superseded wording.
- Keep the source's formal decision separate from the analytical coverage assessment.
- Describe compliance results as a documentary coverage assessment, not an unqualified legal conclusion or a claim about construction progress.
- Absence of a discharge record means only that none was found in the reviewed sources.
- Use scripts for completeness, referential-integrity, enumeration, and closed-set validation; use judgment for substantive summaries and coverage assessments.
- Preserve exact source pointers for current wording, every amendment, and every discharge/compliance record.
