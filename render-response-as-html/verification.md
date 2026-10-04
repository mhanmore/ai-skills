# HTML verification

Verify observable behavior, not only source appearance.

## Structural checks

- HTML5 doctype and balanced document structure.
- Embedded JSON parses and expected record counts/identifiers match the validated input.
- JavaScript passes a syntax check.
- No accidental runtime dependency: external script/style source, `@import`, remote CSS `url()`, remote display image/iframe, `fetch`, dynamic import, module import, CDN, or hosted font.
- Intentional reference hyperlinks are clearly identifiable and do not supply required UI resources.
- Every claimed attachment is present in the encoded registry; no attachment resolves to a sibling, sidecar, or local filesystem dependency.
- No placeholder, scaffold, or fallback text remains.

## Behavioral checks

- Open the artifact through `file://` in a browser.
- Exercise every control, useful default state, filter combination, expansion, toggle, link, and reset.
- Confirm derived counts and visible states reconcile to the input data.
- Check the narrowest and widest supported viewport sizes.
- Confirm keyboard access, focus visibility, accessible labels, non-hover access to essential content, and reduced-motion behavior.
- Check long values, empty values, special characters, and user/source text containing markup-like characters.
- Inspect for console errors.
- Review print/PDF output for legibility.
- For SVG/diagrams, check label overlap, clipping, and container resizing.

When domain profiles state invariants, add focused tests for them. A generic “page loads” check is insufficient.
