# Embedding source documents

Load whenever the presentation includes source documents. This is a fixed build procedure, not a design choice: follow it exactly and do not invent another attachment mechanism.

The page is written with a marker where the documents belong. A build script then reads the original file bytes, injects them, and checks the saved file. The agent never types, pastes or shortens base64 itself.

## 1. Page conventions

- Put exactly one marker, `<!-- ATTACHMENTS -->`, on its own line immediately before the application `<script>`, near the end of `<body>`.
- Give each document a stable id matching `^[a-z0-9][a-z0-9-]{0,63}$`, such as `doc-1` or a graph `document_id` that already matches.
- Put every citation in one JSON block, keyed by citation id. `page` is the 1-based PDF page used for navigation; `label` is what the reader sees, usually the printed page label.

```html
<script id="citations" type="application/json">
{"c1": {"attachment": "doc-1", "page": 4, "label": "p 3", "quote": "The development hereby permitted shall be begun…"}}
</script>
```

- Keep presentation data in its own `<script type="application/json">` blocks. Serialise with a JSON serialiser and replace every `<` with `<` before writing it into the page.
- Never put an attachment path, `file://` URL, sibling file or network fetch in the page, and never a `data:application/pdf` URL.

## 2. Build and check

Write this script to a working folder (not the folder holding the sources), then run it with `python3 -I build_presentation.py spec.json`. It needs only the standard library; it uses `pypdf` for page counts if installed and `node` for a JavaScript syntax check if available, and says when it skipped either.

`spec.json`:

```json
{
  "template": "work/briefing.template.html",
  "output": "briefing.html",
  "attachments": [
    {"id": "doc-1", "path": "sources/notice.pdf", "title": "Decision notice 19/01234/FUL",
     "expected_sha256": "<from the graph, if known>", "pages": 6}
  ]
}
```

`title` and `pages` are optional; `filename` defaults to the source name. Give `expected_sha256` whenever the graph records one, so the script refuses a different file.

