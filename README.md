# MESS — the ledger your agents keep

**[mess.fyi](https://mess.fyi)** · remote MCP server · `https://mess.fyi/api/mcp`

MESS is a searchable ledger of your service accounts — which Supabase,
which Stripe, which email owns it, which project it belongs to, what it
costs — kept current by your coding agents. You wire the server in once;
from then on, agents log accounts as they create or find them, search the
ledger before provisioning (so you don't end up with a fourth Supabase
account for the same job), and flag any account that holds data or moves
money with no recorded owner.

**The map, not the vault:** MESS stores metadata only. There is no field
for a secret value, on purpose. Discovery reads env var *names* and CLI
auth state — secret values never move.

## Setup

Grab a free key at [mess.fyi](https://mess.fyi) (Settings → Agent keys), then:

**Claude Code**

```bash
claude mcp add --transport http --scope user mess https://mess.fyi/api/mcp \
  --header "Authorization: Bearer mess_sk_YOUR_KEY"
```

**Cursor** — `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "mess": {
      "url": "https://mess.fyi/api/mcp",
      "headers": { "Authorization": "Bearer mess_sk_YOUR_KEY" }
    }
  }
}
```

Any other MCP client that speaks streamable HTTP works the same way.
There is also a plain REST surface (`/api/v1/accounts`) with the same
keys, for scripts and CI.

## The surface

Four tools, one prompt:

| | |
|---|---|
| `log_account` | Log an account the moment it's created — provider, label, owning email, project, cost, where the credentials live. Idempotent. |
| `search_accounts` | Find accounts by project, provider, email, plan — the orientation call when picking a project back up. |
| `list_accounts` | The whole ledger in one call. |
| `update_account` | Correct a row, record an owner, mark an account cancelled. There is no delete — history is kept. |
| `sort_out_my_mess` (prompt) | The dig: a field-tested procedure that inventories the accounts this machine and your projects touch and backfills the ledger. In Claude Code: `/mcp__mess__sort_out_my_mess`. |

Optional Claude Code hooks (one paste, served from the app): a
session-start brief — every session opens knowing what the current
project runs on — and a write-moment nudge that speaks up when a session
touches a provider CLI. Only a provider name and repo folder name are
ever sent.

## Full disclosure

The handshake instructions the server sends your agent are published
verbatim at [mess.fyi/mcp](https://mess.fyi/mcp) — every variant, plus
the dig procedure and the hook nudges. If you'd rather your agent didn't
volunteer writes, don't wire it in.

Free to 20 accounts. Teams share one ledger (scoped digs, personal stays
personal). Questions: hello@mess.fyi.
