# Embedded and downloadable source documents

This is a deterministic build requirement, not an optional presentation enhancement. If the HTML claims to include a PDF or other attachment, the original file bytes must be inside the HTML and must be recoverable through a working Download control. Do not offer linked, sibling, sidecar, multi-file, `file://`, or runtime-fetch variants.

## 1. Encode the original bytes

For every attachment:

1. Read the file in binary mode. Never read a PDF as text.
2. Calculate its byte length and SHA-256 digest.
3. Base64-encode the exact bytes with the standard alphabet. Do not URL-encode the base64, insert line breaks, truncate it, or prefix it with `data:`.
4. Do not optimize, rewrite, recompress, linearize, rasterize, or otherwise alter a PDF unless the user separately asks for that transformation.
5. Give it a stable internal ID and a conservative download filename. The filename is metadata; it must not be used as HTML.

A reliable build-time encoding operation is equivalent to:

```python
import base64
import hashlib

raw = attachment_path.read_bytes()
record = {
    "filename": attachment_path.name,
    "mime": "application/pdf",
    "size": len(raw),
    "sha256": hashlib.sha256(raw).hexdigest(),
    "base64": base64.b64encode(raw).decode("ascii"),
}
```

Do not use shell output that may wrap base64 lines unless wrapping is explicitly disabled and later verified. Do not paste a shortened payload from terminal output.

## 2. Put records in the HTML itself

Embed one registry in a non-executable JSON script element. The final HTML must contain the full base64 string, not a placeholder, path, variable that will be filled later, or shortened sample.

```html
<script id="attachment-registry" type="application/json">
{
  "principal-notice": {
    "filename": "principal-notice.pdf",
    "mime": "application/pdf",
    "size": 123456,
    "sha256": "<64 lowercase hexadecimal characters>",
    "base64": "<the complete base64 payload>"
  }
}
</script>
```

Serialize the registry with a JSON serializer. Escape `<` in metadata as `\u003c` before placing the serialized JSON in the script element so a hostile filename cannot form `</script>`. Base64 itself cannot contain `<`.

Parse it locally; do not fetch it:

```js
const ATTACHMENTS = JSON.parse(
  document.getElementById('attachment-registry').textContent
);
```

Do **not** use any of these substitutes:

- `<a href="document.pdf">` or `<embed src="document.pdf">`;
- an absolute/relative filesystem path;
- `fetch()` or XHR to load the attachment;
- a `data:application/pdf;base64,...` URL containing the whole PDF;
- base64 placed in a visible element or HTML attribute;
- a manifest entry whose payload is omitted, abbreviated, or stored elsewhere.

## 3. Decode to Blob without corrupting large files

Use chunked base64 decoding. Avoid a giant numeric array, spreading bytes into a function call, or constructing a data URL; those approaches commonly fail on larger PDFs.

```js
const blobUrlCache = new Map();

function attachmentBlob(record) {
  const b64 = record.base64.replace(/\s/g, '');
  const sliceChars = 4 * 16384; // multiple of four
  const chunks = [];
  let decodedSize = 0;

  for (let offset = 0; offset < b64.length; offset += sliceChars) {
    const binary = atob(b64.slice(offset, offset + sliceChars));
    const bytes = new Uint8Array(binary.length);
    for (let i = 0; i < binary.length; i += 1) {
      bytes[i] = binary.charCodeAt(i);
    }
    decodedSize += bytes.length;
    chunks.push(bytes);
  }

  if (decodedSize !== record.size) {
    throw new Error(`Attachment size mismatch: expected ${record.size}, decoded ${decodedSize}`);
  }
  return new Blob(chunks, { type: record.mime || 'application/octet-stream' });
}

function attachmentUrl(id) {
  if (blobUrlCache.has(id)) return blobUrlCache.get(id);
  const record = ATTACHMENTS[id];
  if (!record) throw new Error(`Unknown attachment: ${id}`);
  const url = URL.createObjectURL(attachmentBlob(record));
  blobUrlCache.set(id, url);
  return url;
}

window.addEventListener('beforeunload', () => {
  for (const url of blobUrlCache.values()) URL.revokeObjectURL(url);
});
```

## 4. Provide a real Download control

Every included attachment needs a visible Download control. Create the Blob URL from the registry, set the anchor's `download` filename, programmatically click it during the user's click event, and remove the temporary anchor.

