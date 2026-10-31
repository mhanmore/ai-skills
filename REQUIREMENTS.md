# Skill requirements

The files each skill needs, for anyone uploading skills to a platform. In this repository skills link to shared resources so there is one source of truth. Platforms expect each skill to carry its own files, so on upload every listed file is copied into that skill.

Rules version: **0.1-draft** (2026-10-31)

## Upload checklist

1. Create the skill on the platform with the skill's name (the `/name` used to invoke it).
2. Paste or upload `SKILL.md` as the instructions.
3. Attach every other file listed for that skill, keeping the same filenames.
4. In the uploaded `SKILL.md`, change any link of the form `../../resources/planning/<file>.md` to `<file>.md`.
5. Paste the files as they are. Don't let a platform skill-builder regenerate or summarise them: rule IDs (e.g. `PERM-02`) are cross-references between files.
6. After any resource changes, re-upload that resource to every skill listed against it in the [index by resource](#index-by-resource).

## /local-search

Reads a local authority search into the matter graph and answers questions about it.

| File | Source in repo | Loaded | Rules relied on |
| --- | --- | --- | --- |
| `SKILL.md` | `skills/local-search/SKILL.md` | Always | |
| `outputs.md` | `skills/local-search/outputs.md` | At the Emit and Respond steps | Templates T1–T5, examples E1–E8 |
| `core.md` | `resources/planning/core.md` | Always | CORE, VOCAB, REL, EVID, PKG, CUR |
| `local-authority-searches.md` | `resources/planning/local-authority-searches.md` | Always | SRCH, RESP, LLC, CON29R, CON29O, HAND |
| `permissions.md` | `resources/planning/permissions.md` | Always (planning handoff) | PERM-01/02/05/06, FAM-08 |

Five files. This exceeds the four-file load cap seen in an earlier harness. If a platform enforces that cap, `outputs.md` is the file to make optional.

## /permission-family

Maps a permission family from the matter graph and supplied instruments, and lists the documents to obtain.

| File | Source in repo | Loaded | Rules relied on |
| --- | --- | --- | --- |
| `SKILL.md` | `skills/permission-family/SKILL.md` | Always | |
| `outputs.md` | `skills/permission-family/outputs.md` | At the Respond step | Templates T1–T4, examples E1–E8 |
| `core.md` | `resources/planning/core.md` | Always | CORE, VOCAB, REL, EVID, PKG, CUR |
| `permissions.md` | `resources/planning/permissions.md` | Always | PERM, READ, S73, S96A, RM, COND, ROUTE, FAM |

Four files.

## Index by resource

| Resource | Used by |
| --- | --- |
| `resources/planning/core.md` | /local-search, /permission-family |
| `resources/planning/permissions.md` | /local-search, /permission-family |
| `resources/planning/local-authority-searches.md` | /local-search |

## Maintaining this file

- Update this file in the same change as any edit to a skill's **Load** section or its local files.
- Each skill's **Load** list and its table here must match exactly.
- Bump the rules version when any rule is added, retired or changes meaning, and note the date. Change it here and in the "Rules version" line of every `SKILL.md`; skills record it in the matter graph (PKG-07).
- `README.md` files and `archive/` are never uploaded.
