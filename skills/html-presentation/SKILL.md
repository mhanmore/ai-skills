---
label: HTML Presentation
description: >-
  Use when the user asks for information to be presented as a self-contained
  HTML file: a briefing, explainer, interactive report, register, timeline,
  dashboard, walkthrough or "a single file I can send or open offline".
  Trigger on requests such as "present this as HTML", "make an interactive
  briefing from these documents", "turn the planning history into a page I
  can click through" or "build a register with the source PDFs inside". Use
  it to present an answer, data or a matter graph built by another skill,
  embedding every cited source document in the file. Use this skill for
  presentation only: use local-search, decision-notice or permission-family
  to read documents or build the planning graph first. Do not use when the
  user asks for Word, PowerPoint, PDF or Excel output, or for a chart shown
  in the chat reply.
userFacingDescription: >-
  Presents an answer, dataset or matter graph as one engaging, interactive
  HTML file that works offline, with every cited source PDF embedded and
  each citation opening the document at the cited page.
---

# HTML presentation

Status: draft · Planning profile written for rules version 0.3-draft.

Typical prompts: "present the family as an interactive page", "make a briefing on these three leases with the PDFs inside", "turn this answer into something I can send to the client".

The output is one `.html` file that works when opened from disk, with no server, network connection, CDN, hosted font or sidecar file. Every source document it cites is embedded in it, and every citation opens that document at the cited page.

This skill presents; it does not decide. Facts, classifications and judgments come from the conversation, the supplied data or another skill's output, and the page shows them without recomputing or improving them. It calls no other skill, so any skill or prompt can hand work to it.

## Load

1. [embedding.md](references/embedding.md) whenever source documents are included: the build script, runtime and verification levels. Follow it exactly.
2. [planning-views.md](references/planning-views.md) when the content comes from a planning matter graph.
3. [outputs.md](references/outputs.md), the templates and examples, read at the Verify and Deliver steps.

## Contract

| | |
| --- | --- |
| Input | Content to present (an answer in the conversation, structured data, or a matter graph and its derived views); the source documents it cites |
| Output | One `.html` file, saved among the matter's documents (beside the graph for planning views), plus a short reply |
| Documents embedded | Every document cited, by default; the user may add or exclude documents |
| Not produced | New facts, extraction, legal analysis, or changes to the graph or source documents |
| Verification claimed | `structure-checked`, `bytes-verified` or `browser-tested` (embedding.md §5), never higher than reached |

## Precondition: original bytes

Embedding needs a script run over the original file bytes. Before designing anything, confirm you can read the source files as binary and run the build script from embedding.md. If you cannot:
- say so, and do not produce a file that claims to contain the documents;
- offer a version whose citations show document, page and quote without View or Download, labelled on the page "Source documents not included". Build it only if the user agrees.

Never write, paste or abbreviate base64 yourself.

## Workflow

### 1. Settle the brief

Decide from the request, asking only where the answer would change the build:
- **Reader and use:** who opens it and what they must do with it (decide, check, present in a meeting, look something up).
- **Content source:** an answer in this conversation, supplied data, or a matter graph. Use its values as given.
- **Documents:** every document the content cites. Read them with [@ kind="capability" id="document-reading" label="Document reading"] only to confirm page numbers for citations that lack them; never take new facts from them.
- **Confidentiality:** the file carries complete copies of every embedded document. Say so in the reply.

### 2. Choose the form

Pick the form that fits the reader's task, and combine forms only where each earns its place:

| Reader's task | Form |
| --- | --- |
| Understand a position quickly | Briefing: headline findings first, each expanding to its evidence |
| Look up or check items | Register: search, filters, sortable rows that expand to detail |
| Follow events | Timeline or chronology, keyed by labelled dates |
| See structure or lineage | Relationship map (inline SVG), each node opening its record |
| Compare versions or options | Side-by-side or redline view |
| Present to a room | Walkthrough: one section per screen, arrow-key navigation, a contents rail |

Rules that make it engaging without getting in the way:
- Lead with the answer. A reader who stops after the first screen should have the main message.
- Use progressive disclosure: summary visible, evidence one click away, never more than two levels deep.
- Interaction must reduce effort: filters with counts, search that highlights, expand-all and collapse-all, a reset. No decoration that moves.
- Nothing essential behind hover. Every control works by keyboard, focus is visible, and colour is never the only signal.
- Choose type, palette and density for the subject. Professional documents favour restraint: one accent colour, clear hierarchy, generous line height.
- Show uncertainty as the source does: gaps, flags and conflicting values stay visible and labelled.

### 3. Build

- Write the page as a template with presentation data in `<script type="application/json">` blocks, citations in the `citations` block and the `<!-- ATTACHMENTS -->` marker, as embedding.md §1 specifies. Paste its runtime (§3) and required markup (§4) unchanged.
- Render all source text with `textContent` or DOM methods, never `innerHTML`, `document.write` or string-built HTML.
- Inline all CSS, JavaScript and SVG. Use a library only if it is embedded in full and materially improves the result; otherwise use HTML, CSS, SVG and plain JavaScript.
- Include an **About this file** panel: what it presents, the content source (with graph revision if any), when it was built, the embedded documents, the verification level reached and any limitations.
- Add a print stylesheet: everything expanded, controls hidden, citation chips printed as text, sensible page breaks.
- Respect `prefers-reduced-motion`. Lay out for phone and desktop widths.
- Run the build script (embedding.md §2). It writes the final file and checks it.

### 4. Verify

Reach the highest verification level available (embedding.md §5). Also check, at any level:
- every count, total and label on the page reconciles to the content source;
- every filter, toggle, search and reset works, and filters combine correctly;
- long values, empty values and markup-like source text display safely;
- no placeholder or draft text remains.

Fix and rebuild until the build script reports no problems. Record what was and was not checked in the About panel.

### 5. Deliver

Save the file. Reply with template T1 from [outputs.md](references/outputs.md). Do not describe the page at length: the file is the deliverable.

## When the user asks to test this skill

Use this section only when the user explicitly asks to test, evaluate or debug the skill. Build the file for real (Part A), then a separate diagnostic report (Part B) covering:
- whether binary reading, script execution, file saving and a browser were available, and which verification level that allowed;
- the file size and how the platform handled uploading, previewing and downloading it;
- instructions that were ambiguous or missing, and forms or content the workflow did not anticipate;
- for planning views, any graph fields the profile did not cover.

Propose concrete instruction changes, but do not change the skill unless asked.

## Calibration

This skill is uncalibrated on the platform. First checks:
- whether the agent can read matter documents as bytes and run the build script;
- the largest file the platform will save and let users download;
- whether the platform's own preview runs scripts and shows PDFs (expected: not reliably; the downloaded file is what counts);
- a planning family view built from a real graph, with a scanned notice and a search over 10 MB embedded.
