# HTML presentation: templates and examples

Load at the Verify and Deliver steps. Templates fix the shape of the output; examples show common mistakes next to the correct version. All references are synthetic.

## Templates

### T1 · Delivery reply

```markdown
[File name] · [size] MB · [form, e.g. "briefing with conditions register"]
Embedded: [n] documents ([names, or "see Sources in the file"]). The file contains complete copies of them; share it as you would the documents.
Verification: [structure-checked / bytes-verified / browser-tested]. [Not yet checked: what remains, if not browser-tested.]
Open it from your downloads in Chrome, Edge or Firefox on a computer. [Previews inside other apps may not run it; phones open the documents in a new tab.]
[Planning only: built from graph r[n].]
```

No description of the page beyond this. Offer one further view in one line where useful.

### T2 · About this file panel

```markdown
**About this file**
Presents: [subject and purpose]
Source: [conversation answer / data file / matter graph r[n], rules version, validation level]
Built: [date and time]
Documents embedded: [table: title, file name, pages, size, SHA-256 first 12 characters]
Verification: [level reached]; [checks performed]; [checks not performed]
Limitations: [documents not embedded and why; identity matched without checksum; anything the source marks as a gap]
```

### T3 · Citations-only fallback (precondition failed)

```markdown
I can't embed the source documents here: [reason, e.g. "I can't read the files as bytes or run the build step in this environment"].
I can build the page with each citation showing the document, page and quoted text, but without the documents inside it. It will say "Source documents not included" at the top. Shall I go ahead?
```

## Examples

Each pair shows the mistake (✗) and the correct version (✓).

**E1 · Base64 written by the agent**
✗ The agent writes the payload into the page itself, or pastes the start of it with "…"
✓ The template has the `<!-- ATTACHMENTS -->` marker; the build script injects the bytes and verifies them from the saved file

**E2 · Link instead of embedding**
✗ `<a href="sources/notice.pdf">View notice</a>` or `<iframe src="file:///…/notice.pdf">`
✓ `<button type="button" data-attachment-view="doc-1" data-page="4">View</button>`, backed by the embedded payload

**E3 · Printed page used for navigation**
✗ Chip "p 3" opening `#page=3`, when printed page 3 is the fourth PDF page
✓ `"page": 4, "label": "p 3"`; the card reads "p 3 (PDF page 4)"

**E4 · Hover-only evidence**
✗ The quote appears in a tooltip on mouse-over
✓ A chip button opens the citation card by click or keyboard; the chip text is the citation, so it prints

**E5 · Verification overstated**
✗ "Fully tested and working" after the build script alone
✓ "Bytes-verified. Not yet checked: opening each document in a browser and at phone width."

**E6 · Source text as markup**
✗ `card.innerHTML = cite.quote`
✓ `quote.textContent = cite.quote`; a quote containing `<script>` displays as text

**E7 · Document altered to save space**
✗ The 40 MB scanned search recompressed to 8 MB before embedding
✓ The original embedded unchanged; the reply gives the size

**E8 · Status computed by the page (planning)**
✗ A green "Discharged" badge on condition 7 because a condition decision links to it
✓ "Linked: EX/20/0990/DOC, 'approved in part' (instrument reviewed, p 2)". No discharge status.

**E9 · Empty list shown as a negative (planning)**
✗ "Enforcement notices: None"
✓ "Enforcement notices: none extracted within the CON29 enquiries answered (pp 6–7)"

**E10 · Conflict resolved by the page (planning)**
✗ Decision date shown as the newer of two values
✓ "Decision date: sources conflict: 12/03/2020 (notice p 1); 21/03/2020 (search Q3.4)"

**E11 · Wrong file embedded (planning)**
✗ A notice with the same file name but a different SHA-256 embedded for the cited document
✓ The build script refuses it (`expected_sha256`); the citation shows without View, and the About panel lists the document as not embedded
