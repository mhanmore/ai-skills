---
name: render-response-as-html
description: Create a polished, self-contained HTML artifact from an answer, validated dataset, or domain rendering profile. Use when the user explicitly requests HTML, an interactive report, dashboard, tracker, or browser-openable standalone visualization; do not invoke merely because another skill can optionally produce HTML.
---

# Render Response as HTML

Produce one polished HTML5 artifact that works directly under `file://` without a server, build step, package manager, CDN, API call, external stylesheet, hosted font, or runtime network dependency. Reference hyperlinks may be external citations, but presentation, functionality, and included attachments must not depend on them.

This skill owns presentation construction and verification. A domain skill owns factual extraction, classification, legal/technical interpretation, canonical data, and semantic constraints. When a domain rendering profile is provided, honor it as the interface contract and do not recompute domain judgments in JavaScript.

## Design

1. Identify the central message, essential evidence, secondary detail, uncertainty, and relationships worth visualizing.
2. Choose an information architecture suited to the reading task: document view, register, comparison, timeline, process, dashboard, map, or another defensible structure.
3. Add interaction only where it reduces cognitive load or supports useful exploration. Essential information must not require hover.
4. Prefer semantic HTML/CSS, then inline SVG and native JavaScript. Use Canvas or a fully embedded library only when it materially improves the result.
5. Choose typography, palette, density, and layout for the subject rather than applying a fixed house style. Do not rely on colour alone.

## Build

- Embed CSS, JavaScript, SVG, required display data, and necessary images directly in the file.
- Keep content understandable if optional interaction fails.
- Use semantic headings, labelled controls, keyboard operation, visible focus, sufficient contrast, and textual equivalents for important graphics.
- Respect `prefers-reduced-motion`.
- Support desktop and mobile widths without fixed-viewport assumptions.
- Aim below 10 MB when the artifact has no binary attachments. Embedded files increase size by roughly one third because of base64; report the resulting size when material, but never omit an attachment or create a sidecar to meet the target.
- Keep data and presentation separable: prefer a dedicated `<script type="application/json">` block for structured data and a small configuration/profile object for field mappings and semantic behavior.
- Treat all inserted source text as data, not markup; escape or safely assign it. Never interpolate untrusted text into `innerHTML`, executable script, CSS, or `document.write`.

When the output includes source documents or other attachments, following [embedded-source-documents.md](embedded-source-documents.md) is mandatory. Do not improvise a different attachment mechanism. Every included attachment must be base64-encoded in the HTML attachment registry and exposed through working **View** and **Download** controls backed by decoded `Blob` objects. Never emit an attachment path, `file://` URL, sibling file, sidecar, or network fetch. External hyperlinks remain permissible only as citations to material that is not included as an attachment.

## Verify and deliver

Follow [verification.md](verification.md) before delivery. Deliver the `.html` file rather than substituting a prose description of it. Explain implementation details only when relevant to limitations or requested by the user.
