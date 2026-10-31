# Skill requirements

The files each skill needs, for anyone uploading skills to a platform. In this repository skills link to shared resources so there is one source of truth. Platforms expect each skill to carry its own files, so on upload every listed file is copied into that skill.

Rules version: **0.3-draft** (2026-10-31)

## Legora format

Confirmed from Legora's own skill files.

- **File:** `SKILL.md` at the root of the skill folder. It must start with `---` and close the YAML header with `---`; Markdown instructions follow.
- **Frontmatter:** exactly these fields for a new skill:
  - `label`: the skill name shown to users. At most 80 characters; quote it if it contains YAML punctuation such as `:`. It must be distinct from existing skills, because a matching label can be treated as an update.
  - `description`: activation instructions (when to use the skill, trigger requests, exclusions). At most 1,024 characters, written as `>-` with the text indented two spaces.
  - `userFacingDescription`: the summary shown in skill lists. About 30–40 words, at most 1,024 characters, written as `>-`.
- **`name`:** never add one. Legora assigns it (e.g. `org-html-response-CIkUA`). When updating an existing skill, keep the `name` from its exported copy exactly as it is.
- **No other fields.** Put status and rules version in the body, not the frontmatter.
- **Supporting files:** use skill-relative paths under `references/`.
- **Tools:** describe actions; don't name internal tools. Use capability chips where a step needs one, e.g. `[@ kind="capability" id="document-reading" label="Document reading"]`. Known ids: `document-reading`, `charts-and-diagrams`.

Repository skills are written directly in this format.

## Upload checklist

1. Import the ZIP from `builds/`, or create the skill by hand: paste `SKILL.md`, then attach every other listed file at the path shown.
2. Leave the files exactly as they are. Don't let a platform skill-builder regenerate or summarise them, because rule IDs (e.g. `PERM-02`) cross-reference between files.
3. When a skill is updated, export it first and copy its `name:` line into the new `SKILL.md`.
4. After a resource changes, rebuild every skill listed against it in the [index by resource](#index-by-resource).

## ZIP layout

```text
<skill-folder>/
├── SKILL.md                links rewritten: ../../resources/planning/<file> → references/<file>
└── references/
    ├── outputs.md          copied from skills/<skill>/references/
    └── <each resource listed for the skill>
```

Builds are committed to `builds/` as `<skill-folder>-v<rules version>.zip`. Never overwrite a build; a new rules version gets a new file. Links between resources stay valid because they share a folder. Exception: `core.md`'s pointer to `local-authority-searches.md` dangles in Permission Family. It's harmless, because that skill never needs the search bundle.

| Build | Rules version | Date | Status |
| --- | --- | --- | --- |
| `builds/local-search-v0.1-draft.zip` | 0.1-draft | 2026-10-31 | Superseded: Legora rejected its frontmatter |
| `builds/permission-family-v0.1-draft.zip` | 0.1-draft | 2026-10-31 | Superseded: Legora rejected its frontmatter |
| `builds/local-search-v0.2-draft.zip` | 0.2-draft | 2026-10-31 | Superseded: efficiency refinements in 0.3 |
| `builds/permission-family-v0.2-draft.zip` | 0.2-draft | 2026-10-31 | Superseded: core.md changed in 0.3 |
| `builds/local-search-v0.3-draft.zip` | 0.3-draft | 2026-10-31 | Current |
| `builds/permission-family-v0.3-draft.zip` | 0.3-draft | 2026-10-31 | Current |

## Local Search (`skills/local-search/`)

Reads a local authority search into the matter graph and answers questions about it.

| Path in skill | Source in repo | Loaded | Rules relied on |
| --- | --- | --- | --- |
| `SKILL.md` | `skills/local-search/SKILL.md` | Always | |
| `references/outputs.md` | `skills/local-search/references/outputs.md` | At the Emit and Respond steps | Templates T1–T6, examples E1–E10 |
| `references/core.md` | `resources/planning/core.md` | Always | CORE, VOCAB, REL, EVID, PKG, CUR |
| `references/local-authority-searches.md` | `resources/planning/local-authority-searches.md` | Always | SRCH, RESP, LLC, CON29R, CON29O, HAND |
| `references/permissions.md` | `resources/planning/permissions.md` | Always (planning handoff) | PERM-01/02/05/06, FAM-08 |

## Permission Family (`skills/permission-family/`)

Maps a permission family from the matter graph and supplied instruments, and lists the documents to obtain.

| Path in skill | Source in repo | Loaded | Rules relied on |
| --- | --- | --- | --- |
| `SKILL.md` | `skills/permission-family/SKILL.md` | Always | |
| `references/outputs.md` | `skills/permission-family/references/outputs.md` | At the Respond step | Templates T1–T4, examples E1–E8 |
| `references/core.md` | `resources/planning/core.md` | Always | CORE, VOCAB, REL, EVID, PKG, CUR |
| `references/permissions.md` | `resources/planning/permissions.md` | Always | PERM, READ, S73, S96A, RM, COND, ROUTE, FAM |

## Index by resource

| Resource | Used by |
| --- | --- |
| `resources/planning/core.md` | Local Search, Permission Family |
| `resources/planning/permissions.md` | Local Search, Permission Family |
| `resources/planning/local-authority-searches.md` | Local Search |

## Maintaining this file

- Update this file in the same change as any edit to a skill's **Load** section or its local files.
- Each skill's **Load** list and its table here must match exactly.
- Bump the rules version when any rule is added, retired or changes meaning, and note the date. Change it here and in the "Rules version" line of every `SKILL.md`; skills record it in the matter graph (PKG-07).
- `README.md` files and `archive/` are never uploaded.
