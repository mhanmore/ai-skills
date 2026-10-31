---
label: Permission Family
description: >-
  Use when the user asks to map, update or check the family of an English
  planning permission: identifying the root permission and linking section 73
  permissions, section 96A non-material amendments, reserved matters
  approvals, approvals of details under conditions and appeals. Trigger on
  requests such as "map the permission family for 19/01234/OUT", "update the
  family now the notices are in", "which permission does this NMA belong to?"
  or "what documents are missing from the family?". Works from planning
  records in the matter graph. Use this skill for permission-family mapping,
  not standalone reading of a local search or decision notice: use local-
  search or decision-notice respectively for those tasks. Do not use for
  drafting planning applications or advising whether a permission is
  implemented, lawful or extant.
userFacingDescription: >-
  Maps a planning permission's family (section 73 branches, non-material
  amendments, reserved matters and condition approvals) from the records in
  the matter graph, showing evidence levels, flagging ambiguities and
  listing the documents that would complete the picture.
---

# Permission family

Status: draft · Rules version: 0.3-draft. Record the rules version in the graph manifest (PKG-07).

Typical prompts: "map the permission family for 19/01234/OUT", "update the family now the notices are in", "which family does 21/00456/NMA belong to?", "what's missing from the family?".

Every request goes through the same three stages:
1. **Check the graph is current**, and add candidate links.
2. **Derive the family view** from the root.
3. **Answer, and list the documents that would most improve the family.**

This is an analysis skill: it reads the matter graph and never reads documents. Searches and instruments enter the graph only through @local-search and @decision-notice, which create every node and stated link; this skill calls them for any document not yet read. Each round of documents read raises evidence levels, and re-running this skill redraws the family. It maps structure only. It does not decide which permission is operative, whether development was implemented, or whether conditions are complied with (CORE-04, FAM-05).

## Load

Read these completely before starting:
1. [core.md](../../resources/planning/core.md)
2. [permissions.md](../../resources/planning/permissions.md)
3. [outputs.md](references/outputs.md), the templates and examples, read at the Respond step

## Locate the inputs

1. **Unread documents:** attachments and matter documents not yet in the graph (PKG-07). Call @local-search for each local search and @decision-notice for the planning instruments and context documents, then continue from the revision they write. Leave other documents unread and list them in the reply.
2. **Matter graph** (PKG-01). If there is still none, say so and stop.
3. **Entry point:** a reference, a document, or a description ("the 2019 outline"). If none is given and the graph holds more than one unrelated permission, list the candidate roots and ask.

## Contract

| | |
| --- | --- |
| Input | A matter graph; an entry point |
| Output | A new graph revision adding only candidates and family flags (PKG-03); `family-<root-ref>.json` and `family-<root-ref>.md` beside it (PKG-08); a reply |
| Node types interpreted | `permission`, `condition`, `decision`, `application`, `authority`, `document`, `unresolved_reference`, `search_report` |
| Relationships interpreted | `grants`, `determines`, `varies_under_s73`, `amends_under_s96a`, `reserved_matters_under`, `approves_details_under_condition`, `appeal_of`, `corrects`, `relates_to_obligation`, `reported_as_related_to` |
| Relationships created | `possibly_related_to` only |
| Not produced | Nodes or stated links (@decision-notice, @local-search); consolidated condition wording or discharge status (a planned Permission Conditions skill); conclusions about which permission is operative, implementation or expiry |

## Workflow

### 1. Add candidates (REL-02, FAM-03)

- Generate `possibly_related_to` from a shared reference stem, the same site address, a condition-decision reference such as `/DOC1` or `/DIS` near a grant, or the chronology. Mark each `inferred` with its basis. Match references only within an authority.
- Skip pairs already joined by a stated link, or by an existing candidate with the same basis.
- Never promote a candidate yourself; promotion needs document wording, recorded by @decision-notice.

### 2. Find the root and build branches (FAM-01, FAM-02, FAM-06, FAM-07, FAM-09, FAM-10)

- Walk upward from the entry point to the root. Then collect every node reachable downward through stated links. Hold candidates in a separate tier.
- Each s73 permission is a branch head with its own condition set and children. s96A decisions modify the permission they target. Reserved-matters and condition decisions hang from the permission they cite. An appeal grant hangs from the appeal decision.
- Where a stated link targets an application or an `unresolved_reference` (because the cited permission had not been reviewed when the link was recorded), place the member against the cited record, and follow `data.resolved_node_id` where it is set.
- Flags:
  - `branch_target_ambiguous`: a child cites the pre-s73 reference while a later branch exists (FAM-09).
  - `overlapping_permissions`: two or more branch heads or parallel permissions on the same land (FAM-04). Raise it without concluding.
  - `provisional_root`: the upward walk ended at an unresolved reference.
  - `search_only_member`: a family member known only from a search.

### 3. Work out what to ask for

Rank the missing evidence by how much it would change the family view:
1. An unresolved root or branch-head instrument.
2. A family member known only from a search, especially a grant of planning permission (there is no permission node until its notice is reviewed).
3. Cited conditions whose owning instrument has not been reviewed.
4. s96A or condition decisions whose target has not been reviewed.
5. Candidates that need confirming.

Record `family_search.performed: false`. The harness cannot yet traverse planning portals, so completeness rests on the documents read into the graph. A portal export enters through @decision-notice as a context source (READ-06). Never claim or attempt a portal search.

### 4. Validate

- Every family member is reachable from the root by stated links, or is listed as a candidate.
- Every `permission` in the family has a reviewed granting instrument. If one does not, flag it rather than repair it.
- No condition approval is placed on a different branch from the condition it cites unless a document says so (COND-05).
- Every candidate has an `inferred` assertion with its basis.
- s73 children are drawn as branch heads, never merged into their parent.
- The graph revision, history and family files are written.

## Respond

Use the templates in [outputs.md](references/outputs.md), and check the reply against its examples before sending.

- **Family view (T1):** list the root first; then the branch heads (s73 and s73A permissions), each with its children in date order. Show the operative description where the instrument has been reviewed, otherwise the search description marked `[S]`, shortened with "…" if long. Candidates and unresolved references go in separate sections after the tree.
- **Documents to obtain (T2):** the ranked list from step 3, with the reason for each. Say that, once supplied, they are read into the graph by @decision-notice and the family redrawn.
- **Diagram (T3):** only if asked.
- **Every reply** ends with the footer (T4). Do not say which permission is operative, current or lapsed.

## When the user asks to test this skill

Run the skill on the supplied graph or documents for real. Keep the task output (Part A) separate from a diagnostic report (Part B). Part B covers:
- ambiguous, missing or misleading instructions, and the rule IDs involved;
- graph shapes the branch model (FAM-06) did not anticipate;
- which validation checks were actually performed;
- whether the calls to @local-search or @decision-notice returned a usable graph revision;
- which steps were limited by the environment, such as being unable to save or reload the graph.

Propose concrete instruction changes, but do not change the skill unless asked.

## Flag codes

`branch_target_ambiguous`, `overlapping_permissions`, `provisional_root`, `search_only_member`. Flags raised at ingestion (for example `numbering_defect`, `description_changed_under_s73`) are read from the graph and shown against their members.

## Calibration

This skill is uncalibrated. Graph fixtures to build first (synthetic, under `tests/`), each produced by running @decision-notice or @local-search on synthetic documents:
- an outline permission with phased reserved-matters approvals and condition decisions;
- an s73 chain where a later condition decision cites the original reference;
- an s96A against an s73 permission;
- an appeal grant;
- a search-only family, then the same family after its notices are read, to test candidates and the evidence-level markers.
