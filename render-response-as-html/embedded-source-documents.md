# Embedded source documents

Every source document or other binary attachment included with an HTML output must be encoded inside that HTML file. Do not offer or generate linked, sibling, sidecar, or multi-file variants. External citation links may identify material that is not being included, but they are not a substitute for embedding an attachment that the output claims to contain.

Large binaries can overwhelm the size budget. Warn the user when necessary and optimize safe-to-compress assets, but preserve the single-file contract unless the user changes the requirement.

Use a registry keyed by stable document identifiers. Store MIME type, a sanitized display name, and base64 bytes. Validate every entry by decoding it and comparing the result byte-for-byte with the original.

For an embedded PDF opened from a user action:

1. Open the blank target tab synchronously so popup blockers do not treat it as an unsolicited window.
2. Decode the bytes and create an `application/pdf` Blob URL.
3. Populate the new tab with a minimal wrapper containing an `<embed>` or `<object>` that fills the viewport.
4. Escape all wrapper content. Never place an unsanitized filename or source value into `document.write`.
5. Revoke Blob URLs when safe to do so.

Direct navigation to a PDF Blob URL may download rather than render in some browsers, so test the wrapper behavior in the target browser.

Use one embedded-document registry and access path. Do not create a linked-file adapter or alternate multi-file template.
