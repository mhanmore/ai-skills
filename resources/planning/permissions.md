---
id: planning.permissions
kind: resource_bundle
jurisdiction: England
status: draft
requires: [planning.core]
nodes: [PERM, READ, S73, S96A, RM, COND, ROUTE, FAM]
---

# Planning permissions

Rules for extracting and linking applications, decisions, permissions and their family records. Load with [core](core.md). Rule tags are defined there.

The aim is useful extraction, not legal determination. Where a document departs from what the law appears to allow, record it as issued and flag it (CORE-02).

## PERM · Applications, decisions and permissions

- **PERM-01** `model` A planning-history row is an `application` unless the evidence shows more. Add a `decision` where an outcome is stated. Withdrawn, invalid and pending items stay as applications with no decision.
- **PERM-02** `model` Default permission threshold: create a `permission` only where a decision granting planning permission is evidenced at `instrument_reviewed` (a decision notice or appeal decision letter). Below that level, record the application and a decision with `outcome: granted` and the actual `evidence_level`. A skill contract may allow promotion to a provisional permission; record that promotion as an explicit assertion.
- **PERM-03** `model` Keep authority and decision maker apart. `authority_id` is the LPA in whose area the land lies, as named on the instrument; references are scoped to it, and it drives most interpretation. `decision_maker_id` records who decided: the LPA, an Inspector, the Secretary of State, the Mayor of London or a county council. An appeal reference (APP/…) is an additional reference on the appeal decision. Where land spans authorities, record each.
- **PERM-04** `model` Keep references verbatim as strings, with every alternative and portal reference, preserving punctuation and leading zeroes. A reference is unique only within its authority.
- **PERM-05** `model` Keep descriptions as separate fields. `description_operative` is the wording on the decision notice. Each other source (application form, portal, search) gets its own `description_source` entry, because these often differ and search descriptions are often truncated. Never merge them.
- **PERM-06** `model` Record the site as the instrument's address verbatim (`site_address_verbatim`), with a location description where there is no postal address. Matching a site across instruments is an identity judgement: an address match makes a candidate (REL-02), not an identity.
- **PERM-07** `model` Record dates as labelled (EVID-06): received, validated, committee resolution, decision, issue. A committee resolution is not a decision.
- **PERM-08** `model` Record time-limit wording as printed. Calculated deadlines belong only in derived assessments, never in extraction.

## READ · Reading an instrument

- **READ-01** `practice` Capture the whole notice: description; numbered conditions, each with its reason; informatives kept separate; the approved plans and documents list (number, revision, title, date); and any document incorporated by reference.
- **READ-02** `law` A permission is read as a public document. Its meaning comes from the notice, its conditions and reasons, and the documents it incorporates (*Trump International v Scottish Ministers* [2015] UKSC 74; *Lambeth LBC v SSCLG* [2019] UKSC 33). Flag any reliance on application material that is not incorporated.
- **READ-03** `practice` Hybrid permissions: record each component's full or outline treatment and its reserved matters, using the notice's labels.
- **READ-04** `practice` Some conditions apply without being printed (CUR-02). An unprinted condition is not evidence that the condition is absent.
- **READ-05** `practice` What each instrument type adds beyond a search entry:

| Instrument | Look for |
| --- | --- |
| LPA decision notice | Header (reference, applicant, site address, received/valid dates); operative description; decision wording; conditions with reasons; approved plans list (often in a "plans" condition); informatives (s106, CIL, BNG); signature or issue date |
| Appeal decision letter | Appeal (APP/…) and LPA references; Inspector or Secretary of State; letter date; the "Decision" paragraph ("allowed and planning permission is granted for … subject to the conditions in the attached schedule"); the conditions schedule at the end; any costs decision, recorded separately |
| Reserved-matters notice | Matters approved, phase or component, approved plans, any further conditions, the outline reference |
| Condition decision (often a letter) | Condition number(s) and parent reference; submitted documents; decision wording ("approved", "discharged in part", "the details are acceptable"); what remains outstanding; continuing duties |
| s96A letter or notice | Target reference; the exact change; replacement plans; amended or new conditions; "read in conjunction with" wording |

- **READ-06** `practice` Officer and committee reports, application forms and portal pages are context, not instruments. Use them to find candidates and references, never as a source of operative content (READ-02).

## S73 · Section 73 and section 73A

- **S73-01** `law` A grant under s73 is a new permission; the earlier permission continues to exist. `model`: create a new permission node, link it with `varies_under_s73`, and never mutate the earlier node.
- **S73-02** `model` Conditions printed on the new instrument belong to it. Record which conditions the decision says it varies or removes. Where the new instrument cross-refers to earlier approvals, record that reference; otherwise do not carry approvals forward.
- **S73-03** `law` → flag. Section 73 concerns conditions. It cannot change the description of development (*Finney v Welsh Ministers* [2019] EWCA Civ 1868), extend the commencement time limit (s73(5)), or be granted once the earlier permission has expired (s73(4)). Some instruments still attempt these things: record them as issued and raise a review flag, without rejecting or correcting.
- **S73-04** `practice` Planning obligations do not automatically attach to a s73 permission. Record any deed of variation or linking clause; otherwise flag that the obligation linkage is not evidenced.
- **S73-05** `law` Section 73A grants retrospective permission, including for development carried out without complying with a condition. Create a permission node with `procedure: s73a`.

## S96A · Non-material amendments

