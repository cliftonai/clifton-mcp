# MCP v2 plugin activation — pending

This draft prepares the packaged plugin for the separately approved MCP v2 code promotion. It does not assert that public v2 is live. Do not merge or mark it ready until the backend promotion and final public endpoint checks are recorded.

The plugin endpoint becomes `https://ai.cliftonapi.com/v2/mcp`, its version becomes 0.2.0, and its skill restores the previously reviewed v2 six-tool instructions. The skill follows the live schema for model access and preserves unknown-outcome, polling and Vault download rules; it introduces no new model or routing strategy. Recurring agents remain console-only.

The backend candidate is `5ec74ac6076b041f8a0a63faddb963353c52dda8` (Router 116), deployed by [candidate run 44445971382](https://github.com/cliftonai/src/actions/runs/44445971382). Exact-source personal TEST passed all 13 required checks and 72 browser tests; private candidate PROD passed all 82 checks, including all four cleanup checks. Original six-tool execution, long-request, failure-path and actual Vault-download evidence is retained with its original source identity and executable-source parity assessment. These receipts prove backend behavior, not every consumer app.

Claude Web host registration was not run at Emma's request. The remaining authenticated ChatGPT Web host test was also skipped at her request after dynamic registration and authorization routing succeeded in TEST; no token or tool call was obtained. The owned temporary plugin was uninstalled, with its inactive private definition retained. Claude Desktop was not verified; ChatGPT Desktop custom MCP is not a supported setup path in the current official documentation. No host pass is inferred from CLI or native browser checks.

Before merge:

- Record separate live promotion authority and the completed promotion run for this exact backend.
- Verify public v2 metadata/authentication and current caller-specific tool inventory against the promoted release. Keep checkpointing off.
- Update LIVE.md, README.md and preview setup labels to the actual promoted state; keep the host limitations above explicit.
- Finish exact-head static CI and Claude cross review.

The current published-state guides remain accurate until activation. This draft is preparation, not a completed public cutover.