```python
#!/usr/bin/env python3
"""Inject source documents into an HTML presentation, then verify the saved file."""
import base64, hashlib, json, re, shutil, subprocess, sys, tempfile
from pathlib import Path

MARKER = '<!-- ATTACHMENTS -->'
SAFE_ID = re.compile(r'^[a-z0-9][a-z0-9-]{0,63}$')
MIME = {'.pdf': 'application/pdf', '.png': 'image/png', '.jpg': 'image/jpeg',
        '.jpeg': 'image/jpeg', '.docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
        '.xlsx': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'}
EXTERNAL = [
    (r'<script\b[^>]*\bsrc\s*=', 'external script'),
    (r'<link\b[^>]*rel\s*=\s*["\']?stylesheet', 'external stylesheet'),
    (r'@import\b', 'CSS @import'),
    (r'url\(\s*["\']?(?:https?:|//|file:)', 'remote CSS url()'),
    (r'<(?:img|iframe|embed|object|video|audio|source)\b[^>]*\bsrc\s*=\s*["\']?(?:https?:|//|file:)', 'remote media'),
    (r'\bfetch\s*\(|XMLHttpRequest|\bimport\s*\(|<script\b[^>]*type\s*=\s*["\']?module', 'runtime fetch or import'),
    (r'file://', 'file:// reference'),
    (r'data:application/pdf', 'PDF data: URL'),
]

def page_count(raw):
    try:
        import io, pypdf
        return len(pypdf.PdfReader(io.BytesIO(raw)).pages)
    except Exception:
        return None

def build(spec):
    template = Path(spec['template']).read_text(encoding='utf-8')
    if template.count(MARKER) != 1:
        sys.exit(f'Template must contain exactly one {MARKER}')
    index, blocks, originals = {}, [], {}
    for a in spec['attachments']:
        if not SAFE_ID.match(a['id']) or a['id'] in index:
            sys.exit(f"Bad or duplicate id: {a['id']!r}")
        path = Path(a['path'])
        raw = path.read_bytes()
        digest = hashlib.sha256(raw).hexdigest()
        if a.get('expected_sha256') and a['expected_sha256'].lower() != digest:
            sys.exit(f"{path}: sha256 {digest} does not match expected {a['expected_sha256']}")
        mime = a.get('mime') or MIME.get(path.suffix.lower(), 'application/octet-stream')
        if mime == 'application/pdf' and not raw.startswith(b'%PDF-'):
            sys.exit(f'{path}: not a PDF signature')
        index[a['id']] = {
            'filename': a.get('filename') or path.name, 'title': a.get('title') or path.stem,
            'mime': mime, 'size': len(raw), 'sha256': digest,
            'pages': a.get('pages') or (page_count(raw) if mime == 'application/pdf' else None),
        }
        blocks.append(f'<script id="attachment-data-{a["id"]}" type="application/octet-stream">'
                      f'{base64.b64encode(raw).decode("ascii")}</script>')
        originals[a['id']] = (path, raw)
    index_json = json.dumps(index, ensure_ascii=False).replace('<', '\\u003c')
    injected = '\n'.join([f'<script id="attachment-index" type="application/json">{index_json}</script>', *blocks])
    out = Path(spec['output'])
    out.write_text(template.replace(MARKER, injected), encoding='utf-8')
    return out, originals

def verify(out, originals):
    text = out.read_text(encoding='utf-8')
    problems, notes = [], []
    m = re.search(r'<script id="attachment-index" type="application/json">(.*?)</script>', text, re.S)
    index = json.loads(m.group(1)) if m else {}
    if set(index) != set(originals):
        problems.append('attachment index does not list exactly the supplied documents')
    for att_id, (path, raw) in originals.items():
        b = re.search(rf'<script id="attachment-data-{re.escape(att_id)}" type="application/octet-stream">([A-Za-z0-9+/=]*)</script>', text)
        decoded = base64.b64decode(b.group(1), validate=True) if b else b''
        rec = index.get(att_id, {})
        if decoded != raw or len(decoded) != rec.get('size') or hashlib.sha256(decoded).hexdigest() != rec.get('sha256'):
            problems.append(f'{att_id}: decoded bytes differ from {path.name}')
        for leak in {str(path.resolve())} | ({str(path)} if '/' in str(path) else set()):
            if leak in text:
                problems.append(f'{att_id}: source path {leak!r} appears in the page')
    page = re.sub(r'<script id="attachment-data-[^"]+" type="application/octet-stream">[A-Za-z0-9+/=]*</script>', '', text)
    for pattern, label in EXTERNAL:
        if re.search(pattern, page, re.I):
            problems.append(f'runtime dependency: {label}')
    if MARKER in page:
        problems.append('marker left in page')
    c = re.search(r'<script id="citations" type="application/json">(.*?)</script>', page, re.S)
    citations = json.loads(c.group(1)) if c else {}
    for cid, cite in citations.items():
        rec = index.get(cite.get('attachment'))
        if rec is None:
            problems.append(f'citation {cid}: unknown attachment {cite.get("attachment")!r}')
        elif not isinstance(cite.get('page'), int) or cite['page'] < 1 or (rec.get('pages') and cite['page'] > rec['pages']):
            problems.append(f'citation {cid}: page {cite.get("page")!r} outside 1..{rec.get("pages") or "?"}')
    for ref in set(re.findall(r'data-attachment-(?:view|download)="([^"]+)"', page)) - set(index):
        problems.append(f'control refers to unknown attachment {ref!r}')
    if any(rec.get('pages') is None for rec in index.values() if rec['mime'] == 'application/pdf'):
        notes.append('page bounds not checked for PDFs without a page count')
    scripts = [s for attrs, s in re.findall(r'<script\b([^>]*)>(.*?)</script>', page, re.S)
               if 'type=' not in attrs or re.search(r'type="(?:text/javascript|module)"', attrs)]
    if shutil.which('node'):
        for i, src in enumerate(scripts):
            with tempfile.NamedTemporaryFile('w', suffix='.js', delete=False) as f:
                f.write(src)
            r = subprocess.run(['node', '--check', f.name], capture_output=True, text=True)
            if r.returncode:
                problems.append(f'script {i + 1} syntax: {r.stderr.strip().splitlines()[-1]}')
    else:
        notes.append('JavaScript syntax not checked (node unavailable)')
    external_links = len(re.findall(r'<a\b[^>]*href\s*=\s*["\']https?:', page, re.I))
    return problems, notes, index, citations, external_links, out.stat().st_size

if __name__ == '__main__':
    spec = json.loads(Path(sys.argv[1]).read_text(encoding='utf-8'))
    out, originals = build(spec)
    problems, notes, index, citations, links, size = verify(out, originals)
    print(json.dumps({'output': str(out), 'size_bytes': size, 'attachments': len(index),
                      'citations': len(citations), 'external_links': links,
                      'problems': problems, 'notes': notes}, indent=2))
    sys.exit(1 if problems else 0)
```