- **S96A-01** `law` Section 96A changes an existing permission; it does not issue a new one. `model`: create a decision node with `procedure: s96a`, linked by `amends_under_s96a` to the stated target. Create no permission node, even where the portal shows a separate reference.
- **S96A-02** `law` / `practice` Changes can impose, remove or alter conditions (s96A(3)). In practice they sometimes alter the description. Record the exact changes and their scope.
- **S96A-03** `model` The amended permission is a derived view: the target plus every s96A decision, in decision order.
- **S96A-04** `practice` Non-material amendments are also made against s73 permissions and against details approved under reserved matters or conditions. Record the target as stated.

## RM · Reserved matters

- **RM-01** `law` The reserved matters are access, appearance, landscaping, layout and scale (DMPO 2015 art 2). Record each matter addressed; use `other` or `unknown`, with the source label, for anything else.
- **RM-02** `model` Create a decision node linked by `reserved_matters_under` to the outline permission. Record the phase or component and whether all or only some matters are covered.
- **RM-03** `practice` Several or alternative approvals for the same matter or phase are normal. Keep them all; do not select one.

## COND · Conditions and approvals

- **COND-01** `model` Give each condition a stable `condition_id`; `condition_number` is local to its owning permission. Map renumbering, splitting or deletion across instruments only on evidence.
- **COND-02** `model` A condition approval is a decision with `procedure: condition_approval`, linked by `approves_details_under_condition` to the condition ID(s). Record the submitted material, decision wording, scope, partial approval, caveats and any continuing duties.
- **COND-03** `model` Keep four questions separate: discharge as reported by a source; the formal decision; documentary coverage; physical compliance.
- **COND-04** `practice` Record each condition's trigger as worded: before commencement, before occupation, compliance or retention, or phase-specific. Flag any pre-commencement condition with no evidenced approval, without concluding on implementation.
- **COND-05** `model` Never propagate an approval to a parallel permission or a similarly numbered condition without evidence.
- **COND-06** `model` Create `condition` nodes only from a reviewed instrument. Record the number as printed (keep any numbering defects), the group heading, the verbatim text and reason, and the trigger wording (COND-04). A citation of a condition whose owning instrument has not been reviewed becomes an `unresolved_reference` naming the permission and the number.

## ROUTE · Other authorisation routes

These are recorded when encountered, not sought (CORE-03).

- **ROUTE-01** `model` Prior approvals, certificates of lawfulness, permission in principle or technical details consent, listed building consent, conservation area consent, advertisement consent and tree works are applications and decisions with the matching `procedure`. They create no permission node.
- **ROUTE-02** `model` A document may cite permitted development, a local or neighbourhood development order, a development consent order, deemed permission or immunity as the basis for development. Record the citation verbatim as the reported basis and flag it; investigate only if the task asks.

## FAM · Family traversal

- **FAM-01** `model` Start from the task's subject record or from an evidenced permission, not from every history row.
- **FAM-02** `model` Follow typed edges in both directions. Keep every branch and every overlapping family (REL-04).
- **FAM-03** `model` Match references within an authority. An address match alone gives only a candidate.
- **FAM-04** `law` → flag. Overlapping permissions on the same land can be mutually incompatible (*Hillside Parks v Snowdonia NPA* [2022] UKSC 30). Keep all of them and flag the overlap. Never discard a permission or mark it lapsed or superseded on that basis.
- **FAM-05** `model` A "current consolidated permission" is a dated, derived assessment over a declared path and a declared evidence set.
- **FAM-06** `model` Branch model:
  - The family is the root permission and everything descending from it.
  - Each `s73` permission is a **branch head**: a fresh permission on its own terms (S73-01), with its own conditions and its own children.
  - An `s96a` decision **modifies** its target and stays on that branch (S96A-01).
  - Reserved-matters and condition decisions attach to the permission they are stated to be made under.
  - An s73 of an s73 branches from its stated parent, not from the root.
- **FAM-07** `model` Finding the root. From any entry point, walk stated parent links upward until reaching a permission with no stated s73 parent; that is the root. Record the entry point. If the walk ends at an unresolved reference, the root is provisional.
- **FAM-08** `practice` Parent links are usually stated in the description or the decision wording:

| Wording (examples) | Relationship | Child `procedure` |
| --- | --- | --- |
| "Variation/removal of condition(s) N of (planning permission) REF", "Section 73 application…", "Minor material amendment to REF" | `varies_under_s73` | `s73` |
| "Non-material amendment to REF", "NMA to REF" | `amends_under_s96a` | `s96a` |
| "Approval of reserved matters (access, …) pursuant to/following outline (permission) REF" | `reserved_matters_under` | `reserved_matters` |
| "Approval/discharge of details reserved by/pursuant to/required by condition(s) N of REF" | `approves_details_under_condition` | `condition_approval` |
| "Appeal against refusal of REF"; appeal letter quoting the LPA reference | `appeal_of` | — |
| "… without complying with condition N of REF" (retrospective) | `varies_under_s73` | `s73a` |

- **FAM-09** `practice` After an s73 grant, later applications sometimes still cite the earlier reference. Record the target as stated. Where both branches are live, flag `branch_target_ambiguous`, and never copy the decision across (COND-05).
- **FAM-10** `model` Within a branch, order children by labelled decision date, and say which label was used. Apply s96A changes in that order (S96A-03).

## Sources

- [TCPA 1990 s73](https://www.legislation.gov.uk/ukpga/1990/8/section/73), [s73A](https://www.legislation.gov.uk/ukpga/1990/8/section/73A), [s96A](https://www.legislation.gov.uk/ukpga/1990/8/section/96A)
- [DMPO 2015](https://www.legislation.gov.uk/uksi/2015/595/contents)
- [PPG: flexible options for planning permissions](https://www.gov.uk/guidance/flexible-options-for-planning-permissions)
