# Receipts by 77systems

Cursor plugin that connects agents to [Receipts](https://www.77systems.ai/receipts) through the hosted Receipts [Model Context Protocol](https://modelcontextprotocol.io/) server.

Receipts checks whether an AI agent's write actually landed. It reads the destination and records the destination's own ID for the object. A write counts as done only when that ID is on record. A 200 response or `success: true` is not proof.

This repository contains only the plugin configuration: the manifest, `mcp.json`, this README, a changelog, and the logo. It contains no Receipts source code. Receipts runs as a hosted service operated by 77systems.

## Install

1. Open **Cursor Settings → Plugins**.
2. Search for **Receipts by 77systems**.
3. Click **Install**, then complete the Receipts sign-in prompt.

Or run `/add-plugin receipts-77systems` in chat.

## MCP

```json
{
  "mcpServers": {
    "receipts": {
      "type": "http",
      "url": "https://receipts.77systems.ai/mcp"
    }
  }
}
```

Auth is OAuth 2.1 with dynamic client registration and PKCE. Cursor registers itself and opens the Receipts sign-in page when the plugin connects. Sign in with Google or an emailed link. There is no API key or client ID to configure, and no token is stored in this plugin.

## Before you connect

You need a Google account or an email address to sign in. Optionally, connect GitHub inside Receipts so Receipts can read your issues itself.

## What each write gets

Every write gets one of five verdicts:

| Verdict | Meaning |
| --- | --- |
| `complete` | Receipts read the destination, found the object, and bound its ID to the SHA-256 digest of the approved payload. Shown as "Verified by Receipts". |
| `reported` | The agent supplied an ID, but Receipts has not read the destination. Shown as "Not independently verified". A later read can upgrade it. |
| `delivery_unknown` | A write may have happened and the answer was lost. Read the destination before doing anything else. |
| `package_unverified` | An object exists but isn't bound to the approved payload. Inspect it; don't create another. |
| `prewrite` | No evidence the write reached the destination. |

No verdict permits an automatic retry or a second write.

## What agents can do

| Category | Tools |
| --- | --- |
| Before the write | `receipts.digest`, `receipts.policy`, `receipts.claim`, `receipts.dispatch`, `receipts.release` |
| After the write | `receipts.observe`, `receipts.recheck`, `receipts.record`, `receipts.bind`, `receipts.complete` |
| Checking | `receipts.classify`, `receipts.verify` |
| Sharing (only if the service has a signing key configured) | `receipts.sign`, `receipts.badge` |

The hosted server is the source of truth for tool names and schemas.

## Example prompts

- "Before you open this GitHub issue, claim it with Receipts. After it's created, have Receipts read it back and show me the receipt."
- "The last write timed out. Use Receipts to find out whether it landed before you try anything else."
- "My agent says it created issue #57. Record that with Receipts, then have Receipts read GitHub and tell me whether it is verified."

## Notes

- GitHub issues are the only destination the hosted service reads today. For other destinations, IDs your agent reports are stored as `reported`.
- Receipts stores IDs, digests and verdicts, not message bodies or issue text. Your audit is append-only and hash-chained, one chain per account.
- The plugin connects only to `https://receipts.77systems.ai`. If you connect GitHub inside Receipts, the Receipts service (not this plugin) reads the issue from `api.github.com` with GET requests only.
- This plugin has no hooks, scripts, rules, skills or local server.

## Pricing

Free forever up to 1,000 verified receipts a month. $19/mo for 25,000, then 0.1¢ each.

Rate limit: 60 checks per minute.

## Links

- Receipts: https://www.77systems.ai/receipts
- Server URL: https://receipts.77systems.ai/mcp
- Support: https://receipts.77systems.ai/support (or hello@77systems.ai)
- Security: security@77systems.ai
- Privacy: https://receipts.77systems.ai/privacy
- Terms: https://receipts.77systems.ai/terms

## License

The [MIT License](LICENSE) in this repository covers only these plugin configuration files: `.cursor-plugin/plugin.json`, `mcp.json`, this README and `CHANGELOG.md`.

Receipts itself is not covered by that license, and its source code is not in this repository. Receipts is a hosted service provided by 77systems, and use of it is governed by the Receipts terms at https://receipts.77systems.ai/terms. The Receipts and 77systems names and the Receipts logo (`assets/logo.png`) are trademarks of 77systems and are not licensed under the MIT License.
