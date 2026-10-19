## 1. Summary

Add a Marketplace for discovering and importing compatible open-standard plugins, previewing their contents, installing them as native skills/MCP connections, and explicitly enabling MCP tools for a bot.

This supports a subset of Agent Plugins, Cursor and Grok manifests—not arbitrary repository execution or every editor-specific plugin feature. The full browser run completed with 259 passed and one avatar-animation timing failure; the unchanged two-test avatar spec passed on an isolated rerun. This contribution is intended for draft review and upstream CI.

## 2. Motivation

A plugin often already describes the skills and remote tools needed for an integration. Re-entering that configuration manually loses the benefit of sharing a plugin in the first place. This change translates supported plugin contents into Rakazo's existing capabilities instead of introducing a second tool runtime.

## 3. Before

Rakazo had skills, MCP connections, OAuth and capability-install records, but no discoverable marketplace or confirmed plugin-to-native installation journey tying them together.

## 4. Problem

Users need to see what a repository will install, retain control over authentication and bot permissions, and remove the resulting capabilities together. Accepting a repository URL alone should not execute its code or silently grant its tools to a bot.

## 5. Approach and scope

Reuse native skills, encrypted MCP credentials, OAuth and bot assignment. Acquire bounded public GitHub content at an immutable commit, preview it, then require the same content digest when installing. Browse public Cursor/xAI indexes with independent failure handling, or import a GitHub repository directly.

Non-goals: plugin updates, local/stdio MCP commands, arbitrary hooks/scripts, editor-only behavior, companion-file skill assets, credential-variable prompts and template import. No new hosted service, model provider, deployment setting or dependency is required.

## 6. Changes

- **Acquisition/adapters:** bounded GitHub parsing, supported manifest formats, SHA pinning, digest confirmation, explicit skipped components and short-lived registry caching.
- **API/contracts:** authenticated preview, catalog and installation; actor-scoped ownership/conflict checks; existing endpoint policy; encrypted headers; atomic native capability creation and uninstall cleanup.
- **Database:** nullable `installId` relations/indexes link plugin-created skills and MCP connections to their install.
- **Web/Electron:** Marketplace navigation, browse/search, progressive GitHub import, preview/Add/Installed/remove, existing OAuth with fixed same-origin return intent, and explicit **Use in this bot**.
- **Mobile:** a localized web handoff rather than pretending a native marketplace exists.
- **Tests:** deterministic acquisition/catalog tests, authorization/installation/secret-cleanup tests, PostgreSQL migration/ownership coverage, UI state tests and browser journeys using fake registries/OAuth.

## 7. User journey and failure handling

`Marketplace → choose plugin or Install from GitHub → Preview → Add → Connect when required → Use in this bot`

Preview identifies supported skills, remote MCP endpoints and skipped parts. Installation does not execute repository code. Missing/mismatched confirmation, duplicate/conflicting skills and unsafe endpoints are rejected. One unavailable registry does not hide another registry's results. Removing a plugin removes its linked capabilities and owned MCP secret material.

Skills are available to the installing user's bots; MCP tool assignment remains a separate approval step. OAuth reuses the existing broker/callback rather than adding another credential flow.

## 8. Delivery state and compatibility

This is an isolated contribution based on upstream `381c4ed7`, not a merge of an application fork. The diff contains only the plugin feature, translations and tests. It has not been merged or released upstream.

Compatibility is deliberately partial. A catalog entry may contain unsupported components, which are shown before confirmation. Live vendor authentication and every published plugin are not covered by the deterministic suite.

## 9. Migration, risks and rollback

The migration adds two nullable foreign-key columns and their indexes; existing skills and MCP records remain unlinked. Uninstall is ownership-scoped and linked records cascade with the installation; MCP secret cleanup is handled transactionally.

Risks include untrusted repository content, public-registry availability, OAuth return behavior and accidental permission expansion. The acquisition limits, immutable confirmation, native endpoint policy, existing encrypted secret store and explicit bot assignment address these boundaries. Reverting application code does not reverse a database migration; retain a database backup for rollout.

## 10. User benefit

Users can bring compatible shared plugin definitions into Rakazo without manually recreating each skill and MCP connection, while keeping preview, authentication and bot permission decisions visible and explicit.

## Verification

- **10,012 unit tests passed** in a PID-isolated runner with child-process reaping; namespace-root permission coverage was additionally exercised as an ordinary user.
- **211 PostgreSQL integration tests passed** across 31 disposable suites, including plugin install ownership, encrypted secret cleanup and migration coverage.
- All **22 typecheck tasks**, repository lint and production builds passed.
- Both new Marketplace browser journeys passed, covering discover/preview/install/OAuth/explicit bot enablement/remove and manual GitHub import/confirmation.
- The full browser regression run completed with **259 passed and one failure** in the unchanged avatar-animation timing assertion. Its two-test spec passed when rerun in isolation without code changes. All Marketplace and MCP OAuth cases passed. This is not represented as an entirely green full-suite invocation.
- No paid provider canaries or real customer data are used by these tests.

Screenshots from `apps/web/e2e/plugin-marketplace.spec.ts` use synthetic fixtures:

![Marketplace search](https://raw.githubusercontent.com/orchidautomation/rakazo-contributions/codex/plugin-marketplace-evidence/docs/evidence/plugin-marketplace/marketplace-search.png)
![Plugin preview](https://raw.githubusercontent.com/orchidautomation/rakazo-contributions/codex/plugin-marketplace-evidence/docs/evidence/plugin-marketplace/marketplace-preview.png)
![Connected plugin with explicit bot enablement](https://raw.githubusercontent.com/orchidautomation/rakazo-contributions/codex/plugin-marketplace-evidence/docs/evidence/plugin-marketplace/marketplace-connected.png)

These are local browser-test captures, not CI screenshots. CI screenshot artifact links can be added once upstream workflows are approved/run.

<details>
<summary>UI copy and progressive disclosure</summary>

Navigation/discovery copy: “Marketplace”, “Discover”, “Installed”, “Search plugins”, “Loading marketplace…”, “No plugins found”, “No plugins installed”, “Some registries are unavailable”, “Close marketplace”. These identify the new surface and show loading/empty/failure states only when applicable.

Import/confirmation copy: “Install from GitHub”, “Plugin GitHub URL”, “Preview”, “Add plugin”, “Remove plugin”, “Confirm removal”, “Retry preview”. GitHub import is collapsed by default; preview and removal confirmation are necessary safety steps. Existing shared action copy is reused where possible.

Permission/connection copy: “Skills”, “Skipped components”, “Skills apply to all your bots. MCP tools require separate approval.”, “Use in this bot”, “Enabled for this bot”, “No sign-in required”. This appears in the selected plugin's details; removing it would obscure supported contents and the distinct skill/tool permission scopes.

Failure-only copy: “Could not enable tools”, “Could not install plugin”, “Could not load installed plugins”, “Could not load marketplace”, “Could not preview plugin”, “Could not refresh installed plugins”, “Could not remove plugin”, “Could not sign in”, “Plugin installed. Could not load connection status.”, “Sign-in cancelled. Try again.” Errors are not persistent explanatory chrome and retain existing retry/recovery behavior.

Native-only copy: “Open marketplace”, “Could not open marketplace”. This is a genuine platform handoff; the native marketplace itself is not implemented, so a web screenshot must not be presented as native UI proof.

</details>
