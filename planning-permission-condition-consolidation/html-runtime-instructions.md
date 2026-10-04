# HTML runtime instructions: conditions register

Pass this profile and the validated canonical JSON to `render-response-as-html` when HTML is explicitly requested.

## Preferred view

A register optimized for scanning conditions, filtering applicability, and opening evidence-backed detail. The renderer may choose the precise layout, but preserve these semantics.

## Required capabilities

- Show condition number, subject summary, trigger, and neutral documentary status in the primary view.
- Search condition number, heading, summary, and full current text.
- Filter independently by category, scope, trigger stage, and status.
- Sort without losing expansion state where practical.
- Expand each condition to show current wording, reason, amendment history, discharge rationale, applications, caveats, and sources.
- Present `review_summary` in ordinary language near the register and keep the structured `review_scope`, family index, and gap detail accessible in an “About this register” disclosure. Never display an internal tier or raw scope keys to users.

## Scope-filter invariant

The default answers “what applies to the selected scope?” and includes conditions assigned to its ancestor scopes. An explicit “this scope only” control switches to literal matching.

```js
function scopeMatches(conditionScopes, selectedScope, exclusiveOnly, scopeAncestors) {
  if (!selectedScope) return true;
  if (conditionScopes.includes(selectedScope)) return true;
  if (exclusiveOnly) return false;
  return (scopeAncestors[selectedScope] || [])
    .some(ancestor => conditionScopes.includes(ancestor));
}
```

Test the narrowest scope: its own, parent, and site-wide conditions must appear; sibling-only conditions must not.

## Status semantics

- Read stored `discharge_status`; do not recalculate substantive assessment in JavaScript.
- Render `not_assessed` as “Not assessed”.
- Render `no_discharge_data` as “No discharge data found”.
- Explain that no record found is not proof that discharge is unnecessary and says nothing about construction progress.
- Make `partially_discharged` rationale immediately available in expanded detail.

Source buttons must not accidentally trigger row expansion. Every source document included with the register must be encoded inside the HTML attachment registry and exposed through separate View PDF and Download PDF controls using the renderer's required Blob mechanism. Source-page controls should pass the cited page to the viewer; downloads must recover the complete original PDF. Do not create sibling or sidecar source files.
