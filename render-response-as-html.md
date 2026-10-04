---
name: org-render-response-as-html
label: Visualise and render interactive output
description: Apply this skill when the user explicitly requests HTML output or expressly @-tags this skill in a prompt.
version: 0.3
---

# Render response as self-contained HTML

When this skill is invoked, produce a polished, fully self-contained HTML document intended to communicate the answer visually and interactively.

The output must work when opened directly using `file://`. It must not require a web server, build step, package manager, CDN, API call, external stylesheet, external JavaScript bundle, font host or other runtime dependency.

CSS, JavaScript, SVG, images, datasets and other display resources must be embedded directly in the HTML.

External hyperlinks may be included as references, but the page's presentation and functionality must not depend upon them.

## Core principle

Do not merely translate a prose response into styled HTML.

Treat the page as a communication design problem. Determine how the information can best be understood using layout, hierarchy, diagrams, visual encoding and interaction.

Prefer the simplest visual mechanism that communicates the information clearly. Interactivity should reveal structure or reduce cognitive load, not exist for decoration.

## Workflow

### 1. DESIGN THE MESSAGE

#### a. Determine the message

Identify:
- the central conclusion or message;
- the information needed to support it;
- secondary detail that should remain available without obscuring the main message;
- comparisons, sequences, relationships, hierarchies, geography, quantities or uncertainty that could be represented visually.

Distil the material without sacrificing accuracy.

#### b. Choose the information architecture

Choose structures appropriate to the content rather than defaulting to prose sections.

Consider, where useful:

- overview → detail layouts;
- timelines and process flows;
- decision trees;
- relationship/network diagrams;
- maps or spatial diagrams;
- comparison matrices;
- annotated schematics;
- dashboards and KPI cards;
- linked charts;
- stepped explainers;
- expandable evidence sections;
- switchable scenarios or perspectives;
- filtering and sorting controls;
- before/after or alternative-state views.

#### c. Choose the interaction model

Consider whether interaction materially improves comprehension.

Useful patterns include:

- segmented selector buttons;
- tabs;
- accordions and disclosure panels;
- hover/focus tooltips;
- clickable diagram regions;
- linked highlighting between text and graphics;
- filters;
- sliders;
- scenario toggles;
- progressive-reveal sequences;
- expandable citations or methodological notes.

All essential information must remain accessible without requiring hover, since the document may be viewed on touch devices.

#### d. Choose the visual implementation

Prefer, in order:

1. semantic HTML and CSS;
2. inline SVG with native JavaScript;
3. Canvas where SVG would be inefficient;
4. an embedded visualisation library only where it substantially reduces complexity or improves the result.

Do not use a library merely because one exists.

For bespoke diagrams and infographic elements, prefer generated inline SVG. SVG should remain part of the DOM where useful so that labels, regions and states can be styled or manipulated interactively.

#### e. Visualisation libraries

Libraries may be used if their complete runtime is embedded in the final HTML and they work under `file://`.

Preferred choices:

- **D3.js** — bespoke data-driven SVG, scales, layouts, transitions and unusual visualisations.
- **Observable Plot** — concise conventional statistical charts.
- **Apache ECharts** — feature-rich interactive charts and dashboard visualisations.
- **Mermaid** — formal flowcharts, sequence diagrams and similar schematic diagrams where its automatic layout provides a clear advantage.

Prefer native SVG/JS for small or highly designed infographic components.

Never load these libraries from a CDN at runtime.

If a library would materially increase file size for something readily implemented natively, do not embed it.

#### f. Visual cues

Use visual hierarchy deliberately.

Consider:

- inline SVG icons;
- restrained use of Unicode symbols or emoji;
- numbered steps;
- colour-coded categories;
- spatial grouping;
- typography and whitespace;
- callouts;
- badges and status indicators.

Do not rely on colour alone to communicate meaning.

#### g. Size budget

Aim for a final HTML file below 10 MB.

Avoid embedding large libraries, images or fonts unnecessarily.

Prefer:
- SVG to raster imagery where appropriate;
- compressed/resized images;
- system font stacks;
- only the data required by the page.

Warn the user when required source material makes the target size impractical.

### 2. BUILD THE PAGE

#### a. Produce one standalone document

Generate a complete HTML5 document containing:

- HTML;
- `<style>` CSS;
- inline or embedded JavaScript;
- inline SVG;
- embedded images/data;
- embedded library code where required.

The artefact should require no installation or configuration.

#### b. Design responsively

The page should work at desktop and mobile widths.

Avoid layouts that depend upon fixed viewport dimensions.

Interactive controls must be usable by mouse, keyboard and touch where practical.

#### c. Progressive enhancement

The principal content should remain understandable even if optional interaction fails.

Do not hide essential conclusions exclusively behind JavaScript-controlled states.

#### d. Accessibility

Use semantic HTML where possible.

Provide:
- meaningful headings;
- accessible control labels;
- keyboard-operable controls;
- sufficient contrast;
- visible focus states;
- textual equivalents for important graphical meaning.

Respect `prefers-reduced-motion` for non-essential animation.

### 3. VERIFY THE ARTEFACT

Before delivery, verify the standalone guarantee.

Search the generated source for accidental external dependencies, including:

- `<script src=...>`
- `<link ... href=...>`
- `@import`
- remote `url(...)`
- `<img src="http...">`
- `<iframe src="http...">`
- `fetch(...)`
- dynamic imports
- module imports that resolve outside the document
- CDN references
- externally hosted fonts.

Reference hyperlinks intentionally shown to the reader are permitted.

Mentally evaluate the document as though opened directly from disk using `file://`.

Check:

- all controls function;
- default states are useful;
- diagrams fit their containers;
- SVG labels do not overlap;
- no text is clipped;
- mobile layout remains usable;
- embedded images render;
- no placeholder or fallback content is exposed;
- print/PDF output is reasonably legible;
- there are no browser-console errors caused by missing resources.

### 4. DELIVER

Deliver the `.html` file as the artefact.

Do not explain routine implementation details unless they are relevant to the user's request.

The finished artefact should feel like a deliberately designed piece of communication, not a web-development demonstration.

## Rules

- The output must be professional, polished and visually coherent.
- Choose colour, typography and layout to suit the subject rather than using a fixed house style.
- Optimise for comprehension before visual novelty.
- Use interaction where it reduces complexity or permits useful exploration.
- Never make presentation functionality depend on an external network resource.
- External links used solely as references are permitted and should be visibly identifiable as external links.
- Do not embed large binary files unless their inclusion materially improves the artefact.
- PDF and other binary attachments may be embedded using data URIs where useful.
- To open an embedded item or reference in another view, prefer a normal `<a target="_blank" rel="noopener">`.
- Avoid `window.open()` except in direct response to a user action.