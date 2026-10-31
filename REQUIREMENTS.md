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
- **Skill references:** one skill calls another with `@<skill-id>`, e.g. `@decision-notice`. The id is the skill folder name: the label lowercased and hyphenated (Local Search → `local-search`). Never rename a live skill's label or folder, because other skills call it by that id. Use `@` references in the body only. In the frontmatter, name another skill by its plain id (`decision-notice`), as in "use local-search or decision-notice respectively for those tasks".
- **Tools:** describe actions; don't name internal tools. Use capability chips where a step needs one, e.g. `[@ kind="capability" id="document-reading" label="Document reading"]`. Known ids: `document-reading`, `charts-and-diagrams`.

Repository skills are written directly in this format.

## Upload checklist

1. Import the ZIP from `builds/`, or create the skill by hand: paste `SKILL.md`, then attach every other listed file at the path shown. The upload replaces the live skill with the same label; there is one live instance of each skill and no versioned copies.
2. Leave the files exactly as they are. Don't let a platform skill-builder regenerate or summarise them, because rule IDs (e.g. `PERM-02`) cross-reference between files.
3. When a skill is updated, export it first and copy its `name:` line into the new `SKILL.md`.
4. After a resource changes, rebuild every skill listed against it in the [index by resource](#index-by-resource).
5. Upload every rebuilt skill in the same sitting. Skills call each other by id, so a live mix of old and new skills can disagree about which skill creates which nodes.
6. **Link cross-skill references by hand.** On every upload, once all the skills it calls are live, open the uploaded skill in the platform editor and link the first `@` reference to each skill it calls. Later references to the same skill need no link. The first uses are listed in [Cross-skill links](#cross-skill-links); re-check that table whenever a skill's body changes.

## Cross-skill links

First `@` reference to each referenced skill, in `SKILL.md` order. These are the references to link by hand at upload (checklist step 6).

Calls run one way only: `permission-family` calls `local-search` and `decision-notice`. The document skills reference `permission-family` only to offer it in a reply, never to call it, so no call can loop.

| Skill | Links to | First reference |
| --- | --- | --- |
| `local-search` | `@permission-family` | Workflow step 5 (Build nodes), the "Candidate links" bullet |
| `local-search` | `@decision-notice` | Respond, the "Conditions or operative wording" bullet |
| `decision-notice` | `@permission-family` | Contract table, "Not produced" row |
| `permission-family` | `@local-search` | Introduction, the paragraph beginning "This is an analysis skill" |
| `permission-family` | `@decision-notice` | Introduction, the same paragraph (after `@local-search`) |

The `references/outputs.md` files also mention `@decision-notice` (Permission Family, examples E3 and E6) and `@permission-family` (Decision Notice, template T6). They need no link: once the first reference in `SKILL.md` is linked, the agent has that skill's platform ID in context, and every later `@` reference to it resolves.

## ZIP layout

```text
<skill-folder>/
├── SKILL.md                links rewritten: ../../resources/planning/<file> → references/<file>
└── references/
    ├── outputs.md          copied from skills/<skill>/references/
    └── <each resource listed for the skill>
```

Builds are committed to `builds/` as `<skill-folder>.zip`, one per skill, and rebuilt in place whenever the skill or a resource it lists changes. Git history is the record of earlier builds. Links between resources stay valid because they share a folder. Exception: `core.md`'s pointer to `local-authority-searches.md` dangles in Decision Notice and Permission Family. It's harmless, because neither skill needs the search bundle.

## Local Search (`skills/local-search/`)

Reads a local authority search into the matter graph and answers questions about it.

| Path in skill | Source in repo | Loaded | Rules relied on |
| --- | --- | --- | --- |
| `SKILL.md` | `skills/local-search/SKILL.md` | Always | |
| `references/outputs.md` | `skills/local-search/references/outputs.md` | At the Emit and Respond steps | Templates T1–T6, examples E1–E10 |
| `references/core.md` | `resources/planning/core.md` | Always | CORE, VOCAB, REL, EVID, PKG, CUR |
| `references/local-authority-searches.md` | `resources/planning/local-authority-searches.md` | Always | SRCH, RESP, LLC, CON29R, CON29O, HAND |
| `references/permissions.md` | `resources/planning/permissions.md` | Always (planning handoff) | PERM-01/02/05/06, FAM-08 |

## Decision Notice (`skills/decision-notice/`)

Reads planning instruments into the matter graph, creating permissions and conditions, and answers questions about them.

| Path in skill | Source in repo | Loaded | Rules relied on |
| --- | --- | --- | --- |
| `SKILL.md` | `skills/decision-notice/SKILL.md` | Always | |
| `references/outputs.md` | `skills/decision-notice/references/outputs.md` | At the Emit and Respond steps | Templates T1–T6, examples E1–E11 |
| `references/core.md` | `resources/planning/core.md` | Always | CORE, VOCAB, REL, EVID, PKG, CUR |
| `references/permissions.md` | `resources/planning/permissions.md` | Always | PERM, READ, S73, S96A, RM, COND, ROUTE, FAM-08 |

## Permission Family (`skills/permission-family/`)

Maps a permission family from the matter graph, and lists the documents to obtain. Calls `@local-search` and `@decision-notice` for documents not yet read.

| Path in skill | Source in repo | Loaded | Rules relied on |
| --- | --- | --- | --- |
| `SKILL.md` | `skills/permission-family/SKILL.md` | Always | |
| `references/outputs.md` | `skills/permission-family/references/outputs.md` | At the Respond step | Templates T1–T4, examples E1–E6 |
| `references/core.md` | `resources/planning/core.md` | Always | CORE, VOCAB, REL, EVID, PKG, CUR |
| `references/permissions.md` | `resources/planning/permissions.md` | Always | PERM-02, S73-01, S96A-01, COND-05, FAM |

## Index by resource

| Resource | Used by |
| --- | --- |
| `resources/planning/core.md` | Local Search, Decision Notice, Permission Family |
| `resources/planning/permissions.md` | Local Search, Decision Notice, Permission Family |
| `resources/planning/local-authority-searches.md` | Local Search |

## Maintaining this file

- Update this file in the same change as any edit to a skill's **Load** section or its local files.
- Each skill's **Load** list and its table here must match exactly.
- Bump the rules version when any rule is added, retired or changes meaning, and note the date. Change it here and in the "Rules version" line of every `SKILL.md`; skills record it in the matter graph (PKG-07), so a graph built under older rules can be re-extracted. The rules version is a data version only; builds and deployments are not versioned.
- `README.md` files and `archive/` are never uploaded.
