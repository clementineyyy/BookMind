# Zotero attachment text and source-location research

Context: [Wayfinder decision ticket: 查明 PDF、EPUB 与网页的书籍内容提取路径](https://github.com/clementineyyy/BookMind/issues/3)

## Findings

Checked against Zotero's current documentation and official repositories on 2026-11-07. Zotero's published `structured-document-text` (SDT) format and `document-worker` project add a first-party structured extraction route for all three input types. They are strong candidates to evaluate, but Zotero does not document them as a stable plugin-facing API.

### Zotero API and storage boundary

Zotero's local JavaScript API exposes selected library items and child attachments (`ZoteroPane.getSelectedItems()`, `item.getAttachments()`, `Zotero.Items.get(...)`). The documented example reads `attachment.attachmentText` for PDF and HTML attachments. Zotero explicitly warns that the client API documentation is incomplete and that some API use requires inspecting source code. A product should use Zotero's attachment/item APIs and avoid reading Zotero's SQLite database directly, which Zotero describes as more fragile. [Zotero JavaScript API](https://github.com/zotero/zotero-docs/blob/main/content/dev/client_coding/javascript_api.md#L173-L200) · [attachment example](https://github.com/zotero/zotero-docs/blob/main/content/dev/client_coding/javascript_api.md#L412-L427)

Zotero 10 also exposes the local Web API v3 at `http://localhost:23119/api/`. It works offline, but must be enabled in Zotero preferences; reads are unauthenticated, and Zotero says not to expose the port externally. Full-text endpoints are available locally. In Zotero 10, object versions are local to that Zotero database and partitioned by `Zotero-Server-ID`, so a helper that caches data must include that ID in its cache key. This is a useful option for an external local helper; an in-process plugin can avoid an extra localhost HTTP boundary by using the client API. [Zotero Local API](https://www.zotero.org/support/dev/web_api/v3/local_api) · [Full-Text Content API](https://www.zotero.org/support/dev/web_api/v3/fulltext_content)

The source implementation describes `attachmentText` as returning plain text for HTML, PDF and plaintext attachments, and warns that paragraph breaks are lost in PDF text. Its PDF extraction can run on demand, but the cached-file fallback path currently returns an empty string for other cached MIME types if no completed cache exists. Zotero's separate full-text indexing code does support EPUB extraction, so relying on `attachmentText` alone for an EPUB that has not been indexed is unsafe. Treat that behavior as an implementation detail to verify against each supported Zotero release. [Zotero `item.js` at inspected commit](https://github.com/zotero/zotero/blob/228f132e1a2c38745ae07bea6c1600b8bddc3b73/chrome/content/zotero/xpcom/data/item.js)

### PDF

Zotero's built-in full-text system uses its bundled Xpdf-derived tools. The client source delegates PDF extraction to `Zotero.PDFWorker.getFullText()` and defaults its own search index to at most 100 pages per attachment; this is a search-index setting, not necessarily a product limit. [Zotero PDF full-text indexing](https://github.com/zotero/zotero-docs/blob/main/content/pdf_fulltext_indexing.md) · [Zotero `fulltext.js` at inspected commit](https://github.com/zotero/zotero/blob/228f132e1a2c38745ae07bea6c1600b8bddc3b73/chrome/content/zotero/xpcom/fulltext.js) · [default preferences](https://github.com/zotero/zotero/blob/228f132e1a2c38745ae07bea6c1600b8bddc3b73/defaults/preferences/zotero.js)

`attachmentText` is a single plain-text string; the documented API does not promise page offsets, and Zotero's own code comment warns about lost PDF paragraph breaks. It is adequate as a basic extraction source, but not by itself as the basis for page-accurate quotations or citations. If page citations are part of the product contract, the design needs a page-aware extraction/locator layer (or carefully uses the reader's existing annotation position model) and must validate it against scanned PDFs, OCR, rotated pages, and unusual layouts. Zotero's user documentation confirms the built-in reader can reopen an annotation at its source page where possible. [PDF reader annotations](https://www.zotero.org/support/pdf_reader#viewing_an_annotation_in_context)

Zotero's official `document-worker` produces SDT for PDFs, EPUBs, and HTML snapshots. SDT stores an outline, page mappings, source anchors, and content blocks; its binary pack supports reading selected blocks without parsing the entire document. PDF anchors can include page indexes and geometry. This is a promising foundation for page-aware chunking, though no stable plugin API contract for invoking the worker or reading these packs was found. Pin and verify a version against supported Zotero releases before depending on it. [document-worker](https://github.com/zotero/document-worker) · [Structured Document Text README](https://github.com/zotero/structured-document-text#readme) · [SDT schema](https://github.com/zotero/structured-document-text/blob/main/schema.d.ts)

### EPUB

The Zotero-maintained reader identifies itself as a PDF/EPUB/HTML reader. Zotero's current client full-text implementation identifies EPUB by `application/epub+zip`, opens it with Zotero's `EPUB` module, iterates section documents, and concatenates each section's `body.innerText` in sequence. It records indexed and total character counts. This gives a viable local text path for the supported EPUBs, but a single concatenated string does not itself preserve a robust section identifier or EPUB CFI locator for citations. The public JS API example does not document EPUB via `attachmentText`; verify the selected Zotero version's support and establish an extraction contract with explicit section boundaries and source locators. [Zotero reader repository](https://github.com/zotero/reader) · [Zotero `fulltext.js` EPUB implementation at inspected commit](https://github.com/zotero/zotero/blob/228f132e1a2c38745ae07bea6c1600b8bddc3b73/chrome/content/zotero/xpcom/fulltext.js)

SDT supports EPUB physical page mappings or synthetic locations and CFI source anchors, and includes the EPUB outline. This preserves much better citation provenance than the flattened search-index text. [Structured Document Text README](https://github.com/zotero/structured-document-text#readme) · [SDT schema](https://github.com/zotero/structured-document-text/blob/main/schema.d.ts)

### Web pages and snapshots

There are two materially different Zotero cases:

- A saved webpage snapshot is a local `text/html` attachment. Zotero's full-text code loads local HTML in a hidden browser, extracts `bodyText`, disables JavaScript, and blocks remote resources. This is useful for stored article text and avoids relying on a live page, but dynamic or externally rendered content can be absent.
- A linked URL is not a saved snapshot. Zotero's full-text index excludes `LINK_MODE_LINKED_URL`, so there may be no local content to read. Supporting the live URL requires a separate fetching policy and handling authentication, robots/paywall boundaries, changing page content, network failures, and user consent.

The Zotero reader repository includes HTML, but the product should distinguish these input cases in its UX and tell users when a citation points to a saved snapshot versus a live URL. [Zotero `fulltext.js` at inspected commit](https://github.com/zotero/zotero/blob/228f132e1a2c38745ae07bea6c1600b8bddc3b73/chrome/content/zotero/xpcom/fulltext.js) · [Zotero JavaScript API attachment example](https://github.com/zotero/zotero-docs/blob/main/content/dev/client_coding/javascript_api.md#L412-L427)

The official SDT snapshot representation adds DOM/CSS selectors and body-text offsets for extracted content, allowing retrieval blocks to map back to a saved snapshot. A live URL still contains no archived source content and requires a separate fetching decision. [document-worker](https://github.com/zotero/document-worker) · [SDT schema](https://github.com/zotero/structured-document-text/blob/main/schema.d.ts) · [Zotero library item types](https://www.zotero.org/support/kb/library_items)

### Full-text API, freshness, and completeness

The full-text API returns indexed attachment text with `indexedPages`/`totalPages` for PDFs or `indexedChars`/`totalChars` for text; a 404 means no indexed content was found. Zotero 10's local API exposes a `/fulltext?since=...` feed to discover attachments whose indexed content version changed. This is useful for incremental synchronization, but the index has a configurable default cap of 100 pages or 500,000 characters, so it is not proof that a whole book was extracted. Compare indexed counts with totals and use full source extraction for the whole-book feature. Preserve the source file hash and extraction version to invalidate derived chunks when content changes. [Full-Text Content API](https://www.zotero.org/support/dev/web_api/v3/fulltext_content) · [Zotero Local API](https://www.zotero.org/support/dev/web_api/v3/local_api) · [Zotero Searching and index limits](https://www.zotero.org/support/searching) · [SDT schema](https://github.com/zotero/structured-document-text/blob/main/schema.d.ts)

## Architecture implications

1. Keep attachment discovery and local-file access in the Zotero plugin, using the local client API. The plugin should send only content required for the user's current summary or question to the developer service; data retention is a separate product decision.
2. Put extraction behind a format-specific adapter that yields structured segments (`text`, format, attachment identity, section/page locator, offsets, extraction status). Do not flatten early to a single string if the UI must cite and reopen evidence.
3. Use Zotero's built-in extraction/index as a useful path and fallback, but do not assume its search index is the full-book source: PDF indexing can be page-limited, EPUB text can be indexed separately, and webpage links have no local snapshot text.
4. The AI/retrieval pipeline must chunk long works and retain provenance through every transformation. It should report missing, partial, encrypted, scanned-without-OCR, or unsupported content rather than silently treating it as complete.
5. Validate source reopening independently for each format. PDF page location has an existing Zotero annotation model; EPUB section/position and HTML snapshot anchors need explicit design and compatibility checks.
6. Prototype Zotero's official SDT worker against the supported Zotero 10.x range. If its packaging or internal dependencies couple too tightly to Zotero, use it as a reference and build version-pinned extraction adapters with the same structured segment and locator contract.

## Source quality and limits

The strongest sources here are Zotero's own developer documentation and official repositories. The JavaScript API page itself says it is incomplete; SDT is a published format/repository, but no stable Zotero plugin API contract for invoking the worker or reading SDT packs was found. Source-code findings are version-sensitive and should be confirmed against supported Zotero 10.x builds. Scanned/encrypted PDFs, malformed EPUBs, dynamic web pages, or incomplete snapshots may yield partial text; the app should report completeness rather than implying all contents were processed.

