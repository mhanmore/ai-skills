# Planning resources

Design notes for the planning knowledge base. This file is for maintainers; skills should not load it.

Earlier ad-hoc skills in this repository (for example `planning-permission-condition-consolidation/`) pre-date this library. They keep their own contracts until an adapter is agreed.

## Purpose

A shared knowledge base that lets several assistive planning skills read documents, extract records and link them consistently. It is a set of useful tools, not a comprehensive planning encyclopaedia or an expert system. First use case: turn a local authority search PDF into evidence-traceable JSON, then reuse its planning history for permission-family research, condition consolidation and due diligence.

## Principles

1. **Documents first.** Record what sources say, including irregular content. Legal knowledge produces review flags, never filters or corrections.
2. **Assist, don't determine.** Agents extract, link and flag. Legal conclusions are dated assessments with a declared scope, made only where a skill asks for them.
3. **Work within the task's evidence.** Do not go searching for other bases of authorisation unless asked.
4. **Open graph, closed contracts.** Node and relationship types can grow, and unknown types are preserved without being interpreted. Skills exchange data through declared, versioned contracts.
5. **Explicit jurisdiction.** England only, for now. Wales shares the search forms but not the procedure rules.
6. **Spend tokens on what changes agent behaviour.** That means aliases, positive decision rules and corrections to stale model knowledge, not prose that restates the schema.

## Layout

| Path | Loaded by agents | Content |
| --- | --- | --- |
| `core.md` | Always | Stance, vocabularies, aliases, relationships, evidence rules, recent changes |
| `permissions.md` | Permission, family and condition tasks | Applications, decisions, permissions, s73/s73A, s96A, reserved matters, conditions, traversal |
| `local-authority-searches.md` | Search tasks | Components, response and entry model, LLC, CON29R and CON29O codes, handoff |
| `README.md` | No | This file |

Future work: `schemas/planning/` (JSON Schemas, which will become the source of truth for fields), `contracts/planning/skills/`, `examples/planning/` and `tests/planning/`.

## Conventions

- **Bundles of nodes.** Each file is a bundle of concept nodes (the `##` sections), each with a short prefix (`S73`, `LLC`). Skill loading is capped at around four files, so a task should need no more than three bundles: core plus one or two topic bundles. Split a bundle only if it grows beyond about 2,500 words.
- **Rule IDs.** Rules are written `PREFIX-NN`. Never reuse a retired ID. Skills, contracts and validation reports cite IDs instead of restating rules.
- **Rule tags.** `law` (cite the provision or case), `practice`, `model` or `currency`. Only `law` and `currency` items need legal review when the law changes.
- **Single vocabulary.** All enums and relationship types live in `core.md`. Topic bundles refer to them and never redefine them.
- **Field definitions.** Topic bundles describe only fields with non-obvious meaning. Complete field lists belong in the schemas.
- **No client data.** Use synthetic or redacted fixtures only.

## How skills use this library

A skill declares the bundles it loads, the rule IDs it relies on or overrides (for example, a permission-promotion threshold that differs from PERM-02), the node and relationship types it interprets, preserves or rejects, and its output schema version. Validation reports three levels:

1. **Graph-valid:** envelopes, IDs, links and provenance are sound.
2. **Type-valid:** every node and relationship conforms to its registered type.
3. **Skill-ready:** the package meets the named contract's evidence and coverage rules.

## Settled

- Application, decision and permission are separate nodes. Reserved-matters approvals, condition approvals, s96A amendments and certificates are decisions; only grants of planning permission create permission nodes (VOCAB-01).
- Each classification axis lives on one node type (VOCAB-02).
- Authority (whose area) and decision maker are separate (PERM-03).
- The site is identified by the instrument's address verbatim. Polygon geometry is deferred.
- A permission node requires the reviewed instrument by default (PERM-02).
- Evidence is recorded in an assertion registry, not by wrapping each field. There are no numeric confidence scores.

## Open

1. First consumer output: search summary, chronology, condition tracker or due-diligence issues list?
2. Evidence storage: pointers into documents only, or also cropped page images?
3. Identity resolution: which matches can be made automatically?
4. Review policy: which fields need visual verification?
5. Versioning: schema version, extraction revision and enrichment history.
6. Compatibility: an adapter for the existing condition-consolidation skill, including its `discharge_status` vocabulary.
7. Extension governance: how a provisional type or term becomes registered.

## Ingestion workflow (brief for the future search-ingestion skill)

1. Inventory the documents and pages; identify components (SRCH-02).
2. Assess the text layer and OCR each page as needed; record checksums and extraction method.
3. Read the layout visually where tables, ticks or continuation pages need it.
4. Extract responses and entries verbatim before normalising.
5. Reconcile coverage; record gaps (RESP-06).
6. Map entries to nodes; generate candidates (HAND).
7. Validate at the three levels above.
8. Emit JSON, an exception report and a manifest.

## Fixture shortlist

- Clean, scanned and mixed PDFs.
- Split multi-page history tables.
- Negative, unselected and "see attachment" answers; a missing schedule.
- LLC and CON29 duplicates.
- A CON29DW pack mixed in.
- Hybrid and phased outline permissions with several reserved-matters approvals.
- A s73 chain and a s96A amendment; renumbered conditions.
- A partial condition approval.
- Use classes from before 2020.
- A permission subject to BNG.
- An appeal grant.
- The same reference at different authorities.
- A predecessor authority.
- A Welsh search.
