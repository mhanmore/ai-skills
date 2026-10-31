---
name: permission-family
description: Map the family of an English planning permission. Find or confirm the root permission, then link its s73 branches, s96A amendments, reserved-matters approvals, condition decisions and appeals. Sources are the matter graph (for example from /local-search) and any decision notices, appeal letters or other instruments the user supplies. Reading instruments adds operative descriptions, conditions and confirmed links to the graph, and the skill reports which documents would close the remaining gaps. Use whenever a permission family, amendment history or "what has been approved under…" is asked about.
status: draft
---

# Permission family

Rules version: 0.1-draft. Record it in the graph manifest (PKG-07).

Typical prompts: "map the /permission-family for 19/01234/OUT", "here are the decision notices; update the /permission-family", "which /permission-family does 21/00456/NMA belong to?", "what's missing from the /permission-family?".

Every request goes through the same three stages:
1. **Enrich the matter graph** from any new instruments.
2. **Derive the family view** from the root.
3. **Answer, and list the documents that would most improve the family.**

Each round of user-supplied documents raises evidence levels: search-reported records become reviewed instruments, and candidate links become confirmed ones. This skill maps structure and records what the instruments say. It does not decide which permission is operative, whether development was implemented, or whether conditions are complied with (CORE-04, FAM-05).

## Load

Read these completely before starting:
1. [core.md](../../resources/planning/core.md)
2. [permissions.md](../../resources/planning/permissions.md)
3. [outputs.md](outputs.md), the templates and examples, read at the Respond step

## Locate the inputs

1. **Matter** (PKG-01): the one the user names, or the only one in the workspace; otherwise create `matters/default/`.
2. **New documents:** attachments, named paths and unprocessed files in `matters/<matter_id>/sources/`. Skip any document already in the graph (PKG-07).
3. **Entry point:** a reference, a document, or a description ("the 2019 outline"). If none is given and the graph holds more than one unrelated permission, list the candidate roots and ask.

## Contract

| | |
| --- | --- |
| Input | A matter graph, instrument PDFs, or both; an entry point |
| Output | A new graph revision (PKG-03); `family-<root-ref>.json` and `family-<root-ref>.md` beside it (PKG-08); a reply |
| Node types created or enriched | `permission`, `condition`, `decision`, `application`, `authority`, `document`, `unresolved_reference` |
| Relationships | `grants`, `determines`, `varies_under_s73`, `amends_under_s96a`, `reserved_matters_under`, `approves_details_under_condition`, `appeal_of`, `corrects`, `relates_to_obligation`, `possibly_related_to` |
| Not produced | Consolidated condition wording or discharge status (a future `/permission-conditions` skill); conclusions about which permission is operative, implementation or expiry |

## Workflow

### 1. Classify each new document (READ-05, READ-06)

Possible kinds: LPA decision notice, appeal decision letter, reserved-matters notice, condition decision, s96A letter, planning obligation, plans, or context (officer report, application form, portal printout).

Read the text layer with `pdftotext -layout`; render image-only or stamped pages with `pdftoppm -r 120 -png` and read them visually. Decision notices are often scans.

A single PDF may bundle several instruments, so split them by page range.

### 2. Extract each instrument

Follow READ-01 and READ-05, keeping all wording verbatim:
- reference(s); authority; decision maker and route (PERM-03);
- labelled dates (EVID-06: never infer an issue date);
- the operative description (PERM-05);
- the decision wording and outcome;
- the site address as printed (PERM-06);
- conditions as `condition` nodes (COND-06), each with its reason;
- the approved plans list;
- informatives;
- every cited reference: parent permission, condition numbers, obligations.

For s73 grants, extract the complete condition set of the new instrument (S73-02), including conditions restated unchanged.

### 3. Merge and promote (PKG-05, PKG-06)

- Match on authority plus reference. A reviewed grant of planning permission creates or confirms its `permission` node, linked from its decision by `grants`, at `instrument_reviewed`.
- Keep search assertions. A truncated search description against the operative description is not a conflict, because the labels differ. A decision date that differs under the same label is `conflicting`.

### 4. Link

- **Stated links (FAM-08, REL-05):** use the wording in instruments, and in search descriptions already in the graph. Point condition-level links at the `condition_id` on the stated permission, or at an `unresolved_reference` (permission plus number) if that permission has not been reviewed.
- **Candidates (REL-02):** use a shared reference stem, the same site address, or a condition-decision reference such as `/DOC1` or `/DIS` sitting near a grant. Mark each `inferred` with its basis. Never promote a candidate yourself; promotion needs document wording.
- **Obligations and appeals:** a cited s106 becomes `relates_to_obligation` to an unresolved reference unless the deed is supplied. An appeal letter links by `appeal_of` to the LPA application; the permission it grants hangs from the appeal decision.

### 5. Find the root and build branches (FAM-06, FAM-07, FAM-09, FAM-10)

- Walk upward from the entry point to the root. Then collect every node reachable downward through stated links. Hold candidates in a separate tier.
- Each s73 permission is a branch head with its own condition set and children. s96A decisions modify the permission they target. Reserved-matters and condition decisions hang from the permission they cite.
- Flags:
  - `branch_target_ambiguous`: a child cites the pre-s73 reference while a later branch exists (FAM-09).
  - `overlapping_permissions`: two or more branch heads or parallel permissions on the same land (FAM-04). Raise it without concluding.
  - `provisional_root`: the upward walk ended at an unresolved reference.
  - `search_only_member`: a family member known only from a search.

### 6. Work out what to ask for

Rank the missing evidence by how much it would change the family view:
1. An unresolved root or branch-head instrument.
2. A family member known only from a search, especially a grant of planning permission (there is no permission node until its notice is reviewed).
3. Cited conditions whose owning instrument has not been reviewed.
4. s96A or condition decisions whose target has not been reviewed.
5. Candidates that need confirming.

Record `family_search.performed: false`. The harness cannot yet traverse planning portals, so completeness rests on the documents supplied. A portal export the user provides is treated as a context source (READ-06). Never claim or attempt a portal search.

### 7. Validate

- Every family member is reachable from the root by stated links, or is listed as a candidate.
- Every `permission` node has a reviewed granting instrument.
- No condition approval links to a condition on a different branch unless a document says so (COND-05).
- Every edge has an assertion and an `evidence_level`.
- s73 children never merge into their parent node.
- The graph revision and history are written.

## Respond

Use the templates in [outputs.md](outputs.md), and check the reply against its examples before sending.

- **Family view (T1):** list the root first; then the branch heads (s73 and s73A permissions), each with its children in date order. Descriptions are verbatim, shortened with "…" if long. Candidates and unresolved references go in separate sections after the tree.
- **Documents to obtain (T2):** the ranked list from step 6, with the reason for each.
- **Diagram (T3):** only if asked.
- **Every reply** ends with the footer (T4). Do not say which permission is operative, current or lapsed.

## Flag codes

`branch_target_ambiguous`, `overlapping_permissions`, `provisional_root`, `search_only_member`, `description_changed_under_s73` (S73-03), `time_limit_extended_under_s73` (S73-03), `obligation_linkage_not_evidenced` (S73-04), `unprinted_condition_possible` (READ-04, CUR-02), `conflicting_values`, `numbering_defect`.

## Calibration

This skill is uncalibrated: no real instruments have been tested yet. Fixtures to build first (synthetic or redacted, under `tests/`):
- an outline permission with phased reserved-matters approvals and condition decisions;
- an s73 chain where a later condition decision cites the original reference;
- an s96A against an s73 permission;
- an appeal grant;
- a search-only family later enriched with notices, to test promotion and the append-only history.
