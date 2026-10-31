# Local search: templates and examples

Load at the Emit and Respond steps. Templates fix the shape of the output; examples show common mistakes next to the correct version. All references and addresses below are synthetic.

## Templates

### T1 · Planning history

```markdown
**Planning history: [property wording as printed]**
Search [report ref], as at [date(s) with labels]. Evidence: search only.

| Reference | Type (source label) | Description (search wording) | Outcome (source label) | Dates (label: value) | Source | Flags |
| --- | --- | --- | --- | --- | --- | --- |
| EX/94/0480/FUL | Full Application | Conversion of garage and single storey extension… | Approved | Decision issued: 26/05/1994; LLC registered: 24/05/1994 | CON29 1.1(a) p3; LLC Part 3(b) p2 | — |
| EX/03/1122/FUL | Full Application | Two storey rear extension | Refused | Decision issued: 14/08/2003 | CON29 1.1(a) p3 | — |

Ordered by: [date label used]. Includes refusals, withdrawals and pending items.
Building control: [n records; list on request].
Coverage stated by the authority: [e.g. "electronic records from 1 August 1977"].
```

### T2 · Search summary

```markdown
**[Component name] ([form/version], replying body, pages)**
- Material entries: [entry, with source page]
- Negative: [codes answered "No"/"None", grouped on one line]
- Qualified or referred: [code, verbatim key phrase, page]
- Gaps: [missing schedules, unreadable pages]
```

Repeat this block for each component, in source order. Boilerplate is listed as page ranges only.

### T3 · Specific question

```markdown
**[Question restated]**
[Code] ([component], p[n]): "[verbatim answer]"
[LLC entry, if relevant: category, description, p[n]]
Reading note: [only rule-based context, e.g. CON29R-03; no conclusions]
```

### T4 · Reply footer (every reply)

```markdown
---
Search as at: [dates with labels] · Evidence: [search only / instruments reviewed: refs]
Coverage limits: [stated limits or "none stated"] · Graph: [location] r[n]
```

### T5 · Exceptions report

```markdown
# Exceptions: search [report ref] (graph r[n])
1. Unreadable or missing: [page / schedule, effect]
2. Count mismatches: [stated vs extracted, component]
3. Flags for review: [code: subject, page, one line]
4. Unmapped questions: [source code/text, page]
Validation level reached: [graph-valid / …]
```

## Examples

Each pair shows the mistake (✗) and the correct version (✓).

**E1 · Part 3 answers read alone (CON29R-03)**
✗ "3.9(m) Tree preservation order: None. There is no TPO affecting the property."
✓ "3.9(m): 'None' (p10). Part 3 answers exclude matters on the LLC register. LLC Part 3(b) lists 'Tree Preservation Order No. 12' (p2)."

**E2 · Date order (EVID-06)**
✗ `"decision_issued": "2012-06-07"` from "06/07/2012"
✓ `"decision_issued": "2012-07-06"`, with the original text "06/07/2012" kept

**E3 · Permission from search evidence (PERM-02)**
✗ A `permission` node for EX/14/3676/FUL created from a CON29 row reading "Approved"
✓ An `application` node, plus a `decision` node with `outcome: granted` and `evidence_level: search_report_only`

**E4 · Dates merged or derived (EVID-06)**
✗ "Decision date: 24/05/1994", taken from the LLC registration date
✓ "LLC registered: 24/05/1994; CON29 decision issued: 26/05/1994". These are two labels, not a conflict.

**E5 · Gap read as a negative (RESP-06)**
✗ 1.1(a) "See attached", schedule missing → "No planning permissions recorded"
✓ 1.1(a) `see_attachment`, with the gap "schedule referred to at 1.1(a) not in pack"

**E6 · Legal conclusion (CORE-04)**
✗ "The extension has planning permission and there is no enforcement risk."
✓ "1.1(a) lists EX/12/2045/FUL, 'Approved' (p3). 3.9(a)–(f): 'No' (p10). Search evidence only."

**E7 · Building control as planning (CON29R-02)**
✗ A history row "EX/17/1396/BN Conservatory extension: Approved"
✓ Listed under building control: "1.1(j) EX/17/1396/BN… ; 1.1(k) completion 06/09/2017"

**E8 · Pre-2016 numbering (CON29R-04)**
✗ An older-form "3.10 No" recorded as "No CIL liability"
✓ Form identified as pre-2016, with 3.10 mapped to conservation area. CIL is `not_searched`.
