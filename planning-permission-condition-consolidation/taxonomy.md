# Condition taxonomy

Derive the taxonomy after reading the complete condition set. Do not copy labels from an earlier matter.

## Triggers

Represent triggers as an array because one condition may have several milestones. Common patterns include pre-commencement, pre-above-ground, pre-occupation, pre-use, during construction, post-completion, ongoing, event-triggered, and time-limited, but include only patterns supported by the sources.

Each trigger should contain `stage`, applicable `scope`, and an optional qualifier preserving the actual milestone.

## Scope hierarchy

Build both a flat option list and a map from each scope to all broader containing scopes:

```json
{
  "unit_1a": ["phase_1", "site_wide"],
  "phase_1": ["site_wide"],
  "site_wide": []
}
```

This hierarchy is required. It supports both “what applies to this unit?” and “what applies only to this unit?” without conflating them.

## Categories and summaries

Derive categories from recurring substantive topics across the full set. Allow a bespoke category where forcing a singleton into a broad bucket would obscure meaning.

Write an approximately 8–15 word subject summary for each condition. It should distinguish mirrored or scope-varied conditions and reflect the current obligation. Do not fold status into this sentence.

Validate 100% coverage using lookup tables or another reviewable scripted mapping rather than hand-editing records individually.
