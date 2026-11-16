# Repository guidance for AI agents

- Read `README.md` and `CATALOG.md` before modifying documentation.
- Preserve product facts and source attribution; do not invent APIs, parameters, permissions, states, or commands.
- Keep shared execution policy in `shared/rules/` and product-specific facts in `products/`.
- Do not duplicate shared rules inside individual API documents.
- Never commit credentials, tokens, cookies, passwords, or unredacted customer/resource data.
- Update navigation and catalog entries when adding, moving, or retiring documents.
- Treat audit reports as evidence records under `audits/`, not as customer execution instructions.