```js
function safeFilename(value) {
  const cleaned = String(value || 'attachment')
    .replace(/[\\/:*?"<>|\u0000-\u001f]/g, '_')
    .trim();
  return cleaned || 'attachment';
}

function downloadAttachment(id) {
  const record = ATTACHMENTS[id];
  if (!record) throw new Error(`Unknown attachment: ${id}`);
  const a = document.createElement('a');
  a.href = attachmentUrl(id);
  a.download = safeFilename(record.filename);
  a.hidden = true;
  document.body.appendChild(a);
  a.click();
  a.remove();
}
```

Bind it directly to a button; do not put file bytes or filesystem paths in HTML attributes:

```html
<button type="button" data-attachment-download="principal-notice">
  Download PDF
</button>
```

## 5. Provide an inline View control for PDFs

Do not navigate the new tab directly to the Blob URL; some browsers download it instead of displaying it. Open a blank tab synchronously inside the click handler, then construct a wrapper document with DOM methods. Do not interpolate metadata into `document.write`.

```js
function viewPdfAttachment(id, page) {
  const record = ATTACHMENTS[id];
  if (!record) throw new Error(`Unknown attachment: ${id}`);

  const viewer = window.open('', '_blank');
  if (!viewer) throw new Error('The PDF viewer was blocked by the browser');

  try {
    const pageFragment = Number.isInteger(page) && page > 0 ? `#page=${page}` : '';
    const doc = viewer.document;
    doc.title = safeFilename(record.filename);
    doc.documentElement.style.height = '100%';
    doc.body.style.cssText = 'height:100%;margin:0;overflow:hidden';

    const embed = doc.createElement('embed');
    embed.type = 'application/pdf';
    embed.src = attachmentUrl(id) + pageFragment;
    embed.style.cssText = 'display:block;width:100%;height:100%;border:0';
    doc.body.replaceChildren(embed);
  } catch (error) {
    viewer.close();
    throw error;
  }
}
```

Provide separate **View PDF** and **Download PDF** controls. If a source citation includes a page number, the View control may pass that page to `viewPdfAttachment`; Download must always return the complete original file.

Use delegated or direct click handlers that call these functions inside the click event. Catch failures and show a visible error message; never silently fall back to a path or remote URL.

A complete delegated binding for dynamically rendered rows is:

```html
<p id="attachment-error" role="alert" hidden></p>
<script>
function showAttachmentError(error) {
  const output = document.getElementById('attachment-error');
  output.textContent = `Unable to open attachment: ${error.message}`;
  output.hidden = false;
}

document.addEventListener('click', event => {
  const downloadButton = event.target.closest('[data-attachment-download]');
  const viewButton = event.target.closest('[data-attachment-view]');
  const button = downloadButton || viewButton;
  if (!button) return;

  event.preventDefault();
  event.stopPropagation();

  try {
    if (downloadButton) {
      downloadAttachment(downloadButton.dataset.attachmentDownload);
    } else {
      const parsedPage = Number.parseInt(viewButton.dataset.page || '', 10);
      viewPdfAttachment(
        viewButton.dataset.attachmentView,
        Number.isInteger(parsedPage) ? parsedPage : undefined
      );
    }
  } catch (error) {
    showAttachmentError(error);
  }
});
</script>
```

The matching View control is:

```html
<button type="button"
        data-attachment-view="principal-notice"
        data-page="12">
  View PDF
</button>
```

## 6. Mandatory validation before delivery

Validate the completed HTML, not merely the pre-serialization registry:

1. Parse the `attachment-registry` back out of the saved HTML.
2. For every record, remove no characters except JSON decoding itself, base64-decode the stored payload, and compare the decoded bytes byte-for-byte with the original file.
3. Confirm decoded length equals `size` and SHA-256 equals `sha256`.
4. For PDFs, confirm the original and decoded payloads have identical PDF signatures and exact bytes. Do not accept “opens successfully” as proof of equality.
5. Search the HTML for attachment filesystem paths, `file://`, attachment `src` paths, and runtime `fetch`; none may be used.
6. Open the final HTML through `file://`, click **View PDF**, and confirm the PDF renders.
7. Click **Download PDF**, hash the downloaded file, and confirm it matches the original SHA-256.
8. Repeat View and Download for every attachment, not only the first registry entry.

An HTML file with a registry present but untested controls is incomplete. An HTML file whose button points to the original local path is not self-contained.
