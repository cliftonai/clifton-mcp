# MCP v2 plugin activation — pending

This draft prepares the packaged plugin for a separately approved MCP v2 backend promotion. It does not assert that public v2 is live. Do not merge or mark it ready until the backend promotion and final public endpoint checks are recorded.

The plugin endpoint becomes `https://ai.cliftonapi.com/v2/mcp`, its version becomes 0.2.0, and its skill restores the previously reviewed v2 six-tool instructions. The skill follows the live schema for model access and preserves unknown-outcome, polling and Vault download rules; it introduces no new model or routing strategy. Recurring agents remain console-only.

Backend acceptance passed in TEST and private candidate PROD, including required cleanup. Original six-tool execution, long-request, failure-path and actual Vault-download evidence retains its original source identity and executable-source parity assessment. Backend receipts do not prove that every consumer app has completed sign-in or can execute the workflow.

Claude Web and the remaining authenticated ChatGPT Web host test were not run at the operator's request. Dynamic registration and authorization routing succeeded in TEST, but no token or host tool call was obtained. The temporary plugin was uninstalled, with its inactive private definition retained. Claude Desktop was not verified; ChatGPT Desktop custom MCP is not a supported setup path in the current official documentation. No host pass is inferred from CLI or native browser checks.

Before merge:

- Record separate live promotion authority and the completed backend promotion, then verify public v2 metadata, authentication and caller-specific tool inventory against that release.
- Verify the proposed packaged plugin's integration in an isolated Claude Code configuration: install the exact reviewed version, complete OAuth with callback port 8765, inspect its tool inventory and perform a bounded no-model discovery call. Record the actual consumer result separately from original backend ask/poll/download evidence. A complete consumer workflow claim requires actual ask, wait, saved-response and Vault-download observations; do not infer it or replay paid work merely to attach a new commit identity.
- Reconcile current-endpoint and release statements in `LIVE.md`, `README.md`, `API.md`, `INSTALL.md`, `TROUBLESHOOTING.md`, `SECURITY.md`, `AGENT-TOOLS.md`, `CHANGELOG.md` and `examples/` with the actual promoted state. Preserve honest host limitations instead of removing all preview labels indiscriminately.
- Remove the README draft banner and replace this temporary checklist with the actual release-validation record. No branch-specific or private build links belong in published setup instructions.
- Finish exact-head static CI and Claude cross review after those activation changes.

Do not automatically roll back either code or plugin configuration. If a rollback is separately approved, restore a compatible backend/plugin pair and publish a monotonically newer plugin version where required; pointing a plugin back at upgrade-only v1 on a v2 backend is not a valid rollback.

The current published-state guides remain accurate until activation. This draft is preparation, not a completed public cutover.
