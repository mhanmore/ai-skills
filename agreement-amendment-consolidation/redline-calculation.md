# As-amended text and redline calculation

The primary output is the complete agreement text in native order. For a selected set of instruments:

1. Render every canonical textual unit.
2. Leave untouched text verbatim and unmarked.
3. Apply enabled instruments in established order.
4. For amended text, provide both a clean calculated version and a word/phrase-level redline when requested.
5. Associate editorial cross-reference warnings with the affected text and enabled instrument.

Use deletion and insertion markup plus a second visual cue such as colour in rendered formats; never rely on colour alone. Keep change-category badges visually distinct from redline semantics.

Execution status is not a calculation gate. The real gates are:

- canonical extraction passed completeness validation;
- quoted before text was checked against the correct prior version;
- indirect-impact sweep ran for the instrument;
- application order is established or ambiguity is explicitly contained.

If two enabled instruments conflict and their order cannot be established, do not synthesize a winner. Mark the affected unit unresolved while continuing to calculate unaffected units. Disabling an instrument must remove its changes and its dependent warnings.

Non-textual schedules remain typed attachments/placeholders at their native structural position. Never summarize a replacement drawing as though that produced consolidated textual content. In HTML output, encode every included schedule attachment inside the HTML file.
