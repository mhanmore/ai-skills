---
name: planning-permission-condition-consolidation
description: Consolidate the numbered conditions of a planning permission or comparable regulatory approval across amendments and, when evidence is available, assess documentary discharge or compliance coverage. Use for whole-register reconciliation and tracking, not a one-off question about one condition or an instrument with no amendment history.
---

# Planning Permission Condition Consolidation

Build an evidence-traceable canonical record of every condition, its wording history, taxonomy, and the limits of the review. Preserve source language and distinguish formal source outcomes from analytical assessments.

HTML is not an automatic output of this skill. When the user requests HTML or an interactive tracker, also use the standalone `render-response-as-html` skill and pass it the rendering profile in [html-rendering-profile.md](html-rendering-profile.md). For other output formats, use the same canonical data without invoking the HTML renderer.

## Choose the operating level

- **Level 1 — supplied documents:** consolidate the principal instrument and supplied amendments only. Do not imply that the family is complete or assess discharge.
- **Level 2 — supplied documents plus search:** search the available, authorized corpus for missing amendments and family members. Report known-but-unread documents and unresolved gaps. Do not assess discharge unless the underlying material can be read.
- **Level 3 — documentary compliance review:** additionally read the discharge/compliance submissions and decisions, then assess their coverage against each condition's current wording.

Name the achieved level in `scope_note`. Never silently downgrade a review when the necessary material is available. A prior family index may accelerate discovery, but verify its provenance, date, and coverage before relying on it.

## Workflow

1. Build a family index of the principal instrument, every identified amendment, and—at Level 3—every discharge/compliance application. Record references, dates, outcomes, targets, readable/unreadable status, and sources.
2. Extract every numbered condition and reason from the principal instrument verbatim. Preserve numbering defects and document group headings; never silently renumber.
3. Apply amendments in chronological or legally established order. Keep every version in `amendment_history`, with the complete text, a concise change summary, and a page-level source.
4. At Level 3, assess documentary discharge/compliance coverage against the latest amended wording. Follow [discharge-assessment.md](discharge-assessment.md).
5. Assemble and programmatically validate the canonical JSON using [schema.md](schema.md).
6. Derive trigger, category, and hierarchical scope taxonomies from the actual document family. Follow [taxonomy.md](taxonomy.md).
7. Write one short, discriminating subject-matter sentence per condition. Describe the obligation, not its discharge status.
8. Deliver the canonical data and a concise explanation of level, sources, gaps, and material uncertainties. If HTML was requested, render from the canonical data without re-deriving legal or factual judgments in the presentation layer.

## Essential constraints

- Cover every condition, not only amended or discharged conditions.
- Assess the current amended text, never superseded wording.
- Keep the source's formal decision separate from the analytical coverage assessment.
- Describe Level 3 results as a documentary coverage assessment, not an unqualified legal conclusion or a claim about construction progress.
- Absence of a discharge record means only that none was found in the reviewed sources.
- Use scripts for completeness, referential-integrity, enumeration, and closed-set validation; use judgment for substantive summaries and coverage assessments.
- Preserve exact source pointers for current wording, every amendment, and every discharge/compliance record.