A non-empty `problems` list means the file is not deliverable. Fix the template or spec and rebuild; never edit the injected blocks by hand. The script proves the bytes and structure, not that the controls work in a browser (see [Verification levels](#5-verification-levels)).

Never optimise, recompress, linearise, rasterise or otherwise rewrite a source document unless the user separately asks. Large files make large pages: base64 adds about a third. Report the size; never drop a document or link to it instead.

## 3. Runtime

Paste this script unchanged into the page's application script, before code that renders citations. It decodes a document only when first opened, in chunks so large PDFs survive, and caches one Blob URL per document.

```js
const ATTACHMENTS = JSON.parse(document.getElementById('attachment-index')?.textContent || '{}');
const CITATIONS = JSON.parse(document.getElementById('citations')?.textContent || '{}');
const blobUrls = new Map();

function attachmentRecord(id) {
  if (!Object.hasOwn(ATTACHMENTS, id)) throw new Error(`Unknown document: ${id}`);
  return ATTACHMENTS[id];
}

function attachmentUrl(id) {
  if (blobUrls.has(id)) return blobUrls.get(id);
  const record = attachmentRecord(id);
  const block = document.getElementById(`attachment-data-${id}`);
  if (!block) throw new Error(`Document data missing: ${id}`);
  const b64 = block.textContent.replace(/\s/g, '');
  const chunks = [];
  let size = 0;
  for (let offset = 0; offset < b64.length; offset += 65536) {   // multiple of four
    const binary = atob(b64.slice(offset, offset + 65536));
    const bytes = new Uint8Array(binary.length);
    for (let i = 0; i < binary.length; i += 1) bytes[i] = binary.charCodeAt(i);
    size += bytes.length;
    chunks.push(bytes);
  }
  if (size !== record.size) throw new Error(`Document damaged: expected ${record.size} bytes, found ${size}`);
  const url = URL.createObjectURL(new Blob(chunks, { type: record.mime || 'application/octet-stream' }));
  blobUrls.set(id, url);
  return url;
}

function safeFilename(value) {
  return String(value || 'document').replace(/[\\/:*?"<>|\u0000-\u001f]/g, '_').trim() || 'document';
}

function validPage(record, page) {
  const n = Number.parseInt(page, 10);
  return Number.isInteger(n) && n > 0 && (!record.pages || n <= record.pages) ? n : undefined;
}

function downloadAttachment(id) {
  const a = document.createElement('a');
  a.href = attachmentUrl(id);
  a.download = safeFilename(attachmentRecord(id).filename);
  a.hidden = true;
  document.body.append(a);
  a.click();
  a.remove();
}

function openAttachmentInTab(id, page) {
  const record = attachmentRecord(id);
  const tab = window.open('', '_blank');   // must run inside the click
  if (!tab) throw new Error('The browser blocked the new tab');
  const p = validPage(record, page);
  tab.document.title = safeFilename(record.filename);
  tab.document.body.style.cssText = 'margin:0;height:100vh;overflow:hidden';
  const embed = tab.document.createElement('embed');
  embed.type = record.mime;
  embed.src = attachmentUrl(id) + (p ? `#page=${p}` : '');
  embed.style.cssText = 'display:block;width:100%;height:100%;border:0';
  tab.document.body.replaceChildren(embed);
}

function viewAttachment(id, page) {
  const record = attachmentRecord(id);
  const p = validPage(record, page);
  const dialog = document.getElementById('doc-viewer');
  const holder = dialog.querySelector('[data-viewer-frame]');
  const message = dialog.querySelector('[data-viewer-message]');
  dialog.dataset.attachment = id;
  dialog.dataset.page = p || '';
  dialog.querySelector('[data-viewer-title]').textContent =
    record.title + (p ? ` · PDF page ${p}${record.pages ? ` of ${record.pages}` : ''}` : '');
  if (record.mime === 'application/pdf' && navigator.pdfViewerEnabled === false) {
    holder.replaceChildren();
    message.textContent = 'This browser cannot show PDFs inside the page. Use Open in new tab or Download.';
    message.hidden = false;
  } else {
    // A fresh frame each time: changing only the #page fragment does not move an open PDF.
    const frame = document.createElement('iframe');
    frame.title = dialog.querySelector('[data-viewer-title]').textContent;
    frame.src = attachmentUrl(id) + (p ? `#page=${p}` : '');
    holder.replaceChildren(frame);
    message.hidden = true;
  }
  if (!dialog.open) dialog.showModal();
}

function showCitation(button) {
  const cite = CITATIONS[button.dataset.cite];
  const card = document.getElementById('cite-card');
  if (!cite) return;
  if (!card?.showPopover) return viewAttachment(cite.attachment, cite.page);
  const record = attachmentRecord(cite.attachment);
  card.querySelector('[data-cite-source]').textContent =
    `${record.title} · ${cite.label || `PDF page ${cite.page}`}` + (cite.label ? ` (PDF page ${cite.page})` : '');
  const quote = card.querySelector('[data-cite-quote]');
  quote.textContent = cite.quote || '';
  quote.hidden = !cite.quote;
  card.querySelector('[data-cite-view]').dataset.attachmentView = cite.attachment;
  card.querySelector('[data-cite-view]').dataset.page = cite.page;
  card.querySelector('[data-cite-download]').dataset.attachmentDownload = cite.attachment;
  if (card.matches(':popover-open')) card.hidePopover();
  card.showPopover();
}

// Browsers without the Popover API would show the card inline; they get the viewer directly.
if (!HTMLElement.prototype.showPopover) document.getElementById('cite-card')?.setAttribute('hidden', '');

function reportAttachmentError(error) {
  const output = document.getElementById('attachment-error');
  output.textContent = `Could not open the document: ${error.message}`;
  output.hidden = false;
}

document.addEventListener('click', event => {
  const el = event.target.closest(
    '[data-cite], [data-attachment-view], [data-attachment-download], [data-viewer-newtab], [data-viewer-download], [data-viewer-close]');
  if (!el) return;
  event.preventDefault();
  event.stopPropagation();   // a source control inside an expandable row must not toggle the row
  const viewer = document.getElementById('doc-viewer');
  try {
    if (el.dataset.cite !== undefined) showCitation(el);
    else if (el.dataset.attachmentView) viewAttachment(el.dataset.attachmentView, el.dataset.page);
    else if (el.dataset.attachmentDownload) downloadAttachment(el.dataset.attachmentDownload);
    else if (el.dataset.viewerNewtab !== undefined) openAttachmentInTab(viewer.dataset.attachment, viewer.dataset.page);
    else if (el.dataset.viewerDownload !== undefined) downloadAttachment(viewer.dataset.attachment);
    else if (el.dataset.viewerClose !== undefined) viewer.close();
  } catch (error) {
    reportAttachmentError(error);
  }
});

document.getElementById('doc-viewer')?.addEventListener('close', event => {
  event.currentTarget.querySelector('[data-viewer-frame]').replaceChildren();
});
```

## 4. Required markup

Include these once, outside any element that is hidden, collapsed or printed only. Style them to match the page; keep the data attributes.

```html
<p id="attachment-error" role="alert" hidden></p>

<div id="cite-card" popover class="cite-card" aria-label="Source">
  <p data-cite-source></p>
  <blockquote data-cite-quote></blockquote>
  <button type="button" data-cite-view>View page</button>
  <button type="button" data-cite-download>Download</button>
  <button type="button" popovertarget="cite-card" popovertargetaction="hide">Close</button>
</div>

<dialog id="doc-viewer" class="doc-viewer" aria-labelledby="doc-viewer-title">
  <header>
    <h2 id="doc-viewer-title" data-viewer-title></h2>
    <button type="button" data-viewer-newtab>Open in new tab</button>
    <button type="button" data-viewer-download>Download</button>
    <button type="button" data-viewer-close>Close</button>
  </header>
  <p data-viewer-message role="status" hidden></p>
  <div data-viewer-frame></div>
</dialog>
```

Give the dialog most of the viewport (for example `width: min(96vw, 1100px); height: 92vh`) and the frame `width: 100%; height: 100%; border: 0`. Citation chips are buttons:

```html
<button type="button" class="cite" data-cite="c1">Notice p 3</button>
```

The chip's visible text is the citation, so it still reads correctly when printed or when scripts fail. A Sources section lists every embedded document with its title, size, page count, the first 12 characters of its SHA-256, and View and Download controls (`data-attachment-view`, `data-attachment-download`).

Browser behaviour to expect, so the reply can state it:
- **Desktop Chrome, Edge and Firefox:** the viewer opens at the cited page. The browser's own PDF toolbar names the file with a random id, so point readers to the page's Download control, which keeps the real filename.
- **Safari:** the PDF displays; it may open at page 1. The card still shows the page to go to.
- **Phones and tablets:** most cannot show a PDF inside a page (`navigator.pdfViewerEnabled` is false). The viewer says so, and Open in new tab and Download still work.
- **Previews inside other apps** (email clients, chat previews, document platforms): scripts or PDF display are often disabled. The file works once downloaded and opened in a browser.

## 5. Verification levels

Claim the highest level actually reached, in the reply and in the page's About panel.

1. **Structure-checked:** the build script ran over the saved file with no problems, apart from attachment byte checks.
2. **Bytes-verified:** level 1, and every embedded document decoded from the saved file matches its original byte for byte (the build script does both).
3. **Browser-tested:** level 2, and the saved file was opened from disk in a desktop browser with the console clear; every document was opened from the Sources list and from at least one citation, at the cited page; each Download produced a file whose SHA-256 matches the original; and the page was checked at phone width and in print preview.

Never claim level 3 from reading the source. If a browser is not available, say the file is bytes-verified and list what remains to be checked by hand.
