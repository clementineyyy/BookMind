# Zotero 插件开发机制与 Windows 兼容基线

- Wayfinder ticket: [查明 Zotero 插件开发机制与 Windows 兼容基线](https://github.com/clementineyyy/BookMind/issues/2)
- Research branch: `research/zotero-plugin`
- Checked: 2026-11-07

## Findings

### Current target baseline

Zotero’s official download page currently offers Zotero 10 for Windows, including x64 and Windows ARM installers. Therefore, “current mainstream Zotero desktop version” should be interpreted as Zotero 10 for this plan, not Zotero 7 or 9. For a release that claims Zotero 10 compatibility, the official developer note says to test the plugin and then set `strict_max_version` to `10.0.*`. Zotero 10 includes internal API and data-storage changes, even though it retains the Firefox 140 ESR base used by Zotero 9. This is evidence that the target version must be explicit and compatibility checked against the major version.

Sources: [Zotero download page](https://www.zotero.org/download/), [Zotero 10 for Developers](https://www.zotero.org/support/dev/zotero_10_for_developers)

### Runtime and trust boundary

A Zotero plugin runs inside the desktop application and uses Zotero’s internal JavaScript API and internal Firefox APIs. Zotero’s user documentation explicitly warns that plugins have full access to the Zotero library and the user’s computer. They are privileged extensions, not browser-style sandboxed WebExtensions. The Zotero 7 developer guide says plugins retain access to platform internals such as XPCOM and file access, while newer Firefox extensions use a more restricted WebExtensions API.

Implication: code that extracts local attachments runs with broad local privileges. Keep the plugin’s responsibilities narrow, avoid unnecessary filesystem/database access, and treat the backend credential and upload path as a product trust boundary. This recommendation is an inference from Zotero’s privilege model.

Sources: [Zotero Plugin Development](https://www.zotero.org/support/dev/client_coding/plugin_development), [Plugins for Zotero](https://www.zotero.org/support/plugins), [Zotero 7 for Developers](https://www.zotero.org/support/dev/zotero_7_for_developers)

### API and UI integration

The local JavaScript API exposes the current Zotero client and item model to code running in the client. Zotero documents `Zotero.getActiveZoteroPane()` and selected-item access, and a notification system for item add/modify/delete events. Zotero states that its JavaScript API documentation is incomplete and plugin developers often need to inspect Zotero source. Its developer docs recommend the local API over direct reads from `zotero.sqlite`, which they describe as more fragile.

Zotero 7 and later added official integration points such as custom item-pane sections, reader event handlers, item-tree columns, and preferences panes. Zotero 8 added `Zotero.MenuManager` APIs for main-window and reader menus. These provide stable options for entry points and context actions; the ticket on reading experience should decide which UI surface best fits the workflow.

Sources: [Zotero JavaScript API](https://www.zotero.org/support/dev/client_coding/javascript_api), [Zotero 7 for Developers](https://www.zotero.org/support/dev/zotero_7_for_developers), [Zotero 8 for Developers](https://www.zotero.org/support/dev/zotero_8_for_developers)

### Packaging, development, installation, and updates

The Zotero 7+ plugin model uses a WebExtension-style `manifest.json` plus a `bootstrap.js` lifecycle entry point; Zotero’s documentation says the manifest must include Zotero application metadata. The plugin is packaged as an `.xpi`. Users install it by downloading the XPI and dragging it into Tools → Plugins. Updates can use a JSON update manifest and XPI download links. The manifest’s maximum Zotero version should reflect versions actually tested; compatibility can also be updated in the update manifest when code changes are unnecessary.

For development, Zotero documents loading a source tree through an extension proxy in a dedicated Zotero profile and recommends a separate profile/data directory for developer builds. An official sample plugin is the starting point. The plugin documentation describes running against source and using debug flags; some old cache instructions are marked as potentially unnecessary since Zotero 7, so confirm against the current app rather than cargo-culting Zotero 6 steps.

Sources: [Zotero Plugin Development](https://www.zotero.org/support/dev/client_coding/plugin_development), [Zotero 7 for Developers](https://www.zotero.org/support/dev/zotero_7_for_developers), [Plugins for Zotero](https://www.zotero.org/support/plugins)

## Architectural consequences for this product

- Build a Zotero desktop plugin, not a standalone browser add-on: the needed library selection, attachment access, and reader entry points live in the privileged client API.
- Keep Zotero-specific code behind a small client adapter. Use documented Zotero APIs where available and avoid direct SQLite reads as the core integration surface.
- Treat local extraction and network transmission as high-trust operations. The plugin can reach local content, but the product must decide and clearly disclose which extracted content is sent to the developer-hosted AI service.
- Declare Zotero 10.0.* as the first compatibility target only after verification on the current Zotero 10 release. Windows-only support does not eliminate the need to account for Windows x64 and ARM installers; the initial CPU architecture support still needs a product decision.
- Use the official sample plugin for lifecycle and manifest structure; select a maintained community build scaffold only after checking its current Zotero 10 compatibility and license.

The first three bullets are recommendations inferred from the documented API and trust model. This research does not select a UI design, backend language/framework, AI provider, data-retention policy, or licensing model.

## Open questions for the architecture decision

1. Is Windows x64 the only first-release architecture, or must Windows ARM also work?
2. Which Zotero reader UI hook gives the best citation navigation workflow for PDF, EPUB, and web content?
3. Which content must be uploaded to the hosted AI service, and can extraction/indexing remain on the client?
4. What minimum Zotero release will be supported, and how will compatibility be checked across future major releases?
